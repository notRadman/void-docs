# 02 — الصوت (PipeWire + ALSA)

---

## التثبيت

```bash
sudo xbps-install \
  pipewire \
  wireplumber \
  alsa-utils \
  alsa-tools \
  alsa-pipewire \
  libspa-bluetooth \
  pavucontrol \
  helvum
```

> `libspa-bluetooth` ضروري عشان الـ Bluetooth headphones تشتغل مع PipeWire.

---

## إعداد WirePlumber

```bash
mkdir -p ~/.config/pipewire/pipewire.conf.d

ln -s /usr/share/examples/wireplumber/10-wireplumber.conf \
      ~/.config/pipewire/pipewire.conf.d/
```

---

## إعداد PipeWire-Pulse (لدعم برامج PulseAudio)

```bash
sudo xbps-install pipewire-pulse

ln -s /usr/share/examples/pipewire/20-pipewire-pulse.conf \
      ~/.config/pipewire/pipewire.conf.d/
```

---

## إعداد ALSA مع PipeWire

```bash
sudo mkdir -p /etc/alsa/conf.d

sudo ln -s /usr/share/alsa/alsa.conf.d/50-pipewire.conf \
           /etc/alsa/conf.d/

sudo ln -s /usr/share/alsa/alsa.conf.d/99-pipewire-default.conf \
           /etc/alsa/conf.d/

alsactl init
```

---

## التشغيل

على Void، PipeWire **بيشتغل من داخل سواي**، مش كـ system service.

في كونفيج سواي:
```
exec pipewire &
```

> ⚠️ لا تحاول تفعّله بـ `ln -s /etc/sv/pipewire /var/service/` — مش هيشتغل صح.

---

## ضبط مستويات الصوت

```bash
# CLI
alsamixer

# GUI
pavucontrol
```

---

## التحقق

```bash
pactl info | grep "Server Name"
# المفروض: PulseAudio (on PipeWire ...)

# شوف الـ devices
pactl list sources short
pactl list sinks short
```

---

## مشاكل شائعة

**الصوت مش شغّال:**
```bash
# تأكد إن PipeWire شغّال
pactl info

# لو مش شغّال، شغّله يدوياً للتجربة
pipewire &
pactl info
```

**Bluetooth Headphones مش بتشتغل:**
```bash
# تأكد من وجود libspa-bluetooth
sudo xbps-install libspa-bluetooth
# بعدين restart سواي
```

**No Sound Device Found:**
```bash
# تأكد من ALSA
aplay -l
# لو فاضي، راجع GPU drivers والـ kernel modules
```
