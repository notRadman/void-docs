# 12 — GNOME Keyring

---

## إيه اللي بيعمله بالظبط

GNOME Keyring بيحل 3 مشاكل:

| المشكلة | من غير Keyring | مع Keyring |
|---|---|---|
| WiFi passwords | NetworkManager بيطلبها كل boot | بتتحفظ وتتفتح تلقائياً |
| SSH keys | بتدخل الـ passphrase كل مرة | بتدخلها مرة واحدة |
| Secrets (VS Code, apps) | خطأ "no secrets service" | بيشتغل صح |

---

## التثبيت

```bash
sudo xbps-install gnome-keyring libsecret seahorse
```

- `gnome-keyring` — الـ daemon نفسه
- `libsecret` — الـ library اللي البرامج بتتكلم بيها مع الـ keyring
- `seahorse` — GUI لإدارة الـ keyring (مفيد للـ troubleshooting)

---

## الإعداد — 3 خطوات

### الخطوة 1 — PAM (الفتح التلقائي عند الـ login)

```bash
sudo nano /etc/pam.d/login
```

أضف سطرين — واحد في نهاية `auth` وواحد في نهاية `session`:

```
#%PAM-1.0
auth       required    pam_securetty.so
auth       requisite   pam_nologin.so
auth       include     system-local-login
auth       optional    pam_gnome_keyring.so          # ← أضف هنا

account    include     system-local-login

session    include     system-local-login
session    optional    pam_gnome_keyring.so auto_start  # ← أضف هنا
```

> ده اللي بيفتح الـ keyring تلقائياً لما بتدخل الـ password عند الـ login — من غيره هيطلب منك password تاني.

### الخطوة 2 — تشغيل الـ daemon في سكريبت سواي

في `~/.local/bin/start-sway`، **قبل** سطر `exec dbus-run-session sway`، أضف:

```bash
# GNOME Keyring
eval $(gnome-keyring-daemon --start --components=gpg,pkcs11,secrets,ssh \
    | sed 's/^\(.*\)/export \1/g')
export SSH_AUTH_SOCK
```

المحتوى الكامل للسكريبت بعد التعديل:

```bash
#!/bin/sh

# XDG_RUNTIME_DIR
if [ -z "$XDG_RUNTIME_DIR" ]; then
    export XDG_RUNTIME_DIR=/tmp/${UID}-runtime-dir
    mkdir -p "$XDG_RUNTIME_DIR"
    chmod 0700 "$XDG_RUNTIME_DIR"
fi

# متغيرات Wayland
export XDG_SESSION_TYPE=wayland
export XDG_SESSION_DESKTOP=sway
export XDG_CURRENT_DESKTOP=sway
export MOZ_ENABLE_WAYLAND=1
export QT_QPA_PLATFORM=wayland
export ELECTRON_OZONE_PLATFORM_HINT=auto
export _JAVA_AWT_WM_NONREPARENTING=1

# GNOME Keyring — لازم قبل dbus-run-session
eval $(gnome-keyring-daemon --start --components=gpg,pkcs11,secrets,ssh \
    | sed 's/^\(.*\)/export \1/g')
export SSH_AUTH_SOCK

exec dbus-run-session sway
```

### الخطوة 3 — XDG Portal للـ Secrets

عشان البرامج تعرف تتكلم مع الـ keyring عبر الـ portal:

```bash
nano ~/.config/xdg-desktop-portal/sway-portals.conf
```

أضف سطر الـ Secret:

```ini
[preferred]
default=gtk
org.freedesktop.impl.portal.Screenshot=wlr
org.freedesktop.impl.portal.ScreenCast=wlr
org.freedesktop.impl.portal.Secret=gnome-keyring
```

---

## التحقق

```bash
# بعد تشغيل سواي، افتح terminal وجرّب:
echo $SSH_AUTH_SOCK
# المفروض يطلع مسار زي: /run/user/1000/keyring/ssh

# تأكد إن الـ daemon شغّال
ps aux | grep gnome-keyring

# اختبر الـ secrets service
secret-tool store --label="test" service test user test
# لو طلب password وقبل → شغّال صح
secret-tool lookup service test
secret-tool clear service test
```

---

## Seahorse — إدارة الـ Keyring

```bash
seahorse
```

من هنا تقدر:
- تشوف كل الـ passwords والـ SSH keys المحفوظة
- تغيّر الـ master password للـ keyring
- تحذف أي secret مش محتاجه

---

## مشاكل شائعة

**"No such secret service" أو VS Code بيشتكي:**
```bash
# تأكد من SSH_AUTH_SOCK
echo $SSH_AUTH_SOCK

# لو فاضي، الـ keyring مش اشتغل — راجع سكريبت التشغيل
cat ~/.local/bin/start-sway | grep keyring
```

**الـ keyring بيطلب password في كل مرة:**
```bash
# تأكد من PAM
grep gnome_keyring /etc/pam.d/login
# المفروض يظهر السطرين اللي أضفتهم
```

**SSH key مش بيتحفظ:**
```bash
# أضف الـ key للـ keyring
ssh-add ~/.ssh/id_rsa

# تأكد إن SSH_AUTH_SOCK شغّال
echo $SSH_AUTH_SOCK
ssh-add -l
```

**Seahorse مش بتظهر الـ passwords:**
```bash
# الـ daemon ممكن يكون اشتغل غلط — وقّفه وعيد
pkill gnome-keyring-daemon
eval $(gnome-keyring-daemon --start --components=gpg,pkcs11,secrets,ssh \
    | sed 's/^\(.*\)/export \1/g')
export SSH_AUTH_SOCK
seahorse
```
