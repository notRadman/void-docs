# 09 — KVM/QEMU Virtualization

---

## الخطوة 1 — تأكد إن الـ CPU يدعم Virtualization

```bash
lscpu | grep -i virtualization
grep -E '(vmx|svm)' /proc/cpuinfo
```

- **Intel:** لازم يظهر `VT-x` أو `vmx`
- **AMD:** لازم يظهر `AMD-V` أو `svm`

لو مش ظاهر، فعّله من الـ BIOS: ابحث عن *Virtualization Technology* أو *Intel VT-x*.

---

## الخطوة 2 — التثبيت

```bash
sudo xbps-install virt-manager libvirt qemu dnsmasq bridge-utils openbsd-netcat
```

| الحزمة | الوظيفة |
|---|---|
| `virt-manager` | GUI لإدارة الـ VMs |
| `libvirt` | الـ virtualization API |
| `qemu` | المحاكي |
| `dnsmasq` | DNS/DHCP للشبكات الافتراضية |
| `bridge-utils` | network bridging |

---

## الخطوة 3 — تفعيل الـ Services

```bash
sudo ln -s /etc/sv/dbus /var/service/        # لو مش متفعّل
sudo ln -s /etc/sv/libvirtd /var/service/
sudo ln -s /etc/sv/virtlockd /var/service/
sudo ln -s /etc/sv/virtlogd /var/service/
```

---

## الخطوة 4 — إضافة المستخدم لـ Group

```bash
sudo usermod -aG libvirt $USER
# logout وادخل تاني
```

---

## الخطوة 5 — تحميل KVM Module

```bash
# Intel
sudo modprobe kvm_intel

# AMD
sudo modprobe kvm_amd
```

### تحميل تلقائي عند الـ Boot

```bash
# Intel
echo "kvm_intel" | sudo tee /etc/modules-load.d/kvm.conf

# AMD
echo "kvm_amd" | sudo tee /etc/modules-load.d/kvm.conf
```

---

## الخطوة 6 — إعداد libvirt (اختياري)

لتشغيل الـ VMs بدون sudo:

```bash
sudo nano /etc/libvirt/libvirtd.conf
```

شيل الـ `#` من:
```
unix_sock_group = "libvirt"
unix_sock_ro_perms = "0777"
unix_sock_rw_perms = "0770"
```

```bash
sudo nano /etc/libvirt/libvirt.conf
```

شيل الـ `#` من:
```
uri_default = "qemu:///system"
```

```bash
sudo sv restart libvirtd
```

---

## الخطوة 7 — Reboot

```bash
sudo reboot
```

---

## التحقق بعد الـ Reboot

```bash
# KVM modules
lsmod | grep kvm

# /dev/kvm
ls -la /dev/kvm

# libvirt
sudo virsh list --all

# groups
groups | grep libvirt
```

---

## تشغيل virt-manager

```bash
virt-manager
```

---

## أوامر virsh المهمة

```bash
virsh list --all              # كل الـ VMs
virsh start <vm-name>         # تشغيل
virsh shutdown <vm-name>      # إيقاف ناعم
virsh destroy <vm-name>       # إيقاف قسري
virsh undefine <vm-name>      # حذف
virsh dominfo <vm-name>       # معلومات
virsh edit <vm-name>          # تعديل الكونفيج
```

---

## Nested Virtualization (لو شغّال Void جوّه VM)

```bash
# Intel
sudo modprobe -r kvm_intel
sudo modprobe kvm_intel nested=1

# دائم
echo "options kvm_intel nested=1" | sudo tee /etc/modprobe.d/kvm.conf
```

---

## Troubleshooting

```bash
# "Unable to connect to libvirt"
sudo sv status dbus
sudo sv status libvirtd
sudo sv up libvirtd

# "Could not access KVM kernel module"
lsmod | grep kvm
sudo modprobe kvm_intel
ls -la /dev/kvm

# "Permission denied"
groups | grep libvirt
sudo usermod -aG libvirt $USER
# logout وادخل تاني
```
