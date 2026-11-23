# 01 — Packages and apps

The commands below use CachyOS package names. `--needed` makes re-running them
safe: packages that are already installed are skipped.

## First, update

```bash
sudo pacman -Syu
```

## Official repository packages

### Editor

```bash
sudo pacman -S --needed micro
```

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

> **Source:** The audio and OBS package recommendations come from the CachyOS
> Wiki, [General system tweaks — Audio and software enhancements](https://wiki.cachyos.org/configuration/general_system_tweaks/#audio-and-software-enhancements).

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
sudo systemctl disable --now power-profiles-daemon.service
sudo systemctl mask power-profiles-daemon.service
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

```bash
pacman -Ss <package>
paru -Ss <package>
```
