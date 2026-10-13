---
title: "Fixing Iceberg Metadata Tables & CALL Procedures in Spark"
description: "Three distinct problems when using Iceberg metadata tables and CALL procedures in Spark: procedure name confusion, version mismatch between client/server, and SQLite permissions on Docker volumes."
date: "July 17 2026"
draft: false
---

# Fixing Iceberg Metadata Tables & CALL Procedures in Spark

> **AI Generated Content - for documentation purposes only, but the struggle and error are real**

> **TL;DR** — If `spark.sql("CALL default.system.list_refs(...)")` throws `PARSE_SYNTAX_ERROR`, or `spark.sql("SELECT * FROM silver.hibah_lengkap.refs")` returns a 500 / `NoSuchTableException` / "Suspicious Path Character", the root cause is a **version mismatch between your Spark client JARs and the Iceberg REST server image**, compounded by a SQLite permissions bug on the persistent catalog volume. This guide explains the three distinct problems and how we fixed each one.

---

## Background: what are Iceberg metadata tables and CALL procedures?

When you use Iceberg from Spark, the Iceberg Spark extensions (`IcebergSparkSessionExtensions`) add two kinds of SQL syntax:

### 1. Metadata tables
Iceberg stores the full history of every change to a table (snapshots, branches, schemas, manifest files). You can query these from SQL with a dot-suffix on the table name:

```sql
SELECT * FROM silver.hibah_lengkap.refs;          -- branches and refs
SELECT * FROM silver.hibah_lengkap.snapshots;     -- individual snapshots
SELECT * FROM silver.hibah_lengkap.history;        -- snapshot commit timeline
SELECT * FROM silver.hibah_lengkap.metadata_log_entries;  -- metadata JSON versions
```

### 2. CALL procedures
Iceberg exposes operations that **change** the table state as `CALL` statements:

```sql
CALL default.system.fast_forward('silver.penelitian', 'main', 'audit-swap');
CALL default.system.expire_snapshots('silver.hibah_lengkap', TIMESTAMP '...', 1);
```

> ⚠️ **There is no `CALL default.system.list_refs(...)` procedure.** Listing refs/history/snapshots is done via the metadata tables (the `SELECT * FROM table.refs` syntax above). Confusing `LIST` (a Spark reserved keyword) for a procedure name is the first trap.

---

## Problem 1: `CALL default.system.list_refs(...)` → `PARSE_SYNTAX_ERROR`

### The symptom

```python
spark.sql("CALL default.system.list_refs('silver.hibah_lengkap');")
```

```
ParseException: 
[PARSE_SYNTAX_ERROR] Syntax error at or near 'CALL'.(line 2, pos 0)

== SQL ==
CALL default.system.list_refs('silver.hibah_lengkap')
^^^
```

### The cause

There is **no Iceberg procedure called `list_refs`**. The string `LIST` is a Spark SQL reserved keyword (used in `LIST JAR`, `LIST FILE`, and `LIST` clauses inside CTEs). The Iceberg Spark SQL grammar cannot consume it as a procedure identifier, so the parser rejects the entire `CALL` statement at position 0 — before it even checks whether the procedure exists.

This is pure user error — the same syntax has never worked on any Iceberg version.

### The fix

Don't use `CALL` for listing. Use the metadata table `silver.hibah_lengkap.refs` instead:

```python
spark.sql("SELECT * FROM silver.hibah_lengkap.refs").show()
```

For operations that **do** exist as CALL procedures, use the correct names:
- `fast_forward`
- `expire_snapshots`
- `remove_orphan_files`
- `set_current_snapshot`
- `rollback_to_snapshot`
- `cherrypick_snapshot`
- `migrate`
- `add_files`
- `rewrite_data_files`
- `rewrite_position_delete_files`
- `rewrite_manifests`

---

## Problem 2: `SELECT * FROM silver.hibah_lengkap.refs` → 500 / NoSuchTableException / "Suspicious Path Character"

### The symptom

```python
spark.sql("SELECT * FROM silver.hibah_lengkap.refs").show()
```

```
BadRequestException: Malformed request: {
  "message":"Suspicious Path Character",
  "url":"http://rest:8181/v1/namespaces/silver%1Fhibah_lengkap/tables/refs?snapshots=all",
  "status":"400"
}
```

