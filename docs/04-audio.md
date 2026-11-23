# 04 — Audio (EasyEffects)

EasyEffects plus a few LV2 plugin packs makes the laptop speakers sound much
better than the stock tuning.

## Packages

If the package step in [01-packages.md](01-packages.md) has not been done:

```bash
sudo pacman -S --needed easyeffects lsp-plugins-lv2 zam-plugins calf mda.lv2
```

> The package name is `mda.lv2` (with a dot), not `mda-lv2`.

Audio runs on PipeWire. Check it if sound is misbehaving:

```bash
systemctl --user status pipewire wireplumber
```

## Presets

I use the community presets from **JackHack96/EasyEffects-Presets**:

<https://github.com/JackHack96/EasyEffects-Presets>

Clone the repository somewhere temporary and follow its README:

```bash
git clone https://github.com/JackHack96/EasyEffects-Presets.git
```

On EasyEffects 8.x, the data lives here:

```bash
~/.local/share/easyeffects/output/   # output presets (*.json)
~/.local/share/easyeffects/irs/      # impulse responses (*.irs)
```

Older EasyEffects versions used `~/.config/easyeffects/`. If imported presets
do not appear, check `easyeffects --version` and use the matching directory.

I keep these third-party presets and impulse responses **out of this notes
repo**. They are other people's files with their own licenses; link to the
upstream repo instead of re-hosting them.

## Apply a preset

1. Open EasyEffects.
2. Go to **Presets → Import** (output side) and select the preset JSON.
3. Pick one, for example `Perfect EQ` or `Laptop`.
4. Set up **Autoload** for the built-in output device so the preset loads
   automatically at login.
5. For games, prefer the **Low Latency** variants — the normal presets can add
   audible latency.

If the preset list is empty, restart EasyEffects after copying the files, and
confirm they are in the correct directory for the installed version.