# CachyOS Setup Notes

My personal, copy-pasteable notes for setting up CachyOS the way I like it.
This is a guide for future machines, not an installer: read a section, run the
commands, and adapt package names to the machine.

**Tested on:** CachyOS rolling + KDE Plasma. Hardware-specific sections (for
example ASUS) are marked and can be skipped on other machines.
Commands assume an Arch-based system (`pacman`, `paru`) and CachyOS package
names. On vanilla Arch, a few packages may live in the AUR instead.

## Placeholders

- `<YOUR-USERNAME>` — my login name (`id -un`)
- `<YOUR-UUID>` — UUID of the data partition (`lsblk -f`)

No real UUIDs, API keys, tokens, hostnames, or account details belong in this
repo. Keep anything machine-specific outside git.

## Fresh-install checklist

- [ ] Update the system and install my package set → [01-packages.md](docs/01-packages.md)
- [ ] Mount the data SSD at `/data` and move XDG folders → [02-storage-and-xdg.md](docs/02-storage-and-xdg.md)
- [ ] Shader cache, I/O scheduler, zram → [03-performance.md](docs/03-performance.md)
- [ ] EasyEffects and presets → [04-audio.md](docs/04-audio.md)
- [ ] Install Claude Code and set up the VS Code extension → [05-claude-code.md](docs/05-claude-code.md)
- [ ] Apply the small per-app settings → [06-app-settings.md](docs/06-app-settings.md)
- [ ] Reboot and log back in so the mount, zram, and shader-cache changes apply

## The setup in short

| Preference | What I do |
| --- | --- |
| Data drive | Second SSD mounted at `/data` with a systemd `data.mount` unit |
| Home folders | Real directories on `/data`, symlinked into `$HOME`; `user-dirs.dirs` points at `/data` |
| Swap | zram with **lz4**, `zram-size = ram`, swap priority 100 |
| I/O scheduler | **adios** for NVMe/SATA SSDs, **bfq** for spinning disks, applied by udev |
| GPU shaders | 12 GB shader cache (NVIDIA via `__GL_SHADER_DISK_CACHE_SIZE`, Mesa via `MESA_SHADER_CACHE_MAX_SIZE`) |
| Audio | EasyEffects + LV2 plugins + JackHack96 presets |
| Hardware (ASUS-specific) | `asusctl` / `rog-control-center` for ASUS profiles, fans, and RGB; `power-profiles-daemon` disabled and masked. Skip on non-ASUS machines. |
| Apps | OnlyOffice, Discord, LocalSend, Etcher, Prism, Chromium, VS Code, JDK 8/17/21/25 |
| AI tooling | Claude Code CLI + VS Code extension, pointed at DeepSeek's Anthropic-compatible endpoint |
| Editor | `micro` for every edit (installed in chapter 1); if an old snippet says `nano`, `vim`, or `vi`, use `micro` instead |

## Repo map

| File | Contents |
| --- | --- |
| [01-packages.md](docs/01-packages.md) | pacman/AUR package list, Java, OnlyOffice, optional ASUS tools, power profiles, `tldr` |
| [02-storage-and-xdg.md](docs/02-storage-and-xdg.md) | `/data` mount unit, ownership, symlinks, XDG user dirs |
| [03-performance.md](docs/03-performance.md) | shader cache, udev I/O scheduler rules, zram lz4 |
| [04-audio.md](docs/04-audio.md) | EasyEffects packages, preset import, autoload |
| [05-claude-code.md](docs/05-claude-code.md) | Claude Code install, Fish env, shared settings, VS Code extension |
| [06-app-settings.md](docs/06-app-settings.md) | Small per-app tweaks (Firefox, Dolphin, …), one line each |

## Ground rules

- **Back up system files before editing.** A quick `sudo cp -a file file.bak`
  has saved me more than once.
- **Never put secrets in this repo.** API keys, tokens, SSH keys, real UUIDs,
  and machine serial numbers stay out.
- **`paru` is never run with `sudo`.** Official packages use `sudo pacman`;
  AUR packages are built as my user.
- These are preferences, not universal best practices. If a future kernel or
  desktop changes behavior, verify the result instead of trusting the note.

## Sources

Parts of these notes are adapted from the CachyOS Wiki:

- [Automount with fstab — systemd mount files](https://wiki.cachyos.org/configuration/automount_with_fstab/#safer-and-beginner-friendly-method-mounting-drives-with-systemd-mount-files) — chapter 2.
- [Gaming — Increase maximum shader cache size](https://wiki.cachyos.org/configuration/gaming/#increase-maximum-shader-cache-size) — chapter 3.
- [General system tweaks — ADIOS I/O scheduler](https://wiki.cachyos.org/configuration/general_system_tweaks/#adios-io-scheduler) — chapter 3.
- [General system tweaks — Audio and software enhancements](https://wiki.cachyos.org/configuration/general_system_tweaks/#audio-and-software-enhancements) — chapters 1 and 4.

At the time of writing (2026-11-23), the CachyOS Wiki pages and the
`CachyOS/wiki` repository show no explicit content license. These links are
attribution only; the original authors retain whatever rights they have in the
source material.

## License

MIT — see [LICENSE](LICENSE).

The MIT license covers my original writing only. Adapted sections listed under
Sources are not relicensed beyond what their original terms allow. Third-party
projects linked here (for example the EasyEffects presets) keep their own
licenses.
