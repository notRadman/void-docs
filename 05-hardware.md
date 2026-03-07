# 05 — الأجهزة (Bluetooth + Printer + USB + MTP)

---

## Bluetooth

### التثبيت

```bash
sudo xbps-install bluez blueman libspa-bluetooth
sudo ln -s /etc/sv/bluetoothd /var/service/
sudo usermod -aG bluetooth $USER
# logout وادخل تاني
```

في كونفيج سواي:
```
exec blueman-applet &
```

### الاستخدام (CLI)

```bash
bluetoothctl
power on
scan on
pair XX:XX:XX:XX:XX:XX
connect XX:XX:XX:XX:XX:XX
trust XX:XX:XX:XX:XX:XX
```

### التحقق

```bash
sv status bluetoothd
```

---

## Printer — CUPS

### التثبيت

```bash
sudo xbps-install cups cups-filters gutenprint system-config-printer
# HP:
sudo xbps-install hplip
# Epson:
sudo xbps-install epson-inkjet-printer-escpr

sudo ln -s /etc/sv/cupsd /var/service/
sudo usermod -aG lpadmin $USER
```

### إضافة الطابعة

افتح في المتصفح:
```
http://localhost:631
```
Administration → Add Printer

---

## USB Automount — udiskie

### التثبيت

```bash
sudo xbps-install udisks2 udiskie gvfs ntfs-3g exfat-utils
```

### Polkit Rule (عشان ما يطلبش password)

```bash
sudo mkdir -p /etc/polkit-1/rules.d/
sudo nano /etc/polkit-1/rules.d/50-udisks.rules
```

```javascript
polkit.addRule(function(action, subject) {
  if (subject.isInGroup("wheel")) {
    if (action.id.startsWith("org.freedesktop.udisks2.")) {
      return polkit.Result.YES;
    }
  }
});
```

في كونفيج سواي:
```
exec udiskie --tray &
```

### Unmount يدوي

```bash
udiskie-umount /run/media/$USER/اسم_الـdrive
udiskie-umount --all
```

---

## MTP (موبايل عبر USB)

```bash
sudo xbps-install go-mtpfs libmtp
sudo usermod -aG plugdev $USER
```

### إعداد FUSE (مهم!)

```bash
sudo nano /etc/fuse.conf
# شيل الـ # من السطر ده:
# user_allow_other
```

أو تلقائياً:
```bash
sudo sed -i '/user_allow_other/s/^#//g' /etc/fuse.conf
```

### الاستخدام

```bash
# شوف الأجهزة المتصلة
mtp-detect

# Mount الجهاز
mkdir -p ~/phone
go-mtpfs ~/phone &

# Unmount
fusermount -u ~/phone
```
