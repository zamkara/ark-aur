# ark-aur

Pre-built AUR and custom packages for Ark Linux. Published as GitHub Releases and consumed by `ark-image` during container builds.

## Package Sources

### AUR Packages (auto-built from aur.archlinux.org)

| Package | Notes |
|---------|-------|
| `bootc` | Bootable container manager |
| `bootupd` | EFI bootloader updater |
| `morewaita-icon-theme` | Extended Adwaita icon theme |
| `pods` | GTK4 Podman frontend |
| `distroshelf` | GTK4 Distrobox GUI |
| `starship` | Cross-shell prompt |
| `fastfetch` | System info tool |
| `scrcpy` | Android screen mirror |
| `android-tools` | ADB and fastboot |
| `nix` | Nix package manager |
| `distrobox` | Container-based package management |
| `plymouth` | Boot splash screen |
| `zram-generator` | zRAM swap |
| `github-cli` | GitHub CLI |
| `gnome-backgrounds` | GNOME wallpapers |
| `base-devel` | Arch build tools |
| `gnome-disk-utility-git` | GNOME Disks (git) |

### Local Packages (custom, built from `.github/`)

| Package | Source |
|---------|--------|
| `ark-system-tweaks` | Custom system configuration tweaks |
| `helium-browser` | De-Googled Chromium fork from [imputnet/helium-linux](https://github.com/imputnet/helium-linux) — version fetched dynamically from latest GitHub release |

## Build Schedule

CI runs automatically on **Monday, Wednesday, Friday** at 00:00 UTC. Each successful build triggers rebuilds of `ark-image` and `ark.linux` via `GH_PAT`.

Manual trigger is also available via `workflow_dispatch`.

## Helium Browser

- Source: `https://github.com/imputnet/helium-linux/releases/latest`
- GPG key: `BE677C1989D35EAB2C5F26C9351601AD01D6378E` (Helium signing key, `helium@imput.net`)
- Version is resolved at CI build time — never hardcoded in PKGBUILD
- Installed to `/opt/helium/`, symlinked at `/usr/local/bin/helium`

## Repository Structure

All build files live inside `.github/`:

```
.github/
├── workflows/
│   └── build.yml          — CI workflow (AUR + local packages + pacman cache)
├── helium-browser/
│   └── PKGBUILD
└── ark-system-tweaks/
    └── PKGBUILD
```

No new folders at repo root.

## Commit Rules

- No `Co-Authored-By` or AI attribution in commit messages
- Never push without explicit user authorization
- This repo is Group 1 — must be pushed before `ark-image` and `ark.linux`
- Never hardcode version numbers for external packages — always resolve dynamically at build time
