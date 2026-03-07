# 07 — البرامج

---

## متصفح

```bash
sudo xbps-install firefox firefox-i18n-ar
```

Firefox على Wayland بيشتغل تلقائياً لو `MOZ_ENABLE_WAYLAND=1` موجود في البيئة.

---

## Multimedia

```bash
# Video
sudo xbps-install mpv        # native Wayland
# أو
sudo xbps-install vlc

# Images
sudo xbps-install imv         # native Wayland

# Audio control
sudo xbps-install pavucontrol
```

---

## File Manager

```bash
# Terminal
sudo xbps-install nnn

# GUI (لو محتاج)
sudo xbps-install thunar
```

---

## Archive

```bash
sudo xbps-install unzip zip p7zip file-roller
```

---

## System Monitoring

```bash
sudo xbps-install htop btop fastfetch lm_sensors
```

---

## Trash CLI

```bash
sudo xbps-install trash-cli
```

---

## Flatpak

```bash
sudo xbps-install flatpak xdg-user-dirs
flatpak remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo
```

```bash
# تثبيت
flatpak install flathub com.discordapp.Discord

# تشغيل
flatpak run com.discordapp.Discord

# تحديث الكل
flatpak update

# قائمة المثبت
flatpak list
```

---

## AppImage

```bash
sudo xbps-install fuse
sudo modprobe fuse

chmod +x اسم_البرنامج.AppImage
./اسم_البرنامج.AppImage

# لو بيشتكي من FUSE
./اسم_البرنامج.AppImage --appimage-extract-and-run

# مكان منظم
mkdir -p ~/AppImages
```

> للتفاصيل الكاملة عن Desktop Entries للـ AppImages: راجع `08-appimage-desktop.md`

---

## Emacs

```bash
sudo xbps-install emacs         # مع GUI
sudo xbps-install emacs-nox     # terminal فقط (أخف)
```

الكونفيج في: `~/.config/emacs/init.el`

---

## Terminal Tools للـ Coding

```bash
sudo xbps-install \
  ripgrep \
  fd \
  bat \
  eza \
  fzf \
  zoxide \
  btop \
  lazygit
```

| الأداة | بديل عن | الميزة |
|---|---|---|
| `rg` | grep | أسرع بكتير |
| `fd` | find | أسهل وأسرع |
| `bat` | cat | syntax highlighting |
| `eza` | ls | أجمل وأوضح |
| `fzf` | — | fuzzy finder لكل حاجة |
| `z` (zoxide) | cd | بيتذكر الأماكن |
| `btop` | htop | أجمل وأكتر معلومات |
| `lazygit` | git CLI | TUI لـ git |

### ربط fzf مع الـ history

أضف في `~/.bashrc`:
```bash
source /usr/share/fzf/key-bindings.bash
source /usr/share/fzf/completion.bash
```

بعدين `Ctrl+R` هيفتح fuzzy search في الـ history.