After Spark falls back to a second attempt, the REST server logs:

```
NoSuchTableException: Table does not exist: silver.hibah_lengkap.refs
```

### The cause

A **client/server version mismatch between Iceberg 1.9.0 and 1.10.1**:

- The Spark container had been built with `ICEBERG_VERSION=1.9.0` in `spark/Dockerfile` (Iceberg 1.9.0 client JARs: `iceberg-spark-runtime-3.5_2.12-1.9.0.jar`).
- The REST catalog server image in `docker-compose.yaml` was unpinned: `apache/iceberg-rest-fixture` (defaults to `:latest` on Docker Hub). At the time of the bug, `:latest` resolved to **Iceberg 1.10.1**.

When Spark 1.9.0 requests the `silver.hibah_lengkap.refs` metadata table, the client encodes the multi-level namespace separator `silver.hibah_lengkap` as the raw `%1F` byte (ASCII Unit Separator). The newer 1.10.1 Jetty/URI validator on the server rejects that byte as *Suspicious Path Character* — the request never reaches the catalog handler. When the client retries without the `?snapshots=all` query-string the server simply treats `hibah_lengkap.refs` as a literal table name and replies `404 Table does not exist: silver.hibah_lengkap.refs`.

The Iceberg REST contract is unstable across minor versions for exactly these internal namespace separator encodings; both sides have to match.

### The fix

**Pin both sides to the same Iceberg version.** We upgraded the Spark client to 1.10.1 to match the server.

In `spark/Dockerfile`:
```dockerfile
ENV SPARK_VERSION=3.5.6
ENV SPARK_MAJOR_VERSION=3.5
ENV ICEBERG_VERSION=1.10.1   # was 1.9.0
```

In `docker-compose.yaml`:
```yaml
rest:
  image: apache/iceberg-rest-fixture:1.10.1   # was apache/iceberg-rest-fixture (=:latest)
```

Then rebuild the Spark image and recreate the containers:
```bash
docker compose pull rest
docker compose build spark-iceberg
docker compose up -d rest spark-iceberg
```

After this fix, `SELECT * FROM silver.hibah_lengkap.refs` returns data.

---

## Problem 3: `start.sh` pipelines fail with `UncheckedSQLException` (SQLite READONLY_DIRECTORY)

### The symptom

After upgrading to 1.10.1 and adding a persistent Docker volume for the catalog, running `start.sh`:

```bash
docker compose exec spark-iceberg python /home/iceberg/pipeline/schema/goldSchema.py
```

fails:

```
pyiceberg.exceptions.ServerError: UncheckedSQLException: Failed to execute:
DELETE FROM iceberg_tables WHERE catalog_name = ? AND table_namespace = ?
AND table_name = ? AND (iceberg_type = 'TABLE' OR iceberg_type IS NULL)
```

Checking the REST server logs:

```
Caused by: org.sqlite.SQLiteException:
[SQLITE_READONLY_DIRECTORY] Process does not have permission to create a journal file
in the same directory as the database and the creation of a journal file is a prerequisite
for writing (attempt to write a readonly database)
```

### The cause

The Iceberg REST fixture uses a SQLite database as its JDBC catalog backend. By default it stores the DB at `/tmp/iceberg_catalog.db` inside the container — which is wiped every time the container is recreated.

To persist the catalog across restarts we added a Docker named volume and pointed the URI at it:

```yaml
environment:
  - CATALOG_URI=jdbc:sqlite:/var/lib/iceberg/rest_catalog.db
volumes:
  - iceberg-rest-catalog:/var/lib/iceberg
```

That introduces a **permissions mismatch**:

- Docker creates named volumes owned by `root:root`.
- The `apache/iceberg-rest-fixture` image runs as user `iceberg` (uid `1000`) by default.
- SQLite needs to create a `-journal` (or `-wal` / `-shm`) sidecar file **in the same directory** as the DB to perform any write — and uid 1000 has no write permission on `/var/lib/iceberg/`.

