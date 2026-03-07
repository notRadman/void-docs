# 10 — Troubleshooting

---

## سواي مش بيشتغل

### "Failed to create backend" أو "No backend was able to open a seat"

```bash
# تأكد من elogind أو seatd
sudo sv status elogind
sudo sv status seatd

# لو seatd، تأكد من الـ group
groups | grep _seatd

# تأكد من dbus
sudo sv status dbus
```

### سواي بيبدأ وبيوقف بسرعة

```bash
# شغّله يدوياً وشوف الـ error
dbus-run-session sway 2>&1 | head -30
```

---

## XDG_RUNTIME_DIR

### "XDG_RUNTIME_DIR is not set"

```bash
# لو شغّال elogind
sudo sv status elogind

# حل يدوي مؤقت
export XDG_RUNTIME_DIR=/run/user/$(id -u)
sudo mkdir -p $XDG_RUNTIME_DIR
sudo chown $USER:$USER $XDG_RUNTIME_DIR
sudo chmod 0700 $XDG_RUNTIME_DIR

# حل دائم: pam_rundir (مع seatd)
sudo xbps-install pam_rundir
# أضف في /etc/pam.d/system-login:
# -session optional pam_rundir.so
```

---

## الصوت

### PipeWire مش شغّال

```bash
# تأكد من dbus
sudo sv status dbus

# تأكد من WirePlumber config
ls ~/.config/pipewire/pipewire.conf.d/

# شغّله يدوياً للتجربة
pipewire &
pactl info

# تأكد إن exec pipewire موجود في كونفيج سواي
grep pipewire ~/.config/sway/config
```

### Bluetooth Headphones مش بتشتغل

```bash
sudo xbps-install libspa-bluetooth
# restart سواي
```

---

## Firefox مش شغّال على Wayland

```bash
echo $MOZ_ENABLE_WAYLAND
# المفروض: 1

# تجربة يدوية
MOZ_ENABLE_WAYLAND=1 firefox
```

---

## Screen Sharing مش شغّالة

```bash
# 1. تأكد من التثبيت
xbps-query xdg-desktop-portal-wlr

# 2. تأكد من portals config
cat ~/.config/xdg-desktop-portal/sway-portals.conf

# 3. تأكد من متغيرات الـ dbus
dbus-update-activation-environment --print-environment | grep XDG_CURRENT_DESKTOP
# المفروض: XDG_CURRENT_DESKTOP=sway

# 4. Restart portals يدوياً للتجربة
killall xdg-desktop-portal xdg-desktop-portal-wlr 2>/dev/null
/usr/libexec/xdg-desktop-portal-wlr &
/usr/libexec/xdg-desktop-portal -r &

# 5. اختبار في Firefox
# افتح: https://mozilla.github.io/webrtc-landing/gum_test.html
```

---

## Arabic/Arabic text مش بيظهر صح

```bash
sudo xbps-install font-arabic-misc noto-fonts-ttf
fc-cache -fv

# في Tilix: Preferences → Profile → Custom Font → Noto Sans Arabic
```

---

## برامج Qt بتظهر X11

```bash
echo $QT_QPA_PLATFORM
# المفروض: wayland

# تأكد من وجود qt5-wayland
sudo xbps-install qt5-wayland qt6-wayland
```

---

## Java Apps (شاشة رمادية فارغة)

```bash
echo $JAVA_AWT_WM_NONREPARENTING
# المفروض: 1

# أضف في ~/.profile
export _JAVA_AWT_WM_NONREPARENTING=1
```

---

## AppImage مش بيشتغل

```bash
# تأكد من FUSE
sudo xbps-install fuse
sudo modprobe fuse

# Permissions
chmod 755 ~/AppImages/app.AppImage

# بديل بدون FUSE
./app.AppImage --appimage-extract-and-run
```

---

## KVM مشاكل

```bash
# Services مش شغّالة
sudo sv status libvirtd
sudo sv up libvirtd

# KVM module مش محمّل
sudo modprobe kvm_intel   # أو kvm_amd

# Permission denied
groups | grep libvirt
sudo usermod -aG libvirt $USER
```

---

## Waybar مش بتشتغل أو بتتقفل

```bash
# شغّلها يدوياً لتشوف الـ error
waybar 2>&1

# تأكد من الـ config
cat ~/.config/waybar/config
```

---

## أوامر تشخيص عامة

```bash
# شوف الـ Wayland display
echo $WAYLAND_DISPLAY        # المفروض: wayland-1

# شوف الـ compositor
swaymsg -t get_version

# شوف الـ outputs
swaymsg -t get_outputs

# شوف الـ inputs
swaymsg -t get_inputs

# شوف الـ environment في dbus
dbus-update-activation-environment --print-environment

# logs سواي
journalctl --user -u sway   # لو شغّال elogind
# أو
cat ~/.local/share/sway/sway.log 2>/dev/null
```
