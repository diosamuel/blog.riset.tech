---
title: "Write-Audit-Publish with Iceberg"
description: "Implementasi pola Write-Audit-Publish pada tabel Iceberg: hapus baris tertentu lewat branch, audit dulu, baru publish ke main secara atomik."
date: "July 1 2026"
---

Implementasi pola **WAP (Write-Audit-Publish)** pada tabel `silver.penelitian` di Iceberg. Tujuan: hapus baris `status='ditolak'` lewat branch, audit dulu, baru publish ke `main`.

## 1. Setup SparkSession + Iceberg REST catalog

Koneksi ke REST catalog (`http://rest:8181`) dengan storage MinIO (`s3://warehouse/`).

```python
spark = (
    SparkSession.builder
    .appName("iceberg-query")
    # Iceberg Spark extensions (aktifkan syntax branch/call, dll)
    .config("spark.sql.extensions",
            "org.apache.iceberg.spark.extensions.IcebergSparkSessionExtensions")
    # Catalog: REST + S3FileIO + MinIO endpoint
    .config("spark.sql.catalog.default",
            "org.apache.iceberg.spark.SparkCatalog")
    .config("spark.sql.catalog.default.type", "rest")
    .config("spark.sql.catalog.default.uri", "http://rest:8181")
    .config("spark.sql.catalog.default.warehouse", "s3://warehouse/")
    .config("spark.sql.catalog.default.io-impl",
            "org.apache.iceberg.aws.s3.S3FileIO")
    .config("spark.sql.catalog.default.s3.endpoint", "http://minio:9000")
    .config("spark.sql.catalog.default.default-namespace", "silver")
    # S3A creds untuk MinIO
    .config("spark.hadoop.fs.s3a.impl", "org.apache.hadoop.fs.s3a.S3AFileSystem")
    .config("spark.hadoop.fs.s3a.endpoint", "http://minio:9000")
    .config("spark.hadoop.fs.s3a.access.key", "admin")
    .config("spark.hadoop.fs.s3a.secret.key", "password")
    .config("spark.hadoop.fs.s3a.path.style.access", "true")
    .config("spark.hadoop.fs.s3a.connection.ssl.enabled", "false")
    .config("spark.sql.defaultCatalog", "default")
    .getOrCreate()
)
```

## 2. Aktifkan WAP di tabel

Wajib agar tabel mendukung branch-based staging.

```sql
ALTER TABLE silver.penelitian
SET TBLPROPERTIES ('write.wap.enabled'='true')
```

## 3. Cek kondisi awal (main)

```python
spark.sql("SELECT status, COUNT(*) AS cnt FROM silver.penelitian GROUP BY status").show()
```

```
+---------+---+
|   status|cnt|
+---------+---+
|  ditolak|511|
| diterima|391|
|diusulkan| 29|
+---------+---+
```

## 4. Buat branch & route write ke branch

```python
commits = "delete_ditolak"

# Buat branch kalau belum ada
spark.sql(f"ALTER TABLE silver.penelitian CREATE BRANCH IF NOT EXISTS wap_{commits}")

# Redirect semua write berikutnya ke branch (bukan main)
spark.conf.set('spark.wap.branch', f"wap_{commits}")
```

## 5. Stage: lakukan perubahan (hanya di branch)

```python
spark.sql("delete from default.silver.penelitian where status = 'ditolak'")
```

Karena `spark.wap.branch` diset, delete hanya menulis ke snapshot branch. `main` tidak tersentuh.

## 6. Audit: bandingkan branch vs main

Pakai `VERSION AS OF` untuk baca snapshot tertentu.

```python
print("STAGED (branch):")
spark.sql(f"SELECT status, COUNT(*) AS cnt FROM default.silver.penelitian VERSION AS OF 'wap_{commits}' GROUP BY status").show()

print("PUBLISHED (main):")
spark.sql("SELECT status, COUNT(*) AS cnt FROM default.silver.penelitian VERSION AS OF 'main' GROUP BY status").show()
```

```
STAGED (branch):
+---------+---+
|   status|cnt|
+---------+---+
| diterima|391|
|diusulkan| 29|
+---------+---+

PUBLISHED (main):
+---------+---+
|   status|cnt|
+---------+---+
|  ditolak|511|
| diterima|391|
|diusulkan| 29|
+---------+---+
```

Branch sudah bersih (ditolak hilang), main masih utuh → aman untuk publish.

## 7. Publish: fast_forward main ke branch

```python
spark.sql(f"""
CALL default.system.fast_forward(
    'silver.penelitian',
    'main',
    'wap_{commits}'
)
""").toPandas()
```

Hasil: `main` diarahkan ke snapshot branch. Sekarang `main` = hasil yang sudah diaudit.

## 8. Verifikasi main

```python
spark.sql("select distinct status from default.silver.penelitian").toPandas()
```

```
      status
0   diterima
1  diusulkan
```

`ditolak` sudah hilang dari main.

## Alternatif: cherrypick_snapshot

Kalau hanya ingin ambil satu snapshot tertentu (tanpa fast-forward seluruh branch):

```sql
CALL default.system.cherrpck_snapshot('silver.penelitian','wap_delete_ditolak')
```

## Ringkasan alur WAP

1. `ALTER TABLE ... SET TBLPROPERTIES ('write.wap.enabled'='true')` — sekali per tabel
2. `CREATE BRANCH` — buat branch staging
3. `spark.conf.set('spark.wap.branch', ...)` — route write ke branch
4. Lakukan DML (insert/update/delete) — hanya ke branch
5. Audit — `VERSION AS OF 'branch'` vs `VERSION AS OF 'main'`
6. `CALL system.fast_forward(...)` — publish ke main
7. Verifikasi main

Pembaca `main` tidak pernah melihat data yang belum diaudit. Publish bersifat atomik (snapshot swap).
