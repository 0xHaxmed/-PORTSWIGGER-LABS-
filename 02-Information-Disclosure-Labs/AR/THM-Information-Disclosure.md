# TryHackMe — دليل استكشاف ثغرات تسريب المعلومات وجمع البيانات

[![المنصة](https://img.shields.io/badge/Platform-TryHackMe-red)](https://tryhackme.com/)
[![الفئة](https://img.shields.io/badge/Category-Information%20Disclosure-blue)]()

دليل عملي يشرح استراتيجيات البحث المتقدمة عن البيانات المسربة أثناء جولات اختبار الاختراق والـ Bug Bounty.

---

## 1. التفتيش المنهجي في كود الويب
1. **فحص الـ Source Code والـ JavaScript:** استخدام ميزات البحث في المتصفح عن الكلمات الحساسة:
   `apikey`, `secret`, `password`, `token`, `admin`, `internal`.
2. **فحص ملفات الـ Source Maps (`.js.map`):** تتيح للمهاجم استعادة كود المشروع الأصلي غير المضغوط قبل مرحلة البناء والـ Minification.

---

## 2. تخمين المسارات الحساسة وملفات الباك آب (Fuzzing)
باستخدام أداة `ffuf`:
```bash
ffuf -u http://target.com/FUZZ -w common_files.txt -e .bak,.old,.txt,.zip,.git
```

---

## 3. استخراج المستودعات المسربة (`.git`)
عند العثور على مسار `/.git/`:
1. نستخدم أداة مثل `git-dumper`:
   ```bash
   git-dumper http://target.com/.git/ ./local-repo
   ```
2. نتصفح الـ Commits السابقة للبحث عن كلمات المرور المحذوفة:
   ```bash
   git log -p
   git diff HEAD~1 HEAD
   ```
