# 14 — User Services على Void Linux

سيرفسز خاصة بيوزر معين بدون صلاحيات root.

---

## الفكرة

runit على Void بيشتغل على مستويين:

- **System services** — في `/var/service/` — بتشتغل عند الـ boot كـ root
- **User services** — في `~/path/to/servicesFolder/` — بتشتغل مع الجلسة بتاعت اليوزر

---

## الطريقة الأولى — System-level service يشغّل runsvdir لليوزر

### الخطوة 1: إنشاء الـ service

```bash
sudo mkdir -p /etc/sv/runsvdir-{your-username}
sudo nano /etc/sv/runsvdir-YOUR-{your-username}/run
```

محتوى الملف:

```sh
#!/bin/sh
export USER="YOUR-USERNAME"
export HOME="$HOME"
groups="$(id -Gn "$USER" | tr ' ' ':')"
svdir="$HOME/path/to/servicesFolder"
exec chpst -u "$USER:$groups" runsvdir "$svdir"
```

```bash
sudo chmod +x /etc/sv/runsvdir-YOUR-USERNAME/run
```

### الخطوة 2: إنشاء مجلد الـ services

```bash
mkdir -p ~/path/to/servicesFolder
```

### الخطوة 3: تفعيله

```bash
sudo ln -s /etc/sv/runsvdir-{your-username} /var/service/
```

### ⚠️ قيود هذه الطريقة

- بتشتغل عند الـ boot — **قبل** ما الجلسة الجرافيكية تبدأ
- مش بتشوف `WAYLAND_DISPLAY` أو `DBUS_SESSION_BUS_ADDRESS`
- مناسبة للـ daemons اللي مش محتاجة الجلسة (syncthing، backup scripts، إلخ)
- **مش** مناسبة لبرامج الـ GUI

---

## الطريقة الثانية — تشغيل مع الجلسة (عن طريق سواي)

للسيرفسز اللي محتاجة الجلسة الجرافيكية — أضف في كونفيج سواي:

```bash
exec runsvdir ~/path/to/servicesFolder &
```

بعدين حط السيرفسز في `~/path/to/service/`.

### مثال — syncthing كـ user service

```bash
mkdir -p ~/path/to/servicesFolder/syncthing
nano ~/path/to/servicesFolder/syncthing/run
```

```sh
#!/bin/sh
exec syncthing --no-browser --logfile=default 2>&1
```

```bash
chmod +x ~/path/to/servicesFolder/syncthing/run
```

---

## إنشاء service جديد

كل service عبارة عن فولدر فيه ملف `run`:

```
~/path/to/servicesFolder/serviceName/
└── اسم-السيرفس/
    ├── run          ← ملف التشغيل (إلزامي)
    ├── finish       ← بيتشغّل لما السيرفس يوقف (اختياري)
    └── log/
        └── run      ← logging (اختياري)
```

### مثال — backup script

```bash
mkdir -p ~/path/to/servicesFolder/backup
nano ~/path/to/servicesFolder/backup/run
```

```sh
#!/bin/sh
exec /path/to/your/script.sh 2>&1
```

```bash
chmod +x ~/path/to/servicesFolder/backup/run
```

---

## التحكم في الـ User Services

```bash
# تأكد إنك حاطت SVDIR في ~/.profile
export SVDIR=~/path/to/servicesFolder

# بعدين الأوامر العادية بتشتغل
sv status syncthing
sv up syncthing
sv down syncthing
sv restart syncthing
```

---

## تعطيل service مؤقتاً

```bash
# إنشاء ملف down جوّا الـ service
touch ~/path/to/servicesFolder/اسم-السيرفس/down
sv down اسم-السيرفس
```

---

## ملاحظات مهمة

لما تعمل `runsvdir` من سواي بـ `exec runsvdir ~/path/to/servicesFolder &` — هي بتموت مع الجلسة. ده سلوك صح ومتوقع.

لو عايز سيرفس يفضل شغّال حتى لو خرجت من سواي — استخدم الطريقة الأولى (system-level).
