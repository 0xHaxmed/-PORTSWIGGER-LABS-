# Lab 02 — Traversal Sequences Blocked with Absolute Path Bypass (تجاوز حظر المسار النسبي بالمسار المطلق)

[![التصنيف](https://img.shields.io/badge/الفئة-Path%20Traversal-purple)]()
[![النوع](https://img.shields.io/badge/النوع-Path%20Traversal%20—%20Blacklist%20Bypass%20(Absolute%20Path)-orange)]()
[![مستوى الخطورة](https://img.shields.io/badge/الخطورة-عالي%20(High)-critical)]()
[![المصدر](https://img.shields.io/badge/المصدر-PortSwigger%20Academy-blue)](https://portswigger.net/web-security/file-path-traversal)

> هذا الشرح جزء من سلسلة [معامل اجتياز المسارات (Path Traversal)](./README.md) — أكاديمية PortSwigger.

---

## 1. بطاقة تعريف اللاب (Overview)

| الحقل | التفاصيل |
|---|---|
| **اسم اللاب** | Lab 02 — Traversal Sequences Blocked with Absolute Path Bypass (تجاوز حظر المسار النسبي بالمسار المطلق) |
| **تصنيف الثغرة** | Path Traversal — Blacklist Bypass (Absolute Path) |
| **معيار الضعف (CWE)** | CWE-22 / CWE-23: Relative Path Traversal |
| **مستوى الخطورة** | عالي (High) |
| **الهدف من اللاب** | تجاوز فلتر يمنع سلاسل ../ باستخدام المسار المطلق مباشرة لقراءة /etc/passwd. |

---

## 2. فكرة اللاب وسيناريو المشكلة (Concept & Vulnerability Mechanism)

وضع المطور في هذا اللاب فلتر أمان بسيط يفحص القيمة المدخلة: إذا كانت تحتوي على `../` يقوم برفض الطلب. ظن المطور أن هذا الفلتر يمنع الخروج من المجلد نهائياً.

**الخلل الفكري هنا:** في العديد من لغات البرمجة ودوال دمج المسارات (مثل دالة `os.path.join` في بايثون أو بعض دوال C): إذا كان المدخل الثاني يبدأ بسلاش `/` (أي أنه مسار مطلق - Absolute Path)، فإن الدالة تتجاهل المسار الأول الأساسي تماماً وتعتمد المسار الثاني كمسار كامل للملف! وبالتالي، مجرد كتابة `/etc/passwd` تتخطى الفلتر وتقرأ الملف فوراً.

---

## 3. التحليل الجذري للسبب (Root Cause Analysis)

### 🔴 وجهة نظر المهاجم (Attacker Perspective):
المهاجم يدرك أن الفلتر يمنع فقط `../`، فيستغني عنها ويكتب المسار المطلق المباشر `/etc/passwd` مستغلاً سلوك دالة دمج المسارات.

### 🟢 وجهة نظر المدافع والمطور (Defender Perspective):
المطور اعتمد على قائمة سوداء لمنع نمط نسبي محدد (`../`) متجاهلاً أن المسارات المطلقة تؤدي لنفس النتيجة.

---

## 4. خطوات الاستغلال والحل بالتفصيل (Proof of Concept - PoC)

1. اعترض طلب الصورة في Burp Suite: `GET /image?filename=53.jpg`.
2. أرسل الطلب إلى **Repeater**.
3. جرب إرسال `filename=../../../etc/passwd`، ستجد أن السيرفر يرفض أو يرجع خطأ لأن الفلتر منع `../`.
4. الآن، استبدل القيمة بالمسار المطلق المباشر:
`filename=/etc/passwd`.
5. اضغط Send، ستجد أن السيرفر فتح الملف ورد بـ `200 OK` وعرض محتويات ملف الباسورد كاملاً.

---

## 5. أدلة الترافيك (Evidence: Request / Response)

### نموذج طلب الاستغلال (Exploit Request):
```http
GET /image?filename=/etc/passwd HTTP/1.1
Host: <lab-id>.web-security-academy.net
```

### نموذج استجابة السيرفر (Server Response):
```http
HTTP/1.1 200 OK
Content-Type: image/jpeg

root:x:0:0:root:/root:/bin/bash
...
```

---

## 6. الأثر الواقعي للثغرة (Impact)

تجاوز فلاتر المسارات النسبية وقراءة أي ملف على القرص الصلب باستخدام المسار المطلق.

---

## 7. الحماية والإغلاق البرمجي (Remediation)

رفض أي مدخل يبدأ برمز السلاش `/` أو يحتوي على مسار نسبي، والاعتماد على التحقق من المسار القياسي (Canonical Path).

---

## 8. المراجع الرسمية (References)

- [PortSwigger Web Security Academy — Path Traversal](https://portswigger.net/web-security/file-path-traversal)
- [OWASP Top 10 — A01:2021 Broken Access Control](https://owasp.org/Top10/A01_2021-Broken_Access_Control/)
- [CWE-22: Improper Limitation of a Pathname to a Restricted Directory](https://cwe.mitre.org/data/definitions/22.html)

---
*تم إعداد هذا التوثيق للأغراض التعليمية واختبار الاختراق الأخلاقي المصرح به.*
