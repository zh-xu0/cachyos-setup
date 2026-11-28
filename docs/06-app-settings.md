# 06 — App settings

Small per-app tweaks that do not deserve their own chapter. Mostly GUI
toggles, one line each, no commands.

Format for adding an app: one `##` heading, then one bullet per setting.

## Firefox

`about:config` → search each key → set:

```
widget.gtk.global-menu.enabled          true
widget.gtk.global-menu.wayland.enabled  true
```

Global menu in the KDE Plasma panel; both keys are needed on Wayland.

## Dolphin

- **Settings → Configure Dolphin… → View → Remember display style for each
  folder** (checked)

Each folder keeps its own view mode instead of one global layout.


## Theming

Repositories for the Catppuccin theme. Use Catppuccin Macchiato (Pink) if possible

### Firefox: https://github.com/catppuccin/firefox

### Konsole: https://github.com/catppuccin/konsole

Download this file: https://github.com/catppuccin/konsole/blob/main/themes/catppuccin-macchiato.colorscheme
1. Copy the flavor of your choice from [`themes/`](./themes/) to your user konsole directory (usually `~/.local/share/konsole/`).
2. Reload Konsole.
3. Navigate to **Settings** > **Manage Profiles**, then click **New**.
4. Check **Default Profile**.
5. Open the **Appearence** tab and select your flavor.
6. Click **Apply**, then **OK**.

Background color transparency: 10% 

### KDE: https://github.com/catppuccin/kde

1. `git clone --depth=1 https://github.com/catppuccin/kde catppuccin-kde && cd catppuccin-kde`
2. Run the install script using `./install.sh` and follow the instructions.

## Arch-Update

```bash
systemctl --user edit --full arch-update.timer
```
