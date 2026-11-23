# 02 — Data SSD at /data + XDG folders

Goal: mount the secondary SSD at `/data`, keep the real user folders there, put
symlinks in `$HOME`, and point the XDG user directories at `/data`.

> **Source:** Adapted from the CachyOS Wiki,
> [Automount with fstab — Safer and beginner-friendly method: mounting drives with systemd mount files](https://wiki.cachyos.org/configuration/automount_with_fstab/#safer-and-beginner-friendly-method-mounting-drives-with-systemd-mount-files).
> I adapted the unit template and added the ownership and XDG-symlink steps.

> **Before mounting:** `/data` must be empty (or already the mounted disk).
> Anything sitting in `/data` before the mount is hidden underneath it. Check
> with `ls -A /data`.

## 1. Find the data partition

```bash
lsblk -f
```

Copy the UUID of the ext4 partition used for data. Ignore the zram/swap rows.

## 2. Create the systemd mount unit

```bash
sudo mkdir -p /data
sudo micro /etc/systemd/system/data.mount
```

Paste this, replacing `<YOUR-UUID>`:

```ini
[Unit]
Description=Secondary EXT4 Data SSD

[Mount]
What=UUID=<YOUR-UUID>
Where=/data
Type=ext4
Options=defaults,noatime,nodev,nosuid

[Install]
WantedBy=multi-user.target
```

> **Unit name rule:** the file name must match the mount point. `/data` →
> `data.mount`. For another path, use `systemd-escape --path /mnt/data`, which
> gives `mnt-data.mount`.

## 3. Enable and mount it

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now data.mount
systemctl status data.mount
```

Do not also add the same mount to `/etc/fstab` — one mount, one mechanism.

## 4. Take ownership

```bash
sudo chown -R <YOUR-USERNAME>:<YOUR-USERNAME> /data
```

On a fresh ext4 filesystem this is usually only needed at the top level:
`sudo chown <YOUR-USERNAME>: /data`. The recursive version is fine but walks
the whole drive, so it can take a while when the disk is already full.

## 5. Move the home folders and symlink them

Run this as my normal user, with file managers and document apps closed.

This step assumes `/data` was freshly mounted and does **not** already contain
folders with these names. Check first with `ls /data`.

Make sure the source folders exist (KDE creates most of them anyway):

```bash
mkdir -p ~/{Desktop,Documents,Downloads,Music,Pictures,Videos,Public,Templates,Projects}
```

Then move each folder to `/data` and leave a symlink behind:

```bash
mv ~/Desktop   /data/ && ln -s /data/Desktop   ~/Desktop
mv ~/Documents /data/ && ln -s /data/Documents ~/Documents
mv ~/Downloads /data/ && ln -s /data/Downloads ~/Downloads
mv ~/Music     /data/ && ln -s /data/Music     ~/Music
mv ~/Pictures  /data/ && ln -s /data/Pictures  ~/Pictures
mv ~/Videos    /data/ && ln -s /data/Videos    ~/Videos
mv ~/Public    /data/ && ln -s /data/Public    ~/Public
mv ~/Templates /data/ && ln -s /data/Templates ~/Templates
mv ~/Projects  /data/ && ln -s /data/Projects  ~/Projects
```

Each line only creates the symlink if the `mv` succeeded. Run this once: after
it finishes, the home folders are symlinks, so running the same `mv` again
would move the symlink itself.

If a folder already exists under `/data` (for example after a partial run),
`mv` would put the home folder *inside* it instead of replacing it. In that
case merge the two folders manually, then create the symlink yourself, or skip
that line.

## 6. Point XDG at /data

```bash
micro ~/.config/user-dirs.dirs
```

Replace the file contents with:

```bash
XDG_DESKTOP_DIR="/data/Desktop"
XDG_DOWNLOAD_DIR="/data/Downloads"
XDG_TEMPLATES_DIR="/data/Templates"
XDG_PUBLICSHARE_DIR="/data/Public"
XDG_DOCUMENTS_DIR="/data/Documents"
XDG_MUSIC_DIR="/data/Music"
XDG_PICTURES_DIR="/data/Pictures"
XDG_VIDEOS_DIR="/data/Videos"
XDG_PROJECTS_DIR="/data/Projects"
```

Notes:

- The first eight are the standard XDG entries. `XDG_PROJECTS_DIR` is an extra
  non-standard entry that some project-aware apps use; I keep it because I like
  having `~/Projects` handled the same way.
- Setting the values to `/data/...` instead of `$HOME/...` is slightly cleaner,
  even though applications would follow the symlinks anyway.
- If something keeps rewriting the file, disable the updater:

  ```bash
  echo 'enabled=False' >> ~/.config/user-dirs.conf
  ```

## 7. Verify

```bash
findmnt /data
ls -ld ~/Desktop ~/Documents ~/Downloads ~/Music ~/Pictures ~/Videos
cat ~/.config/user-dirs.dirs
```

Log out and back in so file managers, browsers, and other apps pick up the new
paths. If the data disk ever fails to mount, the symlinks will be broken until
it is mounted again — the files are not lost, just unavailable.
