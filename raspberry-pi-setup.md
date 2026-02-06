# Raspberry Pi Kiosk Setup

This guide sets up a Raspberry Pi to run WeatherStar 4000+ International as an unattended kiosk display — fullscreen browser, auto-playing audio, and automatic startup.

## Prerequisites

- Raspberry Pi 4 or 5
- Raspberry Pi OS **Desktop** (64-bit recommended) — you need a display server
- Monitor connected via HDMI
- Network connection

## 1. Install Node.js

```bash
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo bash -
sudo apt install -y nodejs
```

## 2. Clone and install

```bash
cd /home/pi
git clone https://github.com/pavelruzicka/ws4kp-international.git
cd ws4kp-international
npm install
```

## 3. Systemd service for the app

Create `/etc/systemd/system/ws4kp.service`:

```ini
[Unit]
Description=WeatherStar 4000+
After=network.target

[Service]
Type=simple
User=pi
WorkingDirectory=/home/pi/ws4kp-international
ExecStart=/usr/bin/npm run start
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

Enable and start:

```bash
sudo systemctl enable ws4kp
sudo systemctl start ws4kp
```

## 4. Chromium kiosk on boot

Create `/home/pi/.config/autostart/kiosk.desktop`:

```ini
[Desktop Entry]
Type=Application
Name=Kiosk
Exec=/usr/bin/chromium-browser --kiosk --noerrdialogs --disable-infobars --disable-session-crashed-bubble --autoplay-policy=no-user-gesture-required --check-for-update-interval=31536000 http://localhost:8080
```

Key flags:
- `--kiosk` — fullscreen, no browser chrome
- `--autoplay-policy=no-user-gesture-required` — allows audio to auto-play
- `--noerrdialogs --disable-infobars --disable-session-crashed-bubble` — suppresses popups
- `--check-for-update-interval=31536000` — disables update prompts

## 5. Prevent screen blanking

```bash
sudo apt install -y xdotool
```

Add the following to `/etc/xdg/lxsession/LXDE-pi/autostart` (create it if it doesn't exist):

```
@xset s off
@xset -dpms
@xset s noblank
```

## 6. Audio

Set the audio output and volume:

```bash
# Pick HDMI or 3.5mm jack
sudo raspi-config
# → System Options → Audio → select output

# Set volume to max
amixer set Master 100%
```

## 7. Nightly reboot

A nightly reboot keeps things fresh and avoids memory leaks or other long-running issues:

```bash
echo "0 4 * * * /sbin/reboot" | sudo crontab -
```

This reboots the Pi at 4:00 AM every day.
