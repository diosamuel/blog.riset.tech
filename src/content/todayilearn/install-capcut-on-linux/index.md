---
title: "Running CapCut on Linux Mint (XFCE) via Wine"
description: "How to install and run CapCut video editor on Linux using Wine, Picom, and virtual desktop mode."
date: "June 22 2026"
---

> **Note:** This content was AI-generated for documentation purposes only, the problem and struggle was real

CapCut can run on Linux using Wine, but it requires specific configuration to avoid issues like black preview overlays and rendering bugs. This guide documents a working setup using Wine, Picom, CapCut 3.9, and Wine virtual desktop mode.

---

## 1. Install Wine

Install Wine and enable 32-bit support:

```bash
sudo dpkg --add-architecture i386
sudo apt update
sudo apt install winehq-stable
```

Check installation:

```bash
wine --version
```

---

## 2. Install Picom (Fix Transparency Issues)

Picom helps fix Wine window transparency problems on XFCE.

Install:

```bash
sudo apt install picom
```

Disable XFCE compositor:

```bash
xfconf-query -c xfwm4 -p /general/use_compositing -s false
```

Start Picom:

```bash
picom &
```

(Optional: add to startup applications)

---

## 3. Download CapCut Version 3.9

Download a compatible version of CapCut:

https://github.com/ProjectBukkit/CapcutVersions

Extract the archive to a local folder.

---

## 4. Pre-Installation Fix (IMPORTANT)

Before installing CapCut, create required Wine directories:

```bash
mkdir -p ~/.wine/drive_c/users/$USER/AppData/Local/CapCut
mkdir -p ~/.wine/drive_c/users/$USER/AppData/Local/CapCut/User\ Data/Config
mkdir -p ~/.wine/drive_c/users/$USER/AppData/Local/CapCut/User\ Data/Log
```

This prevents installation errors and missing configuration issues.

---

## 5. Run CapCut via Wine

Run the executable:

```bash
wine CapCut.exe
```

Or if installed inside Wine Program Files:

```bash
wine "$HOME/.wine/drive_c/Program Files/CapCut/CapCut.exe"
```

---

## 6. Critical Fix: Enable Virtual Desktop (REQUIRED)

If the preview shows a black overlay:

Run:

```bash
winecfg
```

Go to:
Graphics → Enable "Emulate a virtual desktop"

Set resolution:
1920 x 1080

Apply and restart CapCut.

This fixes the black preview overlay issue.

---

## Result

After setup:

- CapCut installs successfully
- Editing features work normally
- Export works correctly
- Preview issue is fixed using virtual desktop mode
- Stable performance under Wine + NVIDIA GPU

---

## Notes

- Wine Staging may improve compatibility
- CapCut 3.9 is more stable than newer versions
- NVIDIA drivers should be properly installed for best performance
- XFCE compositor conflicts are resolved using Picom or disabling xfwm4 compositing

---

## Glossary

- **Wine** (Wine Is Not an Emulator) — compatibility layer yang memungkinkan aplikasi Windows (.exe) berjalan di Linux tanpa perlu instalasi Windows atau virtual machine. Wine menerjemahkan panggilan sistem Windows ke panggilan sistem POSIX secara real-time.

- **XFCE** — desktop environment ringan untuk Linux yang fokus pada kecepatan dan konsumsi memori rendah. Terdiri dari berbagai komponen modular termasuk panel, file manager (Thunar), dan window manager (xfwm4).

- **Picom** — compositor standalone yang berjalan di background. Tugasnya mengelola efek visual seperti transparansi, bayangan, dan blending antar window. Digunakan sebagai pengganti compositor bawaan XFCE karena lebih kompatibel dengan aplikasi Wine.

- **XFWM4** — window manager bawaan XFCE yang menangani penempatan, pergerakan, dan dekorasi window. XFWM4 memiliki compositor internal yang bisa diaktifkan/dinonaktifkan, namun sering konflik dengan aplikasi Wine sehingga perlu digantikan oleh Picom.

- **Compositor** — komponen sistem yang bertanggung jawab menggabungkan (compose) semua window menjadi satu gambar akhir sebelum ditampilkan di layar. Compositor mengelola efek visual seperti transparansi, bayangan, animasi, dan vsync. Tanpa compositor, window ditampilkan tanpa efek apapun.

---

## References

https://www.reddit.com/r/linux/comments/1mt3em7/running_capcut_on_linux_now_working_update/
https://appdb.winehq.org/objectManager.php?sClass=version&iId=42555

---


