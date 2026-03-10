# 06 — الشبكة (VPN + Hotspot + Tethering)

---

## إعداد NetworkManager (لو مش متعمل)

```bash
sudo xbps-install NetworkManager network-manager-applet vnstat

# إيقاف الخدمات القديمة
sudo sv stop dhcpcd wpa_supplicant 2>/dev/null
sudo rm /var/service/dhcpcd /var/service/wpa_supplicant 2>/dev/null

# تفعيل NetworkManager
sudo ln -s /etc/sv/dbus /var/service/
sudo ln -s /etc/sv/NetworkManager /var/service/

# تفعيل مراقب الاستهلاك vnstat
sudo ln -s /etc/sv/vnstatd /var/service/
```

### DNS ثابت (اختياري)

```bash
sudo nano /etc/resolv.conf
```
```
nameserver 1.1.1.1
nameserver 8.8.8.8
```
```bash
sudo chattr +i /etc/resolv.conf   # يمنع NetworkManager من تغييره
```

---

## VPN

### تثبيت الـ plugins

```bash
# OpenVPN
sudo xbps-install NetworkManager-openvpn

# WireGuard
sudo xbps-install wireguard-tools NetworkManager-wireguard

# L2TP/IPSec
sudo xbps-install NetworkManager-l2tp
```

### إضافة VPN

**Import ملف config (الأسهل):**
```bash
nmcli connection import type openvpn file /path/to/config.ovpn
nmcli connection import type wireguard file /path/to/config.conf
```

**بـ TUI:**
```bash
nmtui
# Edit a connection → Add → اختار نوع الـ VPN
```

### تشغيل/إيقاف

```bash
nmcli connection up اسم_الـVPN
nmcli connection down اسم_الـVPN
nmcli connection show
```

---

## Hotspot

> ⚠️ معظم كروت WiFi مش بتدعم إنترنت + hotspot على نفس الكارت.

### تشغيل سريع

```bash
nmcli device wifi hotspot ifname wlan0 ssid "اسم_الشبكة" password "كلمة_السر"
```

### عرض الـ QR Code

```bash
nmcli device wifi show-password
```

### إيقاف

```bash
nmcli connection down Hotspot
```

### Hotspot دائم

```bash
nmcli con add type wifi ifname wlan0 con-name "MyHotspot" autoconnect no ssid "اسم_شبكتك" \
  802-11-wireless.mode ap \
  802-11-wireless.band bg \
  ipv4.method shared \
  wifi-sec.key-mgmt wpa-psk \
  wifi-sec.psk "كلمة_السر"

nmcli con up MyHotspot
nmcli con down MyHotspot
```

---

## USB Tethering (إنترنت الموبايل على اللاب)

```bash
# وصّل الموبايل بـ USB وفعّل USB Tethering منه
ip link show             # هيظهر interface جديد (usb0 أو enp...)
nmcli device status      # NetworkManager المفروض يشوفه تلقائي
```
