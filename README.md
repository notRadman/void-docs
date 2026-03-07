# Void Linux + Sway — دليل شامل

> دليل شخصي لإعادة بناء البيئة من الصفر على Void Linux + Sway.
> مبني على تجربة فعلية + الدوكس الرسمية.
> كل ملف مستقل — افتح اللي تحتاجه بس.

---

## الملفات

| الملف | المحتوى |
|---|---|
| `01-installation.md` | تثبيت Sway + GPU + Power Management |
| `02-audio.md` | PipeWire + ALSA كامل |
| `03-theming.md` | Theme موحد (GTK + Icons + Cursor + Fonts) |
| `04-screenshot-recording.md` | Screenshot + Screen Recording + Clipboard |
| `05-hardware.md` | Bluetooth + Printer + USB + MTP |
| `06-network.md` | VPN + Hotspot + Tethering |
| `07-apps.md` | Flatpak + AppImage + Emacs + Terminal Tools |
| `08-appimage-desktop.md` | Desktop Entries للـ AppImages بالتفصيل |
| `09-kvm.md` | KVM/QEMU Virtualization |
| `10-troubleshooting.md` | أشهر المشاكل وحلولها |

---

## الترتيب المنطقي لو بتبني من الصفر

```
01 → 02 → 03 → 04 → 05 → 06 → 07
```

---

## قواعد Void الأساسية

```bash
# تثبيت برنامج
sudo xbps-install PACKAGE

# تفعيل service
sudo ln -s /etc/sv/SERVICENAME /var/service/

# بعد أي usermod -aG → لازم logout وادخل تاني

# تشغيل سواي الصح
dbus-run-session sway

# elogind vs seatd → اختار واحد بس، مش ممكن الاتنين
```
