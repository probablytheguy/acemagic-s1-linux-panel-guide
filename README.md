# acemagic-s1-linux-panel-guide
 A beginner-friendly guide for customizing the AceMagic S1 front LCD on Linux.
Customize the AceMagic S1 Front Display on Linux
This guide explains how to turn the AceMagic S1 front LCD into either:

a custom static display; or
a lightweight dashboard for CPU, temperature, memory, disk, uptime, and other system information.
It is based on the open-source AceMagic S1 LED/TFT Linux project.

Important

The S1 LCD has a native resolution of 320 × 170 pixels. It is a USB HID display, not a normal Linux monitor or framebuffer. Keep all LCD communication inside s1panel; do not run a second program that tries to control the same USB device.

Before you begin
This guide assumes:

s1panel is already installed and working;
its files are located at $HOME/acemagic/s1panel;
its systemd service is named s1panel.service; and
the normal display works before you begin customizing it.
Check the installation:

test -d "$HOME/acemagic/s1panel" && echo "s1panel directory found"
systemctl status s1panel --no-pager
If your installation uses a different directory or service name, substitute those values throughout this guide.

Values you are expected to customize
These are the main values used in the examples:

Value	Example	Change it to
Linux account	YOUR_USERNAME	Your actual Linux username
S1 address	YOUR_ACEMAGIC_IP	The IP address or hostname of your S1
Theme name	my-panel	A short name for your custom theme
Source image	my-background.jpg	The filename of your image
Panel title	MY S1	The title you want displayed
To find your Linux username, run:

whoami
1. Back up the working configuration
Always make a timestamped backup before editing anything:

cd "$HOME/acemagic/s1panel"
mkdir -p backups

STAMP="$(date +%Y%m%d-%H%M%S)"
cp -a config.json "backups/config.json-$STAMP"
cp -a themes "backups/themes-$STAMP"
Do not skip this step. It gives you a quick way to return to the last working display.

2. Create a custom theme folder
Create a separate theme instead of changing the factory theme directly:

mkdir -p "$HOME/acemagic/s1panel/themes/my-panel"
Using your own folder keeps upgrades and troubleshooting much simpler.

3. Prepare a background image
Copy your photo or artwork into the S1's Downloads folder. This example expects:

$HOME/Downloads/my-background.jpg
Use ImageMagick to resize and center-crop it to exactly 320 × 170:

convert "$HOME/Downloads/my-background.jpg" \
  -auto-orient \
  -resize '320x170^' \
  -gravity center \
  -extent 320x170 \
  "$HOME/acemagic/s1panel/themes/my-panel/background.png"
This preserves the aspect ratio while cropping the edges as needed to fill the screen.

Verify the result:

identify "$HOME/acemagic/s1panel/themes/my-panel/background.png"
The reported dimensions should be 320x170.

If convert or identify is not found, install ImageMagick using your distribution's package manager. On some newer ImageMagick installations, the equivalent command starts with magick.

4. Create a static-image theme
Create this file:

$HOME/acemagic/s1panel/themes/my-panel/theme.json
Use the following starting point, replacing YOUR_USERNAME with the result of whoami:

{
  "orientation": "landscape",
  "refresh": "redraw",
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
The full path is intentional. It prevents confusion about which directory is used when the systemd service starts the application.

Validate the JSON before continuing:

python3 -m json.tool \
  "$HOME/acemagic/s1panel/themes/my-panel/theme.json" >/dev/null \
  && echo "Theme JSON is valid"
5. Select the custom theme
Open the main configuration:

nano "$HOME/acemagic/s1panel/config.json"
Find its theme setting and point it at your new theme:

"theme": "themes/my-panel/theme.json"
Only replace the existing value. Keep the surrounding JSON and its commas intact.

Validate the main configuration:

python3 -m json.tool "$HOME/acemagic/s1panel/config.json" >/dev/null \
  && echo "Main configuration is valid"
Restart the panel service:

sudo systemctl restart s1panel
systemctl status s1panel --no-pager
The custom image should now appear on the front LCD.

6. Add a system dashboard
The exact widget editor and available widget properties can vary by s1panel version. Use the project's included web interface when possible instead of copying an unverified theme from another machine.

Typical sensor sources include:

clock
calendar
cpu_usage
cpu_temp
memory
space
Typical display widgets include:

text
custom bars
line charts
doughnut charts
bar charts
images
If the editor is listening on port 8686, open it locally at:

http://127.0.0.1:8686
For an S1 without a keyboard or monitor, create an SSH tunnel from another computer:

ssh -L 8686:127.0.0.1:8686 YOUR_USERNAME@YOUR_ACEMAGIC_IP
Keep that terminal open, then visit this address on the other computer:

http://127.0.0.1:8686
This makes the editor available without exposing it to the entire network.

Suggested 320 × 170 layout
Because the screen is small, use short labels and large, high-contrast values.

Area	Suggested position	Content
Header	top, y ≈ 8–30	MY S1
Upper left	x ≈ 12, y ≈ 55	CPU usage
Upper right	x ≈ 170, y ≈ 55	CPU temperature
Lower left	x ≈ 12, y ≈ 100	Memory usage
Lower right	x ≈ 170, y ≈ 100	Disk usage
Footer	x ≈ 12, y ≈ 140	Uptime or clock
Keep important text away from the extreme edges. Test readability on the physical LCD because fine lines and small fonts may look different than they do in a browser preview.

Refresh-rate guidance
The LCD receives image data over USB HID. Full-screen transfers are relatively slow, so faster is not always smoother.

Static background: update only when something changes.
Basic statistics: every 15–30 seconds is normally enough.
Clock: every 30–60 seconds, depending on whether seconds are displayed.
Animation: use changed regions when supported and drop late frames rather than building a queue.
7. Multiple dashboard screens
Themes may contain multiple entries in the screens array. Each screen can use a different wallpaper, widgets, and duration. This can be useful for rotating between:

a general overview;
temperatures and resource use; and
storage, network, or service information.
Start with one screen. Add additional screens only after the first one works reliably.

Troubleshooting
The display did not change
Check the service and its recent log messages:

systemctl status s1panel --no-pager
journalctl -u s1panel -n 100 --no-pager
Also confirm:

both JSON files pass python3 -m json.tool;
the wallpaper path contains the correct username;
the image is exactly 320 × 170;
the service account can read the theme and image; and
no second application is trying to control the LCD.
The image is stretched or has empty borders
Repeat the ImageMagick conversion using -resize '320x170^', followed by -extent 320x170. The caret means “fill the entire target area,” while -extent performs the center crop.

The service fails after an edit
Restore the backed-up main configuration. Replace TIMESTAMP with the timestamp shown in your backup filename:

cp "$HOME/acemagic/s1panel/backups/config.json-TIMESTAMP" \
  "$HOME/acemagic/s1panel/config.json"
sudo systemctl restart s1panel
If necessary, point config.json back to the original theme and restart again.

Good project practices
Keep custom work in a new theme directory.
Back up known-good files before each major change.
Validate JSON before restarting the service.
Use the correct native resolution: 320 × 170.
Avoid frequent full-screen redraws.
Let only the existing s1panel process own the LCD USB device.
Keep passwords, private IP addresses, API keys, and personal hostnames out of public GitHub files.
Add screenshots and clearly state which Linux distribution and s1panel version you tested.
Credits and further reading
Driver, installation, and project details are available from the upstream repository:

tjaworski/AceMagic-S1-LED-TFT-Linux
Review the upstream documentation for installation instructions and any configuration changes introduced after this guide was written.
