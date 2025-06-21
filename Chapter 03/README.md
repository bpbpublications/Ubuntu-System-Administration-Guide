
# Environments and Window Managers

This section covers the installation, configuration, and usage of the **i3 Window Manager** on Ubuntu. You'll learn how to set up i3, configure multiple monitors, customize shortcuts, set backgrounds, manage workspaces, and more.

---

## Install i3 Window Manager

To install i3, open a terminal and run:

```bash
sudo apt install i3
```

After installation:

1. Log out of your current session.
2. On the login screen, click the gear icon.
3. Select **i3** as the window manager.
4. Log in again.

---

## Basic i3 Usage

### Launch Applications

- **Application Launcher**: Press `Win + d`, type the app name (e.g., `firefox`, `brave`), and press `Enter`.
- **New Terminal**: Press `Win + Enter` to open a new terminal.

*i3 is a tiling window manager, so new windows will auto-tile by default.*

---

## Screen Configuration for i3

To configure DPI and cursor size, especially for high-resolution screens:

1. Navigate to your config directory:
   ```bash
   cd ~/.config
   ```

2. Create or edit the `.XResources` file:
   ```bash
   nano .XResources
   ```

3. Add the following:
   ```bash
   Xft.dpi: 220
   Xcursor.size: 30
   ```

4. Apply the configuration:
   ```bash
   xrdb ~/.XResources
   ```

5. Reload i3:
   ```text
   Win + Shift + r
   ```

> These values work well on 4K displays (e.g., Dell XPS), but adjust as needed for your screen.

---

## Screen Resolution and Multi-Monitor Setup

Use `xrandr` to configure monitors:

1. Check connected displays:
   ```bash
   xrandr
   ```

2. Enable a screen:
   ```bash
   xrandr --output <screen_name> --auto
   ```

3. Set resolution and screen position:
   ```bash
   xrandr --output <screen1> --mode 3840x2160 --right-of <screen2>
   ```

4. Disable an external screen:
   ```bash
   xrandr --output eDP-1 --auto --output DP-1 --off
   ```

---

### Multi-Monitor Setup Scripts

Create reusable scripts in `~/.config`.

**Laptop Only:**

```bash
#!/bin/bash
xrandr --output eDP-1 --auto --output DP-1 --off
```

**Office Setup:**

```bash
#!/bin/bash
xrandr --output DP-2 --mode 3840x2160 --right-of eDP-1
```

Make scripts executable:
```bash
chmod +x ~/.config/office-setup.sh
```

---

### Auto-Apply DPI Settings on Startup

Add to your i3 config file (`~/.config/i3/config`):

```bash
exec --no-startup-id xrdb ~/.XResources
```

---

## Extra i3 Configuration

### Sound Control Panel (Pavucontrol)

Install and bind sound settings:

```bash
sudo apt install pavucontrol
```

Add to i3 config:

```bash
bindsym $mod+XF86AudioMute exec pavucontrol
```

### Screenshot Keybindings

Install the screenshot tool:

```bash
sudo apt install gnome-screenshot
```

Add to i3 config:

```bash
bindsym Print exec gnome-screenshot
bindsym Control+Print exec gnome-screenshot -i
```

---

# Custom Shortcuts and Settings

## Edit i3 Config File

Open i3 config:

```bash
gedit ~/.config/i3/config
```

### Change Mod Key

Default is the Windows key (Mod4). To change:

```bash
set $mod Mod4  # Windows key
# or
set $mod Control
```

---

## Set a Background Image

1. Place the image in `~/.config/background/image.jpg`.
2. Install `feh`:
   ```bash
   sudo apt install feh
   ```
3. Add to i3 config:
   ```bash
   exec --no-startup-id feh --bg-fill ~/.config/background/image.jpg
   ```

Reload i3 with `Mod + Shift + r`.

---

## Lock Screen Setup

1. Install lock tools:
   ```bash
   sudo apt install xautolock i3lock
   ```

2. Add to i3 config:

```bash
# Auto-lock after 15 minutes
exec xautolock -time 15 -locker 'i3lock -i ~/.config/background/lock.png' &

# Manual lock
bindsym $mod+l exec i3lock -i ~/.config/background/lock.png
```

---

## Startup Applications: Network, Bluetooth

Add to i3 config:

```bash
exec --no-startup-id nm-applet
exec --no-startup-id blueman-applet
```

Install Bluetooth applet if needed:

```bash
sudo apt install blueman
```

---

## Autostart Apps in Workspaces

Define workspaces:

```bash
set $ws1 "1:com"
set $ws2 "2:term"
set $ws3 "3:web"
set $ws4 "4:code"
set $ws5 "5:media"
set $ws6 "6:vm"
set $ws7 "7:misc"
```

Launch apps on startup:

```bash
exec --no-startup-id i3-msg 'workspace 2:term; exec i3-sensible-terminal'
exec --no-startup-id i3-msg 'workspace 3:web; exec /snap/bin/brave'
exec --no-startup-id i3-msg 'workspace 1:com; exec /snap/bin/slack'
exec --no-startup-id i3-msg 'workspace 1:com; exec /snap/bin/discord'
exec --no-startup-id i3-msg 'workspace 4:code; exec /snap/bin/code'
```

Force apps to open in specific workspaces:

```bash
for_window [class="Slack"] move to workspace 1:com
for_window [class="discord"] move to workspace 1:com
for_window [class="Brave-browser"] move to workspace 3:web
```

---

# Communication Tools

Install chat applications:

```bash
sudo apt install slack
sudo apt install teams
sudo apt install discord
```

---

# Video and Streaming Tools

## VLC for Streaming

```bash
sudo apt install vlc
```

To stream from an IP camera:

```bash
vlc rtsp://10.100.0.90:554/s2
```

## OBS Studio for Live Streaming

```bash
sudo add-apt-repository ppa:obsproject/obs-studio
sudo apt update
sudo apt install ffmpeg obs-studio
```

Launch OBS and set up your scenes and streaming keys.

---

## Webcam Support

Install Cheese webcam viewer:

```bash
sudo apt install cheese
```

---

# File Syncing Tools

### Google Drive (via OpenDrive)
Install from Flathub:  
[OpenDrive on Flathub](https://flathub.org/apps/details/io.github.liberodark.OpenDrive)

### Dropbox
Install from official site:  
[Dropbox for Linux](https://www.dropbox.com/install-linux)

### Mega
[MEGA Desktop App](https://mega.io/desktop)

### Resilio Sync
[Resilio Sync](https://www.resilio.com/)

---

# Git Setup

Install Git:

```bash
sudo apt install git
```

Generate SSH key:

```bash
ssh-keygen
```

Add your key to GitHub:

```bash
cat ~/.ssh/id_rsa.pub
```

Clone a repository:

```bash
git clone https://github.com/bpbpublications/Ubuntu-Linux-in-30-days.git
```

---

# Install Visual Studio Code (VSCode)

Download the `.deb` file from [Visual Studio Code](https://code.visualstudio.com/) or install via terminal:

```bash
sudo dpkg -i <name-of-downloaded-file.deb>
sudo apt --fix-broken install
```

---

```