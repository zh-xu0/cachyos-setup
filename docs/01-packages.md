# 01 — Packages and apps

The commands below use CachyOS package names. `--needed` makes re-running them
safe: packages that are already installed are skipped.

## First, update

```bash
paru -Syu
```

`paru -Syu` updates both the official repositories and the AUR. On a fresh
CachyOS install `paru` is already there. If it is missing, update with
`sudo pacman -Syu` and install `paru` from the CachyOS repository.

## Official repository packages

### Browser, chat, sharing, office

```bash
sudo pacman -S --needed onlyoffice-bin localsend discord etcher-bin
sudo pacman -S --needed chromium
```

> The package is `onlyoffice-bin` on CachyOS — `onlyoffice` alone does not
> exist. `etcher-bin` is also in the CachyOS repository (on vanilla Arch it is
> an AUR package).

### Java

```bash
sudo pacman -S --needed jdk8-openjdk jdk17-openjdk jdk21-openjdk jdk25-openjdk

# See what is installed and which one is the default
archlinux-java status

# Optional: pick a default
sudo archlinux-java set java-21-openjdk
```

If `jdk25-openjdk` is not available on an older snapshot, install the newest
JDK the repositories offer and adjust the commands.

### Minecraft

```bash
sudo pacman -S --needed prismlauncher
```

### KDE global menu integration for GTK apps / OnlyOffice

```bash
sudo pacman -S --needed appmenu-gtk-module libdbusmenu-glib
```

On CachyOS these are in the official repositories. On vanilla Arch,
`appmenu-gtk-module` may be in the AUR.

### Audio stack

EasyEffects and its plugins. The preset setup itself is in
[04-audio.md](04-audio.md).

```bash
sudo pacman -S --needed easyeffects lsp-plugins-lv2 zam-plugins calf mda.lv2
```

> The package name is `mda.lv2` (with a dot), not `mda-lv2`.

### OBS Studio

```bash
sudo pacman -S --needed obs-studio-browser
```

`obs-studio-browser` is the CachyOS build with the browser source. On vanilla
Arch the package is `obs-studio` (browser support may be a separate package).

### ASUS ROG / TUF tools

```bash
sudo pacman -S --needed asusctl rog-control-center
```

Useful commands:

```bash
asusctl profile list
asusctl profile get

# Profiles: Quiet, Balanced, Performance
asusctl profile set Balanced
asusctl profile set -a Performance   # profile while on AC power
asusctl profile set -b Quiet         # profile while on battery
```

`asusd` is D-Bus activated; it starts on demand when `asusctl` or
`rog-control-center` needs it. Check it with `systemctl status asusd`.
Fan curves, keyboard RGB, battery charge limit, and other ASUS features are in
`rog-control-center`.

## AUR packages

Run these as my normal user — never with `sudo`.

```bash
paru -S --needed polychromatic ttf-ms-fonts ttf-wps-fonts visual-studio-code-bin
```

| Package | Why |
| --- | --- |
| `polychromatic` | Razer mouse/keyboard configuration GUI |
| `ttf-ms-fonts` | Microsoft core fonts for Office documents |
| `ttf-wps-fonts` | Font set used by WPS/OnlyOffice documents |
| `visual-studio-code-bin` | Microsoft build of VS Code |

## tldr

```bash
tldr --update
```

## Package-name gotchas

These bit me when I copied old notes:

| Old note | Use this | Reason |
| --- | --- | --- |
| `sudo pacman -S onlyoffice ...` | `onlyoffice-bin` | `onlyoffice` is not a CachyOS package |
| `appmenu-gtk-module libdbusmenu-glib` listed as AUR | official on CachyOS | may be AUR on vanilla Arch |
| `etcher-bin`, `obs-studio-browser`, `lsp-plugins-lv2`, `jdk25-openjdk` | CachyOS repos | names can differ elsewhere |
| `mda-lv2` | `mda.lv2` | dot in the package name |

If a package is not found, search both repositories and the AUR:

```bash
pacman -Ss <package>
paru -Ss <package>
```
