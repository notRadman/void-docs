# تثبيت XeLaTeX على Void Linux مع دعم العربي
## الخطوات الصحيحة المُجرَّبة

---

## 1. شيل الـ packages الغلط

```bash
sudo xbps-remove texlive-bin texlive2025-bin
```

> الـ packages دي بتعمل PATH بس من غير ما تثبّت TeX Live فعلاً.

---

## 2. ثبّت الـ packages الصح عن طريق xbps

```bash
sudo xbps-install texlive-XeTeX texlive-core texlive-lang texlive-dvi
```

| Package | الغرض |
|---------|--------|
| `texlive-XeTeX` | xelatex نفسه |
| `texlive-core` | الأساسيات |
| `texlive-lang` | دعم العربي وباقي اللغات |
| `texlive-dvi` | يشمل xdvipdfmx لتحويل الملف لـ PDF |

> **ملاحظة مهمة:** `texlive-dvi` ضروري جداً لأنه بيجيب `xdvipdfmx` اللي بيحوّل الملف لـ PDF — من غيره xelatex بيتعبق.

---

## 3. تحقق إن xelatex شغال

```bash
which xelatex
# المفروض يرجع: /usr/bin/xelatex
```

---

## 4. ملف `.tex` للعربي

```latex
\documentclass[12pt]{article}
\usepackage{fontspec}
\usepackage{polyglossia}

\setmainlanguage{arabic}
\setotherlanguage{english}
\setmainfont{Amiri}

\begin{document}

\begin{Arabic}
مرحباً بالعالم! هذا نص عربي مع XeLaTeX على Void Linux.
\end{Arabic}

\end{document}
```

---

## 5. تصيير الملف

```bash
xelatex test.tex
```

---

## ملاحظات مهمة

- **لا تستخدم `tlmgr`** مع الطريقة دي — الـ packages جاية من xbps وماتخلطش.
- لو محتاج package إضافي: `xbps-query -Rs texlive` وبتثبّته بـ `xbps-install`.
- `texlive-most` موجود في xbps لو محتاجت packages كتير مرة واحدة.
- لا تنسى تحمل خط عربي الدليل هذا لا يحوي تثبيتا لخطوط عربية، يجب على خط Amiri أن يكون في جهازك حتى تتمكن من تصيير الملف المرفق في المثال.
