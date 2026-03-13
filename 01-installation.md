# 01 — التثبيت الأساسي

---

## الخطوة 1 — تحديث النظام أولاً

```bash
sudo xbps-install -Su
```

---

## الخطوة 2 — أدوات البناء الأساسية

```bash
sudo xbps-install base-devel git make ca-certificates wget curl
```

---

## الخطوة 3 — GPU Drivers (Mesa بدون Xorg)

> ⚠️ على Wayland مش محتاج `xorg-server` أو `xf86-video-intel`.

### Intel (مثل Dell Latitude E6430)
```bash
sudo xbps-install mesa-dri intel-video-accel \
  libva-utils mesa-vaapi mesa-vdpau \
  vulkan-loader mesa-vulkan-intel
```

### AMD
```bash
sudo xbps-install mesa-dri mesa-vaapi mesa-vdpau \
  vulkan-loader mesa-vulkan-radeon
```

### NVIDIA
```bash
sudo xbps-install nvidia nvidia-libs
```

---

## الخطوة 4 — Session Manager (اختار واحد بس!)

### الخيار A: elogind (الأنسب للأجهزة اللاب)

```bash
sudo xbps-install elogind
sudo ln -s /etc/sv/elogind /var/service/
```

**مميزات elogind:**
- بيعمل `XDG_RUNTIME_DIR` تلقائياً
- بيدعم power management (suspend، hibernate)
- أكثر استقراراً مع معظم البرامج

### الخيار B: seatd (أخف وأبسط)

```bash
sudo xbps-install seatd
sudo ln -s /etc/sv/seatd /var/service/
sudo usermod -aG _seatd $USER
```

**مميزات seatd:**
- أخف بكتير
- بيدير الـ seat بس (مش XDG_RUNTIME_DIR)
- محتاج `pam_rundir` معاه

> ⚠️ اختار واحد بس — elogind و seatd مش بيشتغلوا مع بعض.

---

## الخطوة 5 — XDG_RUNTIME_DIR

### لو اخترت elogind
مش محتاج حاجة — بيتعمل تلقائياً.

### لو اخترت seatd
```bash
sudo xbps-install pam_rundir
```

أضف في `/etc/pam.d/system-login`:
```
-session optional pam_rundir.so
```

---

## الخطوة 6 — D-Bus

```bash
sudo xbps-install dbus
sudo ln -s /etc/sv/dbus /var/service/
```

---

## الخطوة 7 — المجلدات الافتراضية

```bash
sudo xbps-install xdg-user-dirs
xdg-user-dirs-update
```

---

## الخطوة 8 — Sway والأدوات الأساسية

```bash
sudo xbps-install \
  sway swaybg swayidle swaylock \
  xorg-server-xwayland \
  wlroots wl-clipboard
```

### Terminal
```bash
sudo xbps-install tilix
# أو foot (أخف وأسرع)
sudo xbps-install foot
```

### App Launcher
```bash
sudo xbps-install fuzzel
# أو wofi
sudo xbps-install wofi
```

### Status Bar
```bash
sudo xbps-install waybar
```

### Notifications
```bash
sudo xbps-install mako libnotify
```

### Screenshot
```bash
sudo xbps-install grim slurp
```

### Screen Recording
```bash
sudo xbps-install wf-recorder
```

### Clipboard Manager
```bash
sudo xbps-install clipman
```

---

## الخطوة 9 — XDG Desktop Portals (ضروري لـ Screen Sharing)

```bash
sudo xbps-install \
  xdg-desktop-portal \
  xdg-desktop-portal-wlr \
  xdg-desktop-portal-gtk \
  xdg-utils
```

### ملف إعداد الـ Portals (مهم جداً!)

```bash
mkdir -p ~/.config/xdg-desktop-portal
nano ~/.config/xdg-desktop-portal/sway-portals.conf
```

```ini
[preferred]
default=gtk
org.freedesktop.impl.portal.Screenshot=wlr
org.freedesktop.impl.portal.ScreenCast=wlr
```

> اسم الملف لازم يكون `sway-portals.conf` بالظبط لأن `XDG_CURRENT_DESKTOP=sway`.

---

## الخطوة 10 — Media Codecs

```bash
sudo xbps-install \
  ffmpeg \
  gst-plugins-base1 \
  gst-plugins-good1 \
  gst-plugins-bad1 \
  gst-plugins-ugly1
```

---

## الخطوة 11 — Power Management (للاب)

```bash
sudo xbps-install tlp acpid acpi brightnessctl
sudo ln -s /etc/sv/acpid /var/service/
sudo ln -s /etc/sv/tlpd /var/service/
sudo usermod -aG video $USER
```

### إعداد TLP (مهم لـ WiFi stability)

```bash
sudo nano /etc/tlp.conf
```

ابحث عن وغيّر:
```
WIFI_PWR_ON_AC=off
WIFI_PWR_ON_BAT=off
```

---

## الخطوة 12 — Fonts

```bash
sudo xbps-install \
  jq \
  liberation-fonts-ttf \
  dejavu-fonts-ttf \
  noto-fonts-ttf \
  noto-fonts-ttf-extra \
  noto-fonts-emoji \
  font-ibm-plex-ttf \
  terminus-font \
  font-awesome6
```

