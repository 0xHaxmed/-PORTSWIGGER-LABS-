# Lab 01 — Unprotected Admin Functionality (لوحة إدارة غير محمية)

[![التصنيف](https://img.shields.io/badge/الفئة-Broken%20Access%20Control-red)]()
[![النوع](https://img.shields.io/badge/النوع-Vertical%20Broken%20Access%20Control%20(تصعيد%20صلاحيات%20رأسي)-orange)]()
[![مستوى الخطورة](https://img.shields.io/badge/الخطورة-عالي%20(High)-critical)]()
[![المصدر](https://img.shields.io/badge/المصدر-PortSwigger%20Academy-blue)](https://portswigger.net/web-security/access-control)

> هذا الشرح جزء من سلسلة [معامل التحكم في الوصول (BAC)](./README.md) — أكاديمية PortSwigger.

---

## 1. بطاقة تعريف اللاب (Overview)

| الحقل | التفاصيل |
|---|---|
| **اسم اللاب** | Lab 01 — Unprotected Admin Functionality (لوحة إدارة غير محمية) |
| **تصنيف الثغرة** | Vertical Broken Access Control (تصعيد صلاحيات رأسي) |
| **معيار الضعف (CWE)** | CWE-284: Improper Access Control |
| **مستوى الخطورة** | عالي (High) |
| **الهدف من اللاب** | الوصول إلى لوحة الإدارة وحذف المستخدم carlos. |

---

## 2. فكرة اللاب وسيناريو المشكلة (Concept & Vulnerability Mechanism)

في هذا اللاب، قام المطور بإنشاء لوحة تحكم إدارية في المسار `/administrator-panel` لحذف المستخدمين وإدارة النظام. ولكي يمنع محركات البحث مثل Google من أرشفة هذا الرابط، قام بوضعه داخل ملف `robots.txt` تحت سطر `Disallow: /administrator-panel`.

**الخلل الفكري هنا:** ظن المطور أن مجرد إخفاء الرابط عن أعين المستخدمين العاديين، ووضعه في `robots.txt`، يكفي لحماية الصفحة! لم يقم ببرمجة أي فحص خادمي (Server-side Session Check) يتأكد من أن من يطلب هذا الرابط يحمل رتبة `Admin`. النتيجة هي أن أي شخص على الإنترنت يعرف الرابط يمكنه فتحه واستخدامه بصلاحيات كاملة.

---

## 3. التحليل الجذري للسبب (Root Cause Analysis)

### 🔴 وجهة نظر المهاجم (Attacker Perspective):
المهاجم يقرأ ملف `robots.txt` كخريطة استطلاع تكشف له المسارات الحساسة، ثم يطلب الرابط الإداري مباشرة متجاوزاً أي قيود في الواجهة.

### 🟢 وجهة نظر المدافع والمطور (Defender Perspective):
المطور اعتمد على مبدأ الحماية عبر الإخفاء (Security through Obscurity) واستخدم ملف `robots.txt` كأداة أمان، متناسياً أن روبوتات البحث تطيعه اختيارياً بينما المهاجم يستغله كدليل استكشاف.

---

## 4. خطوات الاستغلال والحل بالتفصيل (Proof of Concept - PoC)

1. افتح المتصفح واذهب إلى المسار الرئيسي للاب، ثم أضف في نهاية الرابط: `/robots.txt`.
2. ستلاحظ وجود سطر يحتوي على: `Disallow: /administrator-panel`.
3. انسخ المسار `/administrator-panel` والصقه في شريط العنوان بالمتصفح واضغط Enter.
4. ستفتح لك لوحة تحكم الأدمن مباشرة دون أن يطلب منك تسجيل دخول أو صلاحيات إدارية!
5. ابحث عن المستخدم `carlos` واضغط على زر **Delete** (أو أرسل طلب `GET /administrator-panel/delete?username=carlos`).
6. ستظهر لك رسالة نجاح حل اللاب (Solved).

---

## 5. أدلة الترافيك (Evidence: Request / Response)

### نموذج طلب الاستغلال (Exploit Request):
```http
GET /administrator-panel/delete?username=carlos HTTP/1.1
Host: <lab-id>.web-security-academy.net
```

### نموذج استجابة السيرفر (Server Response):
```http
HTTP/1.1 302 Found
Location: /administrator-panel
...
```

---

## 6. الأثر الواقعي للثغرة (Impact)

سيطرة غير مصرح بها على وظائف إدارية حساسة تؤدي إلى حذف بيانات المستخدمين أو تخريب النظام بالكامل.

---

## 7. الحماية والإغلاق البرمجي (Remediation)

يجب وضع طبقة حماية برمجية مركزية (Middleware / Auth Filter) تفحص جلسة المستخدم وتتأكد من امتلاكه لدور `Admin` قبل تنفيذ أي كود داخل `/administrator-panel`، مع إرجاع `401 Unauthorized` أو `403 Forbidden` في حال عدم وجود الصلاحية.

---

## 8. المراجع الرسمية (References)

- [PortSwigger Web Security Academy — Access Control](https://portswigger.net/web-security/access-control)
- [OWASP Top 10 — A01:2021 Broken Access Control](https://owasp.org/Top10/A01_2021-Broken_Access_Control/)
- [CWE Database Details](https://cwe.mitre.org/)

---
*تم إعداد هذا التوثيق للأغراض التعليمية واختبار الاختراق الأخلاقي المصرح به.*
