# 11 — xbps-src: بناء البرامج اللي مش في الريبو

---

## الفكرة الأساسية بشكل بسيط

`xbps-src` = زي `PKGBUILD` على Arch أو `ebuild` على Gentoo.

بدل ما تبني من سورس يدوياً وتتعذب، بتكتب (أو بتنسخ) "وصفة" صغيرة اسمها **template**، وخبps-src يبني البرنامج ويعمله `.xbps` package تقدر تثبته وتشيله بـ xbps عادي.

**الفايدة الكبيرة:** البرنامج يبقى managed — تقدر تشيله بـ `xbps-remove` زي أي package تاني.

---

## الخطوة 1 — الإعداد الأولي (مرة واحدة بس)

### ثبّت الأدوات اللازمة

```bash
sudo xbps-install git xtools base-devel
```

> `xtools` فيها أدوات مساعدة مهمة جداً لـ xbps-src.

### نزّل void-packages

```bash
cd ~
git clone --depth=1 https://github.com/void-linux/void-packages.git
cd void-packages
```

> `--depth=1` بتجيب آخر snapshot بس من غير كل الـ history — أسرع بكتير.

### Bootstrap الـ masterdir (مرة واحدة بس)

```bash
./xbps-src binary-bootstrap
```

ده بيعمل بيئة بناء معزولة (container بدون root). هياخد وقت أول مرة.

---

## الخطوة 2 — بناء Package موجود في void-packages

معظم البرامج الشائعة موجودة كـ template حتى لو مش في الريبو الرسمي (مثلاً: nonfree packages زي Chrome أو Spotify).

### ابحث عن الـ template

```bash
# الطريقة الأولى
ls srcpkgs/ | grep -i اسم_البرنامج

# الطريقة التانية (أسرع)
./xbps-src search اسم_البرنامج
```

### بناء Package

```bash
./xbps-src pkg اسم_البرنامج
```

### التثبيت بعد البناء

```bash
# أضف الـ local repo للـ xbps
sudo xbps-install --repository=hostdir/binpkgs اسم_البرنامج

# أو لو الـ package في nonfree
sudo xbps-install --repository=hostdir/binpkgs/nonfree اسم_البرنامج
```

### مثال عملي: تثبيت Google Chrome

```bash
cd ~/void-packages

# اتفق على restricted packages
echo "XBPS_ALLOW_RESTRICTED=yes" >> etc/conf

./xbps-src pkg google-chrome
sudo xbps-install --repository=hostdir/binpkgs/nonfree google-chrome
```

---

## الخطوة 3 — إضافة الـ local repo بشكل دائم

بدل ما تكتب `--repository` كل مرة:

```bash
sudo nano /etc/xbps.d/local-repo.conf
```

```
repository=/home/اسمك/void-packages/hostdir/binpkgs
repository=/home/اسمك/void-packages/hostdir/binpkgs/nonfree
```

بعدين تثبت عادي:
```bash
sudo xbps-install اسم_البرنامج
```

---

## الخطوة 4 — كتابة Template من الصفر

لو البرنامج مش موجود خالص في void-packages، هتعمل template بنفسك.

### إنشاء Template جديد

```bash
cd ~/void-packages
./xbps-src newpkg اسم_البرنامج
```

هيفتح editor جوّاه `srcpkgs/اسم_البرنامج/template`.

---

## تشريح الـ Template

```bash
# مثال بسيط: برنامج Python
pkgname=my-tool
version=1.2.3
revision=1
short_desc="أداة رائعة"
maintainer="اسمك <email@example.com>"
license="MIT"
homepage="https://github.com/someone/my-tool"

# مصدر السورس كود
distfiles="https://github.com/someone/my-tool/archive/v${version}.tar.gz"
checksum="sha256 hash هنا"

# المتطلبات وقت البناء
hostmakedepends="python3"

# المتطلبات وقت التشغيل
depends="python3"

# أوامر البناء (بتشتغل جوّه الـ container)
do_install() {
    python3 setup.py install --prefix=/usr --root="${DESTDIR}"
}
```

### جدول أهم المتغيرات

| المتغير | المعنى |
|---|---|
| `pkgname` | اسم الـ package |
| `version` | الإصدار |
| `revision` | رقم revision (ابدأ بـ 1) |
| `short_desc` | وصف قصير (مش أكثر من 72 حرف) |
| `distfiles` | رابط السورس (tar.gz أو zip) |
| `checksum` | الـ sha256 hash للملف |
| `hostmakedepends` | أدوات البناء (compiler، cmake، إلخ) |
| `makedepends` | مكتبات وقت البناء |
| `depends` | مكتبات وقت التشغيل |
| `build_style` | طريقة البناء التلقائية |

### الـ build_style (يريحك من كتابة do_build يدوياً)

