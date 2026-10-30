# AceMagic S1 front panel on Linux

A practical guide to customizing the 320 × 170 LCD on the front of the AceMagic S1 mini PC: your own background image, then a simple system dashboard.

It builds on [s1panel](https://github.com/tjaworski/AceMagic-S1-LED-TFT-Linux) by tjaworski, which does the actual work of driving the display.

**[Read the full guide →](ACEMAGIC-S1-LINUX-PANEL-GUIDE.md)**

## What this repository is

| This repository | Upstream s1panel |
|---|---|
| A customization guide (Markdown only) | The driver, LCD protocol work and panel software |
| Themes, backgrounds, dashboard layout, troubleshooting | Installation, the web editor, sensors and widgets |
| No code | [tjaworski/AceMagic-S1-LED-TFT-Linux](https://github.com/tjaworski/AceMagic-S1-LED-TFT-Linux), GPL-3.0 |

This guide isn't affiliated with the upstream project or with AceMagic, and it isn't a compatibility list.

## Requirements

- An AceMagic S1 running Linux.
- s1panel installed from git, by following [upstream's instructions](https://github.com/tjaworski/AceMagic-S1-LED-TFT-Linux/tree/main/s1panel), and already showing its demo screen.
- ImageMagick, for the background image.
- `python3`, for checking JSON before you restart anything.

The guide assumes upstream's default git install:

| Setting | Assumed value |
|---|---|
| s1panel directory | `$HOME/acemagic/s1panel` |
| systemd service | `s1panel` |
| Main configuration | `$HOME/acemagic/s1panel/config.json` |
| Your theme | `$HOME/acemagic/s1panel/themes/my-panel/` |

Check that yours matches:

```bash
test -d "$HOME/acemagic/s1panel" && echo "s1panel directory found"
systemctl status s1panel --no-pager
```

> [!NOTE]
> If you installed the snap, your files are in `/root/snap/s1panel/current/` and the commands differ. The guide points out where.

## Getting started

1. [Back up](ACEMAGIC-S1-LINUX-PANEL-GUIDE.md#1-back-up-the-working-configuration) `config.json` and `themes/`.
2. [Create a theme folder](ACEMAGIC-S1-LINUX-PANEL-GUIDE.md#2-create-a-custom-theme-folder).
3. [Make a 320 × 170 background](ACEMAGIC-S1-LINUX-PANEL-GUIDE.md#3-prepare-a-background-image).
4. [Write a minimal `theme.json`](ACEMAGIC-S1-LINUX-PANEL-GUIDE.md#4-create-a-static-image-theme).
5. [Point `config.json` at it](ACEMAGIC-S1-LINUX-PANEL-GUIDE.md#5-select-the-custom-theme) and restart.
6. Once the static image works, [add a dashboard](ACEMAGIC-S1-LINUX-PANEL-GUIDE.md#6-add-a-system-dashboard).

## Before you start

> [!WARNING]
> The `config.json` that ships with upstream sets `"listen": "0.0.0.0:8686"`, which serves the web editor to your whole network. The editor has no login. Set it to `127.0.0.1:8686` and use an SSH tunnel instead. See [Keep the editor off your network](ACEMAGIC-S1-LINUX-PANEL-GUIDE.md#keep-the-editor-off-your-network).

- **The LCD isn't a monitor.** It's a USB HID device. Only s1panel should talk to it, so don't run a second program against it.
- **Faster isn't smoother.** Full-screen USB transfers are slow. Use `"refresh": "update"` in your theme, and refresh most widgets every 15 to 30 seconds.

## Status

- Field names, defaults and commands were checked against upstream commit `c6491eb` (June 2026).
- The examples are starting points. Which distribution and s1panel version they were tested on hasn't been recorded yet.
- If something no longer matches upstream, please [open an issue](../../issues).

## Credits

The reverse-engineering and s1panel itself are [tjaworski's work](https://github.com/tjaworski/AceMagic-S1-LED-TFT-Linux). This repository is a customization guide on top of it.
