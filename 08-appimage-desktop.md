# 08 — AppImage + Desktop Entries

---

## هيكل المجلدات

```
~/AppImages/
├── obsidian.AppImage
├── ticktick.AppImage
└── icons/
    ├── obsidian.png
    └── ticktick.png

~/.local/share/applications/    # Desktop entries
~/.config/mimeapps.list         # MIME associations
```

---

## إعداد AppImage جديد (خطوات سريعة)

```bash
# 1. أنشئ المجلدات
mkdir -p ~/AppImages/icons
mkdir -p ~/.local/share/applications

# 2. انقل الـ AppImage
mv downloaded.AppImage ~/AppImages/appname.AppImage
chmod 755 ~/AppImages/appname.AppImage

# 3. حط الأيقونة
cp icon.png ~/AppImages/icons/appname.png
chmod 644 ~/AppImages/icons/appname.png
```

---

## Desktop Entry Template

```bash
nano ~/.local/share/applications/appname.desktop
```

```ini
[Desktop Entry]
Version=1.0
Type=Application
Name=اسم_البرنامج
Comment=وصف قصير
Exec=/home/USERNAME/AppImages/appname.AppImage %F
Icon=/home/USERNAME/AppImages/icons/appname.png
Terminal=false
Categories=Office;Utility;
MimeType=text/markdown;text/plain;
StartupNotify=true
```

```bash
chmod 644 ~/.local/share/applications/appname.desktop
update-desktop-database ~/.local/share/applications
```

---

## Exec Variables

| المتغير | المعنى |
|---|---|
| `%f` | ملف واحد |
| `%F` | عدة ملفات |
| `%u` | URL واحد |
| `%U` | عدة URLs |

---

## الأيقونات

**الصيغ المدعومة:** PNG (الأنسب)، SVG (الأفضل)، XPM، ICO
**لا تستخدم:** JPG/JPEG

**الحجم:** 128×128 على الأقل — SVG أفضل لأنه بيتكيف مع أي حجم.

---

## MIME Associations

### إضافة MIME لـ Desktop Entry

```ini
[Desktop Entry]
Name=Obsidian
Exec=/home/USERNAME/AppImages/obsidian.AppImage %F
MimeType=text/markdown;text/plain;
```

### تحديد التطبيق الافتراضي

```bash
# بالأمر
xdg-mime default obsidian.desktop text/markdown

# أو في الملف يدوياً
nano ~/.config/mimeapps.list
```

```ini
[Default Applications]
text/markdown=obsidian.desktop
text/plain=obsidian.desktop
application/pdf=pdfviewer.desktop

[Added Associations]
text/markdown=obsidian.desktop;
```

### أهم MIME Types

```
text/plain        →  .txt
text/markdown     →  .md
text/html         →  .html
application/pdf   →  .pdf
image/png         →  .png
image/jpeg        →  .jpg
```

### أوامر مفيدة

```bash
# اعرف MIME type لملف
xdg-mime query filetype document.md

# اعرف التطبيق الافتراضي
xdg-mime query default text/markdown
```

---

## مثال كامل: Obsidian

```bash
mkdir -p ~/AppImages/icons
mv Obsidian-*.AppImage ~/AppImages/obsidian.AppImage
chmod 755 ~/AppImages/obsidian.AppImage
# نزّل الأيقونة أو استخرجها من الـ AppImage
cp obsidian-icon.png ~/AppImages/icons/obsidian.png
chmod 644 ~/AppImages/icons/obsidian.png
```

```ini
# ~/.local/share/applications/obsidian.desktop
[Desktop Entry]
Version=1.0
Type=Application
Name=Obsidian
Comment=Knowledge base and note-taking app
Exec=/home/USERNAME/AppImages/obsidian.AppImage %F
Icon=/home/USERNAME/AppImages/icons/obsidian.png
Terminal=false
Categories=Office;TextEditor;Utility;
MimeType=text/markdown;text/plain;
StartupNotify=true
```

```bash
chmod 644 ~/.local/share/applications/obsidian.desktop
update-desktop-database ~/.local/share/applications
xdg-mime default obsidian.desktop text/markdown
```

---

## Web App كـ Desktop Entry

```ini
[Desktop Entry]
Version=1.0
Type=Application
Name=Gmail
Exec=firefox --new-window https://mail.google.com
Icon=/home/USERNAME/AppImages/icons/gmail.png
Terminal=false
Categories=Network;WebBrowser;
StartupNotify=true
```

---

## تحديث AppImage

```bash
wget -O /tmp/new.AppImage https://download-url
chmod 755 /tmp/new.AppImage
/tmp/new.AppImage       # جرّبه أول
mv /tmp/new.AppImage ~/AppImages/app.AppImage
# الـ desktop entry بيفضل كما هو
```

---

## Troubleshooting

```bash
# Desktop entry مش بيظهر
desktop-file-validate ~/.local/share/applications/app.desktop
chmod 644 ~/.local/share/applications/app.desktop
update-desktop-database ~/.local/share/applications

# AppImage مش بيشتغل
chmod 755 ~/AppImages/app.AppImage
sudo xbps-install fuse   # على Void

# أيقونة مش بتظهر
ls -l ~/AppImages/icons/app.png    # تأكد إنها موجودة
# استخدم PNG مش JPG

# MIME مش بيشتغل
update-desktop-database ~/.local/share/applications
update-mime-database ~/.local/share/mime
xdg-mime default app.desktop text/markdown
```

---

## Quick Reference

```bash
chmod +x file.AppImage
chmod 644 file.desktop
update-desktop-database ~/.local/share/applications
xdg-mime default app.desktop text/markdown
xdg-mime query default text/markdown
xdg-mime query filetype file.md
gtk-launch appname          # تجربة الـ desktop entry
xdg-open file.md            # فتح بالتطبيق الافتراضي
desktop-file-validate app.desktop
```