> **JetBrains Mono:** مش في الريبو، تنزله يدوي من [jetbrains.com/lp/mono](https://www.jetbrains.com/lp/mono/) وتحطه في `~/.local/share/fonts/`

---

## الخطوة 13 — متغيرات البيئة

أضف في `~/.bash_profile` أو `~/.profile`:

```bash
# XDG Base Directories
export XDG_CONFIG_HOME="$HOME/.config"
export XDG_DATA_HOME="$HOME/.local/share"
export XDG_CACHE_HOME="$HOME/.cache"
export XDG_STATE_HOME="$HOME/.local/state"
export PATH="$HOME/.local/bin:$PATH"

# XDG_RUNTIME_DIR (لو مش شغّال elogind)
if test -z "${XDG_RUNTIME_DIR}"; then
    export XDG_RUNTIME_DIR=/tmp/${UID}-runtime-dir
    if ! test -d "${XDG_RUNTIME_DIR}"; then
        mkdir "${XDG_RUNTIME_DIR}"
        chmod 0700 "${XDG_RUNTIME_DIR}"
    fi
fi

# Wayland
export XDG_SESSION_TYPE=wayland
export XDG_SESSION_DESKTOP=sway
export XDG_CURRENT_DESKTOP=sway
export DE=sway

# Apps
export MOZ_ENABLE_WAYLAND=1
export QT_QPA_PLATFORM=wayland
export QT_WAYLAND_DISABLE_WINDOWDECORATION=1
export SDL_VIDEODRIVER=wayland
export ELECTRON_OZONE_PLATFORM_HINT=auto
export _JAVA_AWT_WM_NONREPARENTING=1

# لو بتستخدم seatd
export LIBSEAT_BACKEND=seatd

# Auto-start Sway من tty1
if [ -z "$DISPLAY" ] && [ -z "$WAYLAND_DISPLAY" ] && [ "${XDG_VTNR:-0}" -eq 1 ]; then
    exec dbus-run-session sway
fi
```

---

## الخطوة 14 — نسخ الكونفيج الافتراضي

```bash
mkdir -p ~/.config/sway
cp /etc/sway/config ~/.config/sway/config
```

---

## الخطوة 15 — إضافات أساسية في كونفيج سواي

افتح `~/.config/sway/config` وأضف:

```
### أول سطر — تحديث بيئة dbus ###
exec dbus-update-activation-environment DISPLAY WAYLAND_DISPLAY SWAYSOCK XDG_CURRENT_DESKTOP

### XDG Desktop Portal ###
exec /usr/libexec/xdg-desktop-portal-wlr &
exec /usr/libexec/xdg-desktop-portal -r &

### PipeWire ###
exec pipewire &

### Notifications ###
exec mako &

### Polkit ###
exec /usr/libexec/polkit-gnome-authentication-agent-1 &

### Network ###
exec nm-applet --indicator &

### Bluetooth ###
exec blueman-applet &

### Clipboard ###
exec wl-paste -t text --watch clipman store &

### USB Auto-mount ###
exec udiskie --tray &

### Keyboard Layout ###
input type:keyboard {
    xkb_layout us,ara
    xkb_options grp:lwin_toggle
    repeat_delay 300
    repeat_rate 50
}

### Touchpad ###
input type:touchpad {
    tap enabled
    natural_scroll enabled
    dwt enabled
}

### Shortcuts ###
# Screenshot
bindsym Print exec grim ~/Pictures/screenshot-$(date +%Y%m%d-%H%M%S).png
bindsym Shift+Print exec grim -g "$(slurp)" ~/Pictures/screenshot-$(date +%Y%m%d-%H%M%S).png
bindsym Ctrl+Print exec grim -g "$(slurp)" - | wl-copy

# Volume
bindsym XF86AudioRaiseVolume exec pactl set-sink-volume @DEFAULT_SINK@ +5%
bindsym XF86AudioLowerVolume exec pactl set-sink-volume @DEFAULT_SINK@ -5%
bindsym XF86AudioMute exec pactl set-sink-mute @DEFAULT_SINK@ toggle

# Brightness
bindsym XF86MonBrightnessUp exec brightnessctl set +10%
bindsym XF86MonBrightnessDown exec brightnessctl set 10%-

# Lock screen
bindsym $mod+l exec swaylock -f -c 000000

### Idle + Lock ###
exec swayidle -w \
    timeout 300 'swaylock -f -c 000000' \
    timeout 600 'swaymsg "output * dpms off"' \
    resume 'swaymsg "output * dpms on"' \
    before-sleep 'swaylock -f -c 000000'
```

---

## تحقق نهائي

```bash
# Services
sv status dbus
sv status elogind   # أو seatd

# Groups
groups $USER

# XDG_RUNTIME_DIR
echo $XDG_RUNTIME_DIR

# Wayland
echo $WAYLAND_DISPLAY   # بعد تشغيل سواي: wayland-1

# PipeWire
pactl info | grep "Server Name"

# Screenshot
grim ~/test.png
```
