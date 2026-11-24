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
