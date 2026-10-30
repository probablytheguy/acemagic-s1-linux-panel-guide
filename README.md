# AceMagic S1 front panel on Linux

The AceMagic S1 mini PC has a small 320 × 170 LCD on the front. On Linux, the [s1panel](https://github.com/tjaworski/AceMagic-S1-LED-TFT-Linux) project by tjaworski drives it. This repository explains how to replace s1panel's demo theme with your own background image and a simple system dashboard.

**[Read the full guide →](ACEMAGIC-S1-LINUX-PANEL-GUIDE.md)**

## What's here

- A step-by-step guide to:
  - back up your working setup,
  - create your own theme,
  - turn any image into a correctly sized background,
  - build a small dashboard (CPU, temperature, memory, disk),
  - fix the usual problems.
- A few things worth knowing before you start, like refresh settings and keeping the web editor off your network.

## What's not here

- **The software.** The driver, the LCD protocol work, s1panel and its web editor all come from the [upstream project](https://github.com/tjaworski/AceMagic-S1-LED-TFT-Linux) (GPL-3.0). This repository contains no code from it, and it isn't affiliated with that project or with AceMagic.
- **Installation instructions.** Install s1panel by following upstream's README first, then come back here.
- **A compatibility list.** I haven't tested this across distributions or s1panel versions, so I'm not claiming any.

## Before you start

You need:

- an AceMagic S1 running Linux, with s1panel installed from git and already showing its demo screen;
- ImageMagick (`convert` or `magick`) for the background image;
- `python3`, to check your JSON before restarting anything.

The guide assumes the paths you get from upstream's git install: `$HOME/acemagic/s1panel` and a systemd service called `s1panel`. Check yours:

```bash
test -d "$HOME/acemagic/s1panel" && echo "s1panel directory found"
systemctl status s1panel --no-pager
```

If you installed the **snap**, your files are in `/root/snap/s1panel/current/` and the commands differ. The guide notes where.

## Getting started

1. [Back up](ACEMAGIC-S1-LINUX-PANEL-GUIDE.md#1-back-up-the-working-configuration) `config.json` and `themes/`.
2. [Create a theme folder](ACEMAGIC-S1-LINUX-PANEL-GUIDE.md#2-create-a-custom-theme-folder), then [prepare a 320 × 170 background](ACEMAGIC-S1-LINUX-PANEL-GUIDE.md#3-prepare-a-background-image).
3. [Write a minimal `theme.json`](ACEMAGIC-S1-LINUX-PANEL-GUIDE.md#4-create-a-static-image-theme) and [point `config.json` at it](ACEMAGIC-S1-LINUX-PANEL-GUIDE.md#5-select-the-custom-theme).
4. Once the static image works, [add a dashboard](ACEMAGIC-S1-LINUX-PANEL-GUIDE.md#6-add-a-system-dashboard).

## Things worth knowing first

- **The LCD isn't a monitor.** It's a USB HID device that only s1panel should talk to. Don't run a second program against it.
- **The web editor may be open to your network.** The `config.json` shipped upstream sets `"listen": "0.0.0.0:8686"`, and the editor has no login. Set it to `127.0.0.1:8686` and use an SSH tunnel. [Details in the guide](ACEMAGIC-S1-LINUX-PANEL-GUIDE.md#keep-the-editor-off-your-network).
- **Faster isn't smoother.** Full-screen USB transfers are slow. Use `"refresh": "update"` in your theme, and refresh most widgets every 15 to 30 seconds.

## Status

Field names, defaults and commands were checked against the upstream source at commit `c6491eb` (June 2026). Upstream changes over time, so if something here no longer matches, please [open an issue](../../issues).

The examples are starting points. Which distribution and s1panel version they were tested on hasn't been recorded yet.

## Credits

All of the hard work (reverse-engineering the panel and writing s1panel) is [tjaworski's](https://github.com/tjaworski/AceMagic-S1-LED-TFT-Linux). This repository is just a customization guide on top of it.
