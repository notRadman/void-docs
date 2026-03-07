# 03 — الـ Theming (شكل موحد)

---

## المفهوم الأساسي

الـ theme على Wayland = 4 طبقات منفصلة:

| الطبقة | التأثير |
|---|---|
| GTK Theme | ألوان وشكل الـ buttons والـ windows |
| Icon Theme | أيقونات الملفات والبرامج |
| Cursor Theme | شكل المؤشر |
| Font | الخط في كل البرامج |

**المشكلة:** على Wayland، GTK مش بيقرأ الإعدادات تلقائياً — لازم تتحط في `gsettings`.
**الحل:** أداة `nwg-look`.

---

## التثبيت

```bash
sudo xbps-install nwg-look
```

### ثيمات GTK

```bash
# Materia (flat, modern)
sudo xbps-install materia-theme

# Arc
sudo xbps-install arc-theme

# Adwaita:dark — موجود تلقائياً مع GTK
```

لو عايز ثيم من النت (مثال Catppuccin):
```bash
mkdir -p ~/.local/share/themes
cd ~/.local/share/themes
# نزّل وفكّ الـ zip هنا
# nwg-look هيشوفه تلقائياً
```

### Icon Theme

```bash
sudo xbps-install papirus-icon-theme
```

### Cursor Theme

```bash
sudo xbps-install breeze-cursors
```

---

## التطبيق

```bash
nwg-look
# اختار GTK Theme + Icons + Cursor + Font
# اضغط Apply
```

---

## Cursor في كونفيج سواي (خطوة مهمة!)

أضف في `~/.config/sway/config`:
```
seat seat0 xcursor_theme اسم_الثيم 24
```

---

## الفونتات

```bash
sudo xbps-install \
  font-jetbrains-mono \
  noto-fonts-ttf \
  noto-fonts-ttf-extra \
  noto-fonts-emoji \
  font-awesome6 \
  dejavu-fonts-ttf

# تحديث cache بعد أي font جديد
fc-cache -fv
```

### دعم العربية

```bash
sudo xbps-install font-arabic-misc noto-fonts-ttf
```

في كونفيج سواي:
```
font pango:JetBrains Mono 11, Noto Sans Arabic 11, Font Awesome 6 Free 11
```

في Tilix: Preferences → Profile → Custom Font → اختار خط بيدعم عربي (مثل Noto Sans Arabic)

---

## Tilix — Color Scheme منفصل

Tilix عنده color scheme خاص مش مرتبط بالـ GTK theme.

```
Tilix → Preferences → Profile → Color → اختار scheme
```

إضافة scheme جديد (مثال Catppuccin):
```bash
mkdir -p ~/.config/tilix/schemes
# نزّل الـ .json وحطه هنا
```

---

## مكان الثيمات

```bash
# GTK themes
/usr/share/themes/
~/.local/share/themes/

# Icons + Cursors
/usr/share/icons/
~/.local/share/icons/

# شوف المثبت
ls /usr/share/themes/
ls /usr/share/icons/
fc-list | grep -i "jet\|noto\|arabic"
```
