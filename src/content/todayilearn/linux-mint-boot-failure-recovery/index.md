---
title: "Linux Mint 22 Boot Failure Recovery"
description: "Recovering an HP Pavilion Gaming laptop that refused to boot after a zram install, swap resize, and BIOS defaults reset — using Live USB, fsck, and Recovery Mode."
date: "July 1 2026"
draft: false
---

> AI Generated Content - for documentation purposes only

**Device:** HP Pavilion Gaming 15-ec0xxx
**OS:** Linux Mint 22 XFCE
**Storage:** NVMe SSD 238 GB (`/dev/nvme0n1`)
**Recovery Media:** Linux Mint 22 Live USB (SanDisk Cruzer)

---

## Initial Symptoms

Setelah laptop dimatikan, sistem gagal boot.

Gejala yang muncul:

- Boot berhenti pada:
  - systemd-udevd
  - `[sda] preferred minimum I/O size 4096 bytes`
  - psmouse serio1
  - input: ETPS/2 Elantech Touchpad
- Tidak masuk desktop.
- Awalnya GRUB sempat masuk ke mode CLI (`grub>`).
- Caps Lock kadang masih merespons, kemudian pada beberapa percobaan ikut hang.

---

## Recent Changes Before Failure

Beberapa perubahan yang dilakukan sebelum masalah muncul:

- Install zram.
- Mengubah ukuran swap menjadi ±18 GB.
- Melakukan "Load BIOS Defaults".

Tidak dapat dipastikan bahwa perubahan tersebut merupakan penyebab langsung.

---

## Troubleshooting Timeline

### 1. Boot Diagnostics

Masuk ke GRUB.

Mencoba:

- normal boot
- nomodeset
- compatibility mode

Hasil:

- Normal boot gagal.
- Compatibility Mode pada Live USB berhasil mencapai desktop.

---

### 2. Live USB Recovery

Boot menggunakan Linux Mint 22 Live USB.

Berhasil masuk desktop menggunakan:

```
Start Linux Mint (Compatibility Mode)
```

---

### 3. Identifikasi Disk

Perintah:

```bash
lsblk
```

Hasil penting:

```
/dev/nvme0n1p1   EFI
/dev/nvme0n1p2   Linux Mint Root
```

---

### 4. Filesystem Check

Menjalankan:

```bash
sudo fsck -f /dev/nvme0n1p2
```

Hasil:

```
FILE SYSTEM WAS MODIFIED
```

Beberapa extent tree berhasil dioptimalkan.

Kemudian:

```bash
sudo fsck -f /dev/nvme0n1p1
```

EFI partition dalam kondisi baik.

---

### 5. Planned Boot Repair

Disiapkan langkah:

```bash
update-initramfs -u -k all
update-grub
grub-install /dev/nvme0n1
```

Namun sebelum seluruh proses selesai dilakukan, sistem kembali masuk ke Recovery Mode.

---

### 6. Recovery Mode

Masuk melalui:

```
Advanced Options
→ Recovery Mode
```

Recovery menu:

- Resume
- Clean
- Dpkg
- Fsck
- Grub
- Network
- Root

---

### 7. Root Shell

Masuk ke maintenance shell.

Filesystem di-remount:

```bash
mount -o remount,rw /
```

---

### 8. Recovery FSCK

Recovery meminta remount filesystem menjadi read-write.

Dipilih:

```
YES
```

Karena diperlukan untuk proses repair.

---

### 9. DPKG Repair

Recovery menawarkan repair packages.

Muncul informasi:

```
516 MB additional disk space will be used
```

Dipilih:

```
Y
```

Karena ini merupakan proses reinstall/reconfiguration package sistem, bukan install ulang OS.

---

## Additional Observations

Selama troubleshooting ditemukan:

- Live USB sempat menghasilkan:

```
Buffer I/O error on dev sda1
```

Karena `/dev/sda1` adalah Live USB, kemungkinan penyebabnya:

- flashdisk mulai bermasalah
- koneksi USB tidak stabil
- image USB corrupt

SSD utama tidak menunjukkan indikasi kerusakan.

---

## Commands Used

Filesystem check:

```bash
sudo fsck -f /dev/nvme0n1p2
sudo fsck -f /dev/nvme0n1p1
```

Disk identification:

```bash
lsblk
```

Root remount:

```bash
mount -o remount,rw /
```

Planned recovery:

```bash
update-initramfs -u -k all
update-grub
grub-install /dev/nvme0n1
```

---

## Root Cause (Current Assessment)

Belum dapat dipastikan 100%.

Kemungkinan terbesar:

1. Filesystem menjadi tidak konsisten akibat shutdown paksa.
2. Bootloader mengalami masalah.
3. Package sistem yang tidak konsisten.
4. Kemungkinan konflik driver/kernel saat inisialisasi hardware (terutama touchpad Elantech), meskipun ini belum dapat dipastikan sebagai penyebab utama.

---

## Lessons Learned

- Jangan langsung reinstall Linux ketika Live USB juga mengalami masalah.
- Selalu jalankan `fsck` sebelum melakukan reinstall.
- Recovery Mode sangat berguna untuk memperbaiki sistem tanpa kehilangan data.
- Dokumentasikan setiap perubahan sistem (misalnya pemasangan zram atau perubahan konfigurasi kernel) agar proses diagnosis lebih mudah.

---

## References

Full debugging session: https://chatgpt.com/share/6a452a67-a2e0-83ec-a478-1888d96d6bd5