Every `DROP TABLE` / `CREATE TABLE` / `CREATE OR REPLACE TABLE` from `pipeline/schema/goldSchema.py` or `pipeline/index.py` hits the 500 Server Error → `UncheckedSQLException`. The pre-existing `:latest` fixture never hit this because it used `/tmp` (world-writable), so the permissions issue only surfaced after adding persistence.

### The fix

Run the REST container as `root` so SQLite can create its journal files in the volume:

```yaml
rest:
  image: apache/iceberg-rest-fixture:1.10.1
  user: "0:0"   # ← added; fixture image defaults to 1000:1000
  environment:
    # ... existing catalog env ...
    - CATALOG_URI=jdbc:sqlite:/var/lib/iceberg/rest_catalog.db
    - CATALOG_CATALOG__IMPL=org.apache.iceberg.jdbc.JdbcCatalog
    - CATALOG_JDBC__STRICT__MODE=true
  volumes:
    - iceberg-rest-catalog:/var/lib/iceberg
```

After recreating:
```bash
docker compose up -d rest
docker compose exec spark-iceberg python /home/iceberg/pipeline/schema/goldSchema.py
docker exec -it lppm-spark-iceberg python3 /home/iceberg/pipeline/index.py
```

Both pipelines now succeed. The catalog DB is persisted in the `iceberg-rest-catalog` volume, and SQLite has the directory permissions it needs.

---

## Summary of all changes

| File | Change | Fixes |
|---|---|---|
| `spark/Dockerfile` | `ICEBERG_VERSION=1.10.1` (was `1.9.0`) | Problem 2 (version mismatch) |
| `docker-compose.yaml` `rest` service | `image: apache/iceberg-rest-fixture:1.10.1` (was unpinned `latest`) | Problem 2 (version mismatch) |
| `docker-compose.yaml` `rest` service | `user: "0:0"` added | Problem 3 (SQLite readonly volume) |
| `docker-compose.yaml` `rest` service | `environment: CATALOG_URI`, `CATALOG_CATALOG__IMPL`, `CATALOG_JDBC__STRICT__MODE` added | Catalog persistence (pre-condition for Problem 3) |
| `docker-compose.yaml` `rest` service | `volumes: iceberg-rest-catalog:/var/lib/iceberg` added | Catalog persistence |
| `docker-compose.yaml` top-level `volumes:` block | `iceberg-rest-catalog:` added | Catalog persistence |
| Notebook usage | Replace `CALL default.system.list_refs(...)` with `SELECT * FROM silver.hibah_lengkap.refs` | Problem 1 (no such procedure) |

---

## Verifying the fix

After applying all changes, this one-liner in any notebook cell confirms both branches work:

```python
spark.sql("SELECT * FROM silver.penelitian.history").show(truncate=False)
```

```
+-----------------------+-------------------+---------+-------------------+
|made_current_at        |snapshot_id        |parent_id|is_current_ancestor|
+-----------------------+-------------------+---------+-------------------+
|2026-07-12 17:07:31.643|7449668716037505807|NULL     |true               |
+-----------------------+-------------------+---------+-------------------+
```

If the metadata-table query returns data, Problem 2 and Problem 3 are both resolved. You can also verify that REAL CALL procedures work:

```python
spark.sql("CALL default.system.fast_forward('silver.penelitian', 'main', 'audit-swap')").show()
```

If `fast_forward` runs without `PARSE_SYNTAX_ERROR` or `Procedure system.fast_forward not found`, Problem 1 is resolved (i.e. the Iceberg Spark extensions are correctly loaded against the matching server).

---

## Key takeaways

1. **Iceberg client and server should be pinned to the same version.** Don't let `docker-compose.yaml` use `:latest` for the REST catalog while Dockerfile pins a specific JAR version. Pin both explicitly.
2. **`LIST` is a Spark reserved keyword** — there is no `list_refs`, `list_snapshots`, or `snapshots` CALL procedure. Use metadata-table syntax for reading, CALL procedures for writing/branching.
3. **SQLite + Docker named volumes require root or dir chown** — the Iceberg REST fixture image runs as `iceberg` (uid 1000), but Docker creates volumes as `root`. SQLite's *same-directory journal file* rule means the server process must own/write the volume directory. Either run the container as `root` (`user: "0:0"`) or fix the volume permissions with an init container.