# 05 — Troubleshooting and undo

Checks for each tweak, plus how to reverse it. Run commands as my normal user
unless they start with `sudo`.

## /data does not mount

```bash
systemctl status data.mount
journalctl -u data.mount -b --no-pager
findmnt /data
lsblk -f
```

Common causes:

- **Wrong UUID.** Compare the `What=UUID=...` line with `lsblk -f`.
- **Wrong unit name.** `/data` must be `data.mount`; for another path use
  `systemd-escape --path /mnt/data`.
- **fstab and a mount unit both manage the same path.** Remove one.
- **Mount point busy.** Log out (or boot to a TTY) and try again:

  ```bash
  sudo systemctl restart data.mount
  ```

To undo:

```bash
sudo systemctl disable --now data.mount
sudo rm /etc/systemd/system/data.mount
sudo systemctl daemon-reload
```

## Files seem to have disappeared from /data

If something was written to `/data` before the disk was mounted, it is still on
the root filesystem, hidden under the mount. Unmount and inspect:

```bash
sudo umount /data
ls -A /data
```

Move anything important out before mounting the disk again.

## Home folders are broken links

```bash
findmnt /data
readlink -f ~/Documents
```

If `/data` is not mounted, mount it. If it is mounted, re-run the symlink loop
from [02-storage-and-xdg.md](02-storage-and-xdg.md); it skips links that are
already correct and reports mismatches.

## XDG folders keep reappearing or resetting

Some tools run `xdg-user-dirs-update` and rewrite `~/.config/user-dirs.dirs`.
Disable that behavior:

```bash
echo 'enabled=False' >> ~/.config/user-dirs.conf
```

Then log out and back in. Applications that hard-code `$HOME/Documents` still
work because the symlinks exist.

## zram is not using lz4

```bash
zramctl
systemctl status systemd-zram-setup@zram0.service
```

- Confirm `/etc/systemd/zram-generator.conf` has a `[zram0]` section and
  `compression-algorithm = lz4`.
- Restart the unit:

  ```bash
  sudo systemctl daemon-reload
  sudo systemctl restart systemd-zram-setup@zram0.service
  ```

- If the restart fails while the swap device is busy, reboot. `zramctl` shows
  the algorithm actually in use.

## I/O scheduler did not change

```bash
cat /sys/block/nvme0n1/queue/scheduler
sudo udevadm control --reload-rules
sudo udevadm trigger
```

The active value is in square brackets. If `adios` is not in the list at all,
the kernel does not provide it — replace it with `mq-deadline` in
`/etc/udev/rules.d/60-ioschedulers.rules` and reload the rules.

## Shader cache variables are missing

```bash
cat ~/.config/environment.d/gaming.conf
systemctl --user show-environment | grep -E '__GL_SHADER|MESA_SHADER'
```

`environment.d` is read when the systemd user session starts. Log out and back
in, then restart Steam or the game launcher so it inherits the new environment.
If the variables are still absent, make sure the file is directly inside
`~/.config/environment.d/` and has a `.conf` extension.

## No sound after enabling EasyEffects

```bash
systemctl --user status pipewire wireplumber
pactl info | head
easyeffects --version
```

Try disabling the EasyEffects output preset first. If that restores sound, the
preset is too aggressive or references a missing impulse response. Restarting
the audio stack is heavier-handed:

```bash
systemctl --user restart pipewire wireplumber
```

## A package is not found

```bash
pacman -Ss <package>
paru -Ss <package>
```

- Official packages: `sudo pacman -S --needed <package>`
- AUR packages: `paru -S --needed <package>` — never with `sudo`.
- Update first: `paru -Syu`.

## Java is the wrong version

```bash
archlinux-java status
sudo archlinux-java set java-21-openjdk
```

## ASUS tools do not respond

```bash
systemctl status asusd
asusctl profile get
```

`asusd` is D-Bus activated, so it may be inactive until `asusctl` or
`rog-control-center` asks for it. If it repeatedly fails, check
`journalctl -u asusd -b`. Fan curves and RGB are configured in
`rog-control-center`; a reboot is sometimes needed after firmware or kernel
changes.

## Undo cheat sheet

| Tweak | Undo |
| --- | --- |
| `/data` mount unit | `sudo systemctl disable --now data.mount`, remove the unit, `sudo systemctl daemon-reload` |
| Symlinked home folders | Remove the symlink, move the directory back from `/data` |
| XDG paths | Restore `~/.config/user-dirs.dirs`, remove `enabled=False` from `user-dirs.conf` |
| Shader cache | Remove `~/.config/environment.d/gaming.conf`, log out/in |
| I/O scheduler | Remove `/etc/udev/rules.d/60-ioschedulers.rules`, reload rules |
| zram lz4 | Change `compression-algorithm` back to `zstd`, restart the unit or reboot |
| EasyEffects | Disable it in the app; presets can stay on disk |
