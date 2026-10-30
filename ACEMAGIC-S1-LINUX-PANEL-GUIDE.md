# Customize the AceMagic S1 Front Display on Linux

This guide explains how to turn the AceMagic S1 front LCD into either:

- a custom static display; or
- a lightweight dashboard for CPU, temperature, memory, disk, uptime, and other system information.

It is based on the open-source [AceMagic S1 LED/TFT Linux project](https://github.com/tjaworski/AceMagic-S1-LED-TFT-Linux).

> [!IMPORTANT]
> The S1 LCD has a native resolution of **320 × 170 pixels**. It is a USB HID display, not a normal Linux monitor or framebuffer. Keep all LCD communication inside `s1panel`; do not run a second program that tries to control the same USB device.

## Before you begin

This guide assumes:

- `s1panel` is already installed and working;
- its files are located at `$HOME/acemagic/s1panel`;
- its systemd service is named `s1panel.service`; and
- the normal display works before you begin customizing it.

Check the installation:

```bash
test -d "$HOME/acemagic/s1panel" && echo "s1panel directory found"
systemctl status s1panel --no-pager
```

That location and service name are what you get from upstream's "New Install" steps if you clone the project into your home directory as `acemagic` and run `sudo ./install` from `acemagic/s1panel`. If your installation uses a different directory or service name, substitute those values throughout this guide.

> [!NOTE]
> **Snap installs are different.** If you installed `s1panel` from the Snap Store, the configuration and themes live in `/root/snap/s1panel/current/`, and upstream says to stop the snap (`sudo snap stop s1panel`) before editing those files manually. The paths and `systemctl` commands in this guide assume the git install. This guide hasn't been checked against the snap.

You'll also need ImageMagick (for the background image) and `python3` (only to check your JSON).

## Values you are expected to customize

These are the main values used in the examples:

| Value | Example | Change it to |
|---|---|---|
| Linux account | `YOUR_USERNAME` | Your actual Linux username |
| S1 address | `YOUR_ACEMAGIC_IP` | The IP address or hostname of your S1 |
| Theme name | `my-panel` | A short name for your custom theme |
| Source image | `my-background.jpg` | The filename of your image |
| Panel title | `MY S1` | The title you want displayed |

To find your Linux username, run:

```bash
whoami
```

## 1. Back up the working configuration

Always make a timestamped backup before editing anything:

```bash
cd "$HOME/acemagic/s1panel"
mkdir -p backups

STAMP="$(date +%Y%m%d-%H%M%S)"
cp -a config.json "backups/config.json-$STAMP"
cp -a themes "backups/themes-$STAMP"
```

Do not skip this step. It gives you a quick way to return to the last working display.

Upstream also warns that upgrading (`git pull`) can overwrite your changes to `config.json` and the demo theme, so make a fresh backup before every upgrade too.

## 2. Create a custom theme folder

Create a separate theme instead of changing the factory theme directly:

```bash
mkdir -p "$HOME/acemagic/s1panel/themes/my-panel"
```

Using your own folder keeps upgrades and troubleshooting much simpler.

## 3. Prepare a background image

Copy your photo or artwork into the S1's `Downloads` folder. This example expects:

```text
$HOME/Downloads/my-background.jpg
```

Use ImageMagick to resize and center-crop it to exactly 320 × 170:

```bash
convert "$HOME/Downloads/my-background.jpg" \
  -auto-orient \
  -resize '320x170^' \
  -gravity center \
  -extent 320x170 \
  "$HOME/acemagic/s1panel/themes/my-panel/background.png"
```

This preserves the aspect ratio while cropping the edges as needed to fill the screen.

Verify the result:

```bash
identify "$HOME/acemagic/s1panel/themes/my-panel/background.png"
```

The reported dimensions should be `320x170`.

If `convert` or `identify` is not found, install ImageMagick using your distribution's package manager. ImageMagick 7 replaces `convert` with `magick` (same options) and `identify` with `magick identify`; on some systems `convert` still works but prints a deprecation warning.

## 4. Create a static-image theme

Create this file:

```text
$HOME/acemagic/s1panel/themes/my-panel/theme.json
```

Use the following starting point, replacing `YOUR_USERNAME` with the result of `whoami`:

```json
{
  "orientation": "landscape",
  "refresh": "update",
  "screens": [
    {
      "id": 1,
      "name": "My Panel",
      "background": "#000000",
      "duration": 0,
      "widgets": [],
      "wallpaper": "/home/YOUR_USERNAME/acemagic/s1panel/themes/my-panel/background.png"
    }
  ]
}
```

What the fields do, according to the upstream source:

- `orientation`: `landscape` or `portrait`. The theme sets the panel's orientation, overriding `portrait` in `config.json`.
- `refresh`: how changes are sent to the LCD. `update` sends only the regions that changed; `redraw` resends the whole screen on every cycle, which upstream's own comments call the slowest mode. Both modes draw the full screen once at startup. Earlier versions of this guide used `redraw` here.
- `duration`: how long the screen is shown before moving to the next, in milliseconds. `0` means it stays. Upstream keeps every screen up for at least 10 seconds.
- `widgets`: must be present, even if it's empty.

The full wallpaper path is intentional. The service resolves relative paths from its working directory, and it runs as root, so `$HOME` and `~` won't point at your home directory.

Validate the JSON before continuing:

```bash
python3 -m json.tool \
  "$HOME/acemagic/s1panel/themes/my-panel/theme.json" >/dev/null \
  && echo "Theme JSON is valid"
```

## 5. Select the custom theme

Open the main configuration:

```bash
nano "$HOME/acemagic/s1panel/config.json"
```

Find its `theme` setting and point it at your new theme:

```json
"theme": "themes/my-panel/theme.json"
```

Only replace the existing value. Keep the surrounding JSON and its commas intact. The path is relative to the `s1panel` directory.

Optionally, add your theme to the `theme_list` array in the same file so the web editor's theme picker can find it:

```json
"theme_list": [
   {
      "name": "a simple demo portrait orientation",
      "config": "themes/simple_demo/portrait_simple.json"
   },
   {
      "name": "My Panel",
      "config": "themes/my-panel/theme.json"
   }
]
```

Validate the main configuration:

```bash
python3 -m json.tool "$HOME/acemagic/s1panel/config.json" >/dev/null \
  && echo "Main configuration is valid"
```

Restart the panel service:

```bash
sudo systemctl restart s1panel
systemctl status s1panel --no-pager
```

The custom image should now appear on the front LCD.

## 6. Add a system dashboard

The exact widget editor and available widget properties can vary by `s1panel` version. Use the project's included web interface when possible instead of copying an unverified theme from another machine.

Sensors and widgets are modules listed in `config.json`. At the time of writing, upstream ships these:

| Sensors (`sensors/*.js`) | Widgets (`widgets/*.js`) |
|---|---|
| `clock`, `calendar` | `text` |
| `cpu_usage`, `cpu_temp`, `cpu_power` | `custom_bar` |
| `memory` | `line_chart`, `bar_chart`, `doughnut_chart` |
| `space` (disk usage per mount point) | `image`, `iconify` |
| `network` (per interface) | `weather_icon` |
| `weather` | |

Check the `sensors` and `widgets` arrays in your own `config.json` for what your version has. Some sensors need setup: `cpu_temp` needs lm_sensors installed and configured; `network` needs your interface name (the upstream example uses `enp2s0`, which may not match your S1, so check with `ip link`); and `weather` and `iconify` fetch data from the internet.

### Keep the editor off your network

The editor listens on whatever address the `listen` setting in `config.json` gives. **Check it before you use the editor.** The upstream README describes `localhost:8686` as the default, but the `config.json` shipped in the repository sets `"listen": "0.0.0.0:8686"`, which serves the editor to your whole network. The editor has no login, and it can change your configuration.

To keep it local-only, set:

```json
"listen": "127.0.0.1:8686"
```

Then restart the service:

```bash
sudo systemctl restart s1panel
```

On the S1 itself, open:

```text
http://127.0.0.1:8686
```

For an S1 without a keyboard or monitor, create an SSH tunnel from another computer:

```bash
ssh -L 8686:127.0.0.1:8686 YOUR_USERNAME@YOUR_ACEMAGIC_IP
```

Keep that terminal open and visit `http://127.0.0.1:8686` on the other computer. With `listen` set to `127.0.0.1`, the tunnel is the only way in from elsewhere.

### Suggested 320 × 170 layout

Because the screen is small, use short labels and large, high-contrast values. These positions are a starting point for a landscape layout, not a tested theme. Each widget's position goes in its `rect` (`x`, `y`, `width`, `height`, in pixels from the top-left).

| Area | Suggested position | Content |
|---|---:|---|
| Header | top, `y ≈ 8–30` | `MY S1` |
| Upper left | `x ≈ 12`, `y ≈ 55` | CPU usage |
| Upper right | `x ≈ 170`, `y ≈ 55` | CPU temperature |
| Lower left | `x ≈ 12`, `y ≈ 100` | Memory usage |
| Lower right | `x ≈ 170`, `y ≈ 100` | Disk usage |
| Footer | `x ≈ 12`, `y ≈ 140` | Uptime or clock |

Keep important text away from the extreme edges. Test readability on the physical LCD because fine lines and small fonts may look different than they do in a browser preview.

### Refresh-rate guidance

The LCD receives image data over USB HID. Full-screen transfers are relatively slow, so faster is not always smoother.

- Use `"refresh": "update"` in the theme so only changed widgets are sent. `redraw` resends the whole screen every cycle.
- Each widget has its own `refresh` value, **in milliseconds**. The upstream demo uses `1000`. For basic statistics, 15–30 seconds (`15000`–`30000`) is plenty. For a clock, use 30–60 seconds unless it shows seconds.
- Leave the device-level `poll`, `refresh` and `heartbeat` values in `config.json` alone unless you have a reason to change them.

## 7. Multiple dashboard screens

Themes may contain multiple entries in the `screens` array. Each screen can use a different wallpaper, widgets, and `duration` (milliseconds, at least 10 seconds in practice). Recent upstream versions also let you turn off a screen with `"enabled": false` without deleting it. This can be useful for rotating between:

1. a general overview;
2. temperatures and resource use; and
3. storage, network, or service information.

Start with one screen. Add additional screens only after the first one works reliably.

## Troubleshooting

### The display did not change

Check the service and its recent log messages:

```bash
systemctl status s1panel --no-pager
journalctl -u s1panel -n 100 --no-pager
```

Also confirm:

- both JSON files pass `python3 -m json.tool`;
- the wallpaper path is absolute and contains the correct username;
- the image is exactly 320 × 170;
- `config.json` points at the right theme file (a log line saying `failed to load` followed by the theme path means it couldn't read the theme); and
- no second application is trying to control the LCD.

### The image is stretched or has empty borders

Repeat the ImageMagick conversion using `-resize '320x170^'`, followed by `-extent 320x170`. The caret means “fill the entire target area,” while `-extent` performs the center crop.

### The service fails after an edit

Restore the backed-up main configuration. Replace `TIMESTAMP` with the timestamp shown in your backup filename:

```bash
cp "$HOME/acemagic/s1panel/backups/config.json-TIMESTAMP" \
  "$HOME/acemagic/s1panel/config.json"
sudo systemctl restart s1panel
```

If necessary, point `config.json` back to the original theme and restart again.

## Good project practices

- Keep custom work in a new theme directory.
- Back up known-good files before each major change.
- Validate JSON before restarting the service.
- Use the correct native resolution: 320 × 170.
- Avoid frequent full-screen redraws.
- Let only the existing `s1panel` process own the LCD USB device.
- Keep passwords, private IP addresses, API keys, and personal hostnames out of public GitHub files.
- Add screenshots and clearly state which Linux distribution and `s1panel` version you tested.

## Credits and further reading

`s1panel`, the LCD and LED protocol work, and the installation instructions are by the upstream project. It is licensed under GPL-3.0:

- [tjaworski/AceMagic-S1-LED-TFT-Linux](https://github.com/tjaworski/AceMagic-S1-LED-TFT-Linux)

This guide contains no upstream code. Field names and defaults were checked against upstream commit `c6491eb` (June 2026). Review the upstream documentation for installation instructions and any configuration changes introduced after that.
