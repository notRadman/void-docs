# 04 — Screenshot + Screen Recording + Clipboard

---

## الأدوات

```bash
sudo xbps-install grim slurp wf-recorder wl-clipboard clipman
```

---

## Screenshot

### أوامر مباشرة

```bash
# شاشة كاملة
grim ~/Pictures/screenshot-$(date +%Y%m%d-%H%M%S).png

# منطقة معينة
grim -g "$(slurp)" ~/Pictures/screenshot-$(date +%Y%m%d-%H%M%S).png

# نسخ مباشرة للـ clipboard
grim -g "$(slurp)" - | wl-copy
```

### في كونفيج سواي

```
bindsym Print exec grim ~/Pictures/screenshot-$(date +%Y%m%d-%H%M%S).png
bindsym Shift+Print exec grim -g "$(slurp)" ~/Pictures/screenshot-$(date +%Y%m%d-%H%M%S).png
bindsym Ctrl+Print exec grim -g "$(slurp)" - | wl-copy
```

---

## Screen Recording — wf-recorder

```bash
# شاشة كاملة
wf-recorder -f ~/Videos/recording.mp4

# منطقة معينة
wf-recorder -g "$(slurp)" -f ~/Videos/recording.mp4

# مع الصوت
wf-recorder -a -f ~/Videos/recording.mp4

# مع صوت device معين
pactl list sources short          # شوف الـ devices
wf-recorder -a "اسم_السورس" -f ~/Videos/recording.mp4

# إيقاف
Ctrl+C
```

---

## Clipboard Manager — clipman

### تشغيل (في كونفيج سواي)

```
exec wl-paste -t text --watch clipman store &
```

### عرض الـ history

```bash
clipman pick --tool wofi
# أو
clipman pick --tool fuzzel
```

أضف في كونفيج سواي:
```
bindsym $mod+shift+v exec clipman pick --tool fuzzel
```

### مسح الـ history

```bash
clipman clear --all
```
