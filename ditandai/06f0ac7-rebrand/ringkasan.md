# 06f0ac7 — fix: rebrand package description from 'Ark Linux' to 'Arch Linux'

**Hash:** `06f0ac7`
**Pengarang:** zamkara
**Status:** ❌ BERMASALAH (menyebabkan blank screen)

## Perubahan (dibanding `9ffc8fe`)

### A. Rebrand teks — AMAN
- `pkgdesc`: "Ark Linux" → "Arch Linux"
- Action Description: "Applying Ark Linux system tweaks..." → "Applying Arch Linux system tweaks..."

Ini cuma teks, gak mungkin bikin blank.

### B. Systemd dependency — MENINGKATKAN RESIKO
**File:** `distrobox-init.service`
```
Sebelum:  After=network-online.target / Wants=network-online.target
Sesudah:  After=graphical-session.target / Wants=graphical-session.target
```

**Efek:** Service `distrobox-init` sekarang jalan pas user login (graphical session ready), bukan pas network nyala. Ini user service, jadi gak ngaruh ke GDM langsung — tapi bisa ganggu session startup kalo scriptnya lambat/hang.

**Fungsi:** Biar `gsettings set` jalan pas session GNOME udah siap (butuh dconf/gsettings runtime).

### C. distrobox-init.sh — script baru (Ptyxis via runtime)
**File:** `/usr/lib/systemd/user/distrobox-init.sh`

Script ini ngelakuin:
1. Cek apakah container "arch" udah ada → kalo udah, exit
2. Kalo image `archlinux:latest` gak ada di podman, load dari archive lokal (`/usr/share/ark-distrobox/arch-container.tar.gz`)
3. `distrobox create --name arch --image docker.io/archlinux:latest --init-hooks ...`
4. Hapus `arch.desktop` dari `~/.local/share/applications/`
5. Set Ptyxis default profile via `gsettings set` (pindahan dari yang tadinya di gschema override)

**Bedanya dari `9ffc8fe`:** Di `9ffc8fe`, Ptyxis config dipasang via gschema file (dibaca pas compile-schemas). Di sini dipasang via runtime `gsettings set` di script, jadi butuh session GNOME aktif dan baru jalan pas user login.

**Resiko:** Script `set -e` — kalo ada yang gagal di tengah, sisanya gak jalan. `distrobox create` bisa lambat (tarik image, bikin container). Tapi `|| true` di `gsettings` set, jadi aman.

### D. Hapus Ptyxis GSettings dari gschema
**File:** `ark.gschema.override`

Blok `[org.gnome.Ptyxis]` dan `[org.gnome.Ptyxis.Profile:...]` yang tadinya ada di `9ffc8fe` dihapus dari gschema, dipindah ke `distrobox-init.sh` sebagai runtime `gsettings set`.

## Analisis Penyebab Blank

Perubahan yang paling mungkin menyebabkan blank:

| # | Perubahan | Analisis |
|---|---|---|
| B | Systemd dependency ke `graphical-session.target` | Service ini jalan pas session GNOME startup. Kalo lambat/hang, bisa nahan session selesai loading → efeknya kayak blank |
| C | `distrobox create` di script init | `distrobox create` butuh podman, bisa lambat, tapi gak jelas hubungannya sama GDM |
| D | Hapus dari gschema | Gak ngaruh ke GDM — GDM gak pake Ptyxis |

**Tebakan:** **B** (systemd dep `graphical-session.target`) paling mungkin jadi biang, karena `distrobox-init.service` jalan pas user session startup dan bisa nge-delay/tabrak session GNOME.
