# 02 — Data SSD at /data + XDG folders

Goal: mount the secondary SSD at `/data`, keep the real user folders there, put
symlinks in `$HOME`, and point the XDG user directories at `/data`.

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
It is safe to re-run: existing symlinks are skipped.

```bash
dirs=(Desktop Documents Downloads Music Pictures Videos Public Templates Projects)

for dir in "${dirs[@]}"; do
  target="/data/$dir"
  link="$HOME/$dir"

  if [ -L "$link" ]; then
    echo "skip $dir: already a symlink -> $(readlink "$link")"
    continue
  fi

  if [ -e "$link" ] && [ ! -d "$link" ]; then
    echo "skip $dir: $link exists and is not a directory"
    continue
  fi

  if [ -d "$link" ]; then
    if [ -e "$target" ]; then
      echo "skip $dir: both $link and $target exist; merge manually"
      continue
    fi
    mv "$link" "$target"
  else
    mkdir -p "$target"
  fi

  ln -s "$target" "$link"
done
```

Why the checks: the simple form `mv ~/Documents /data/ && ln -s ...` nests the
folder if `/data/Documents` already exists. This loop moves the folder when the
target is free, creates the target when the home folder is missing, and refuses
to guess when both sides have content.

Single-folder equivalent, only when `/data/Documents` does **not** exist:

```bash
mv ~/Documents /data/ && ln -s /data/Documents ~/Documents
```

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
