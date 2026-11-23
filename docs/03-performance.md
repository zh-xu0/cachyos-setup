# 03 — Performance tweaks

Three independent tweaks I apply on every CachyOS install: a larger GPU shader
cache, an I/O scheduler chosen by device type, and zram with lz4.

## Shader cache (NVIDIA and Mesa)

Games compile shaders the first time they see them. The default cache is small
enough that large games keep recompiling; raising the limit avoids some
stutters and long first loads.

```bash
mkdir -p ~/.config/environment.d
micro ~/.config/environment.d/gaming.conf
```

Paste:

```bash
# NVIDIA — size in bytes (12 GB)
__GL_SHADER_DISK_CACHE_SIZE=12000000000

# AMD / Intel Mesa — supports K/M/G suffixes
MESA_SHADER_CACHE_MAX_SIZE=12G
```

Only the line for the installed GPU matters; the other variable is simply
ignored. `MESA_SHADER_CACHE_MAX_SIZE` also applies to Intel graphics, not just
AMD.

The variables are read by the systemd user session at login, so log out and
back in (or reboot). Verify with:

```bash
systemctl --user show-environment | grep -E '__GL_SHADER|MESA_SHADER'
```

If that prints nothing, the session did not import the file; check the path and
log in again. These variables matter for graphical sessions, not SSH shells.

Cache locations, in case a corrupt cache ever needs clearing:

```bash
~/.cache/nvidia/GLCache
~/.cache/mesa_shader_cache
```

## I/O scheduler: adios for SSDs, bfq for HDDs

Create the udev rule:

```bash
sudo micro /etc/udev/rules.d/60-ioschedulers.rules
```

Paste:

```udev
# HDD
ACTION=="add|change", KERNEL=="sd[a-z]*", ATTR{queue/rotational}=="1", \
    ATTR{queue/scheduler}="bfq"

# SSD
ACTION=="add|change", KERNEL=="sd[a-z]*|mmcblk[0-9]*", ATTR{queue/rotational}=="0", \
    ATTR{queue/scheduler}="adios"

# NVMe SSD
ACTION=="add|change", KERNEL=="nvme[0-9]*", ATTR{queue/rotational}=="0", \
    ATTR{queue/scheduler}="adios"
```

Apply without rebooting:

```bash
sudo udevadm control --reload-rules
sudo udevadm trigger
```

Verify:

```bash
for s in /sys/block/*/queue/scheduler; do
  printf '%s: ' "${s%/queue/scheduler}"
  cat "$s"
done
```

The active scheduler is the one in square brackets. NVMe and SSD devices should
show `[adios]`; spinning disks should show `[bfq]`. zram has no scheduler, so an
empty line for it is normal.

> `adios` is available in current CachyOS kernels. If
> `cat /sys/block/nvme0n1/queue/scheduler` does not list it, replace `adios`
> with `mq-deadline` in the rules — writing a scheduler the kernel does not
> support does nothing.

## zram with lz4

CachyOS uses `zram-generator`. My preference is lz4: faster than zstd, at the
cost of a lower compression ratio. On a machine where zram is ordinary swap,
the latency win is worth it.

```bash
sudo micro /etc/systemd/zram-generator.conf
```

Paste:

```ini
[zram0]
compression-algorithm = lz4
zram-size = ram
swap-priority = 100
fs-type = swap
```

If the file does not exist, install the generator first
(`sudo pacman -S --needed zram-generator`). Keep the `[zram0]` section header.

Apply:

```bash
sudo systemctl daemon-reload
sudo systemctl restart systemd-zram-setup@zram0.service
zramctl
```

The `ALGORITHM` column for `/dev/zram0` should now say `lz4`. If the restart
fails because the swap device is busy, a full reboot applies the change.

Notes:

- `zram-size = ram` creates a compressed swap device the size of RAM. On a
  32 GB laptop that is roughly 31 GiB of zram. Lower the value if that much
  swap encourages bad memory behavior; I keep it as-is.
- The old default is usually `zstd`. To go back, change the algorithm and
  restart the unit again.