```bash
# CMake projects
build_style=cmake

# Meson projects
build_style=meson

# Autotools (./configure)
build_style=gnu-configure

# Python (setup.py)
build_style=python3-module

# Go
build_style=go

# Cargo (Rust)
build_style=cargo

# برامج بسيطة (make فقط)
build_style=gnu-makefile
```

---

## كيفية الحصول على الـ checksum

```bash
# الطريقة الأولى: xtools
xgensum -i srcpkgs/اسم_البرنامج/template

# الطريقة التانية: يدوي
wget https://رابط_السورس
sha256sum اسم_الملف.tar.gz
```

---

## مثال كامل: برنامج Rust بسيط (binary)

لو عندك برنامج Rust موجود على GitHub بس مش في xbps:

```bash
# 1. إنشاء template
./xbps-src newpkg zoxide
```

```bash
# srcpkgs/zoxide/template
pkgname=zoxide
version=0.9.4
revision=1
build_style=cargo
short_desc="Smarter cd command"
maintainer="أنا <me@example.com>"
license="MIT"
homepage="https://github.com/ajeetdsouza/zoxide"
distfiles="https://github.com/ajeetdsouza/zoxide/archive/v${version}.tar.gz"
checksum="ضع sha256 هنا"
```

```bash
# 2. احسب الـ checksum
xgensum -i srcpkgs/zoxide/template

# 3. ابني
./xbps-src pkg zoxide

# 4. ثبّت
sudo xbps-install --repository=hostdir/binpkgs zoxide
```

---

## مثال: برنامج Python (AppImage بديل)

```bash
# srcpkgs/mytool/template
pkgname=mytool
version=2.1.0
revision=1
build_style=python3-module
short_desc="My Python tool"
maintainer="أنا <me@example.com>"
license="GPL-3.0-only"
homepage="https://github.com/someone/mytool"
distfiles="https://github.com/someone/mytool/archive/v${version}.tar.gz"
checksum="sha256 هنا"
depends="python3 python3-requests"
```

---

## أوامر xbps-src اليومية

```bash
# بناء package
./xbps-src pkg اسم_البرنامج

# بناء مع إعادة كل حاجة من الأول
./xbps-src -f pkg اسم_البرنامج

# تنظيف builddir بعد البناء
./xbps-src clean اسم_البرنامج

# شوف dependencies الـ template
./xbps-src show اسم_البرنامج

# تحديث void-packages كله
cd ~/void-packages && git pull

# تحديث packages مبنية قديمة
./xbps-src update-sys
```

---

## تحديث package بنيته أنت

```bash
# 1. حدّث الـ git repo
cd ~/void-packages
git pull

# 2. غيّر version في الـ template
nano srcpkgs/اسم_البرنامج/template
# غيّر version= للإصدار الجديد

# 3. حدّث الـ checksum
xgensum -i srcpkgs/اسم_البرنامج/template

# 4. ابني تاني
./xbps-src pkg اسم_البرنامج

# 5. ثبّت التحديث
sudo xbps-install -u اسم_البرنامج
```

---

## إزالة package بنيته

```bash
# نفس أي package عادي
sudo xbps-remove اسم_البرنامج
```

---

## منع الـ official repo من override الـ local package

لو البرنامج موجود في الريبو الرسمي بنسخة مختلفة وعايز تفضل على نسختك:

```bash
sudo xbps-pkgdb -m hold اسم_البرنامج
```

عشان تلغي الـ hold:
```bash
sudo xbps-pkgdb -m unhold اسم_البرنامج
```

---

## Troubleshooting شائع

**"checksum mismatch":**
```bash
xgensum -i srcpkgs/اسم_البرنامج/template
# هيحدّث الـ checksum تلقائياً
```

**"dependency not found":**
```bash
# دور على الـ dependency في void-packages
ls srcpkgs/ | grep -i اسم_الـdependency

# لو مش موجود، هتحتاج تعمل template ليه كمان
```

**"build failed" مش واضح ليه:**
```bash
# شوف الـ log كامل
./xbps-src pkg اسم_البرنامج 2>&1 | tee /tmp/build.log
less /tmp/build.log
```

**بناء بطيء جداً:**
```bash
# في ~/void-packages/etc/conf
echo "XBPS_MAKEJOBS=$(nproc)" >> etc/conf
# هيستخدم كل الـ cores
```

---

## ملخص سريع

```bash
# إعداد أولي (مرة واحدة)
git clone --depth=1 https://github.com/void-linux/void-packages.git
cd void-packages
./xbps-src binary-bootstrap

# تثبيت package موجود
./xbps-src pkg اسم_البرنامج
sudo xbps-install --repository=hostdir/binpkgs اسم_البرنامج

# package جديد
./xbps-src newpkg اسم_البرنامج
# عدّل template
xgensum -i srcpkgs/اسم_البرنامج/template
./xbps-src pkg اسم_البرنامج
sudo xbps-install --repository=hostdir/binpkgs اسم_البرنامج

# إزالة
sudo xbps-remove اسم_البرنامج
```
