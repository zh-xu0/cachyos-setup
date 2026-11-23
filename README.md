# CachyOS Laptop Notes

My personal, copy-pasteable notes for setting up CachyOS the way I like it.
This is a guide for future laptops, not an installer: read a section, run the
commands, and adapt package names to the machine.

**Tested on:** CachyOS rolling, KDE Plasma, ASUS laptop.
Commands assume an Arch-based system (`pacman`, `paru`) and CachyOS package
names. On vanilla Arch, a few packages may live in the AUR instead.

## Placeholders

- `<YOUR-USERNAME>` — my login name (`id -un`)
- `<YOUR-UUID>` — UUID of the data partition (`lsblk -f`)

No real UUIDs, API keys, tokens, hostnames, or account details belong in this
repo. Keep anything machine-specific outside git.

## Fresh-laptop checklist

- [ ] Update the system and install my package set → [01-packages.md](docs/01-packages.md)
- [ ] Mount the data SSD at `/data` and move XDG folders → [02-storage-and-xdg.md](docs/02-storage-and-xdg.md)
- [ ] Shader cache, I/O scheduler, zram → [03-performance.md](docs/03-performance.md)
- [ ] EasyEffects and presets → [04-audio.md](docs/04-audio.md)
- [ ] Reboot, verify, fix problems → [05-troubleshooting.md](docs/05-troubleshooting.md)

## The setup in short

| Preference | What I do |
| --- | --- |
| Data drive | Second SSD mounted at `/data` with a systemd `data.mount` unit |
| Home folders | Real directories on `/data`, symlinked into `$HOME`; `user-dirs.dirs` points at `/data` |
| Swap | zram with **lz4**, `zram-size = ram`, swap priority 100 |
| I/O scheduler | **adios** for NVMe/SATA SSDs, **bfq** for spinning disks, applied by udev |
| GPU shaders | 12 GB shader cache (NVIDIA via `__GL_SHADER_DISK_CACHE_SIZE`, Mesa via `MESA_SHADER_CACHE_MAX_SIZE`) |
| Audio | EasyEffects + LV2 plugins + JackHack96 presets |
| Hardware | `asusctl` / `rog-control-center` for ASUS ROG/TUF features |
| Apps | OnlyOffice, Discord, LocalSend, Etcher, Prism, Chromium, VS Code, JDK 8/17/21/25 |
| Editor | `micro` in all examples (substitute `nano`/`vim` if needed) |

## Repo map

| File | Contents |
| --- | --- |
| [01-packages.md](docs/01-packages.md) | pacman/AUR package list, Java, OnlyOffice, ASUS tools, `tldr` |
| [02-storage-and-xdg.md](docs/02-storage-and-xdg.md) | `/data` mount unit, ownership, symlinks, XDG user dirs |
| [03-performance.md](docs/03-performance.md) | shader cache, udev I/O scheduler rules, zram lz4 |
| [04-audio.md](docs/04-audio.md) | EasyEffects packages, preset import, autoload |
| [05-troubleshooting.md](docs/05-troubleshooting.md) | checks, common failures, how to undo a tweak |

## Ground rules

- **Back up system files before editing.** A quick `sudo cp -a file file.bak`
  has saved me more than once.
- **Never put secrets in this repo.** API keys, tokens, SSH keys, real UUIDs,
  and machine serial numbers stay out.
- **`paru` is never run with `sudo`.** Official packages use `sudo pacman`;
  AUR packages are built as my user.
- These are preferences, not universal best practices. If a future kernel or
  desktop changes behavior, verify the result instead of trusting the note.
