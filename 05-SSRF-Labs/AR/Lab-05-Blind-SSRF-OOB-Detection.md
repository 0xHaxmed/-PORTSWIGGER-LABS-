# Lab 05 — Blind SSRF with Out-of-Band Detection (كشف SSRF أعمى خارجي عبر هيدر Referer)

[![التصنيف](https://img.shields.io/badge/الفئة-SSRF-red)]()
[![النوع](https://img.shields.io/badge/النوع-Server-Side%20Request%20Forgery%20(Blind%20/%20OAST)-orange)]()
[![مستوى الخطورة](https://img.shields.io/badge/الخطورة-عالي%20(High)-critical)]()
[![المصدر](https://img.shields.io/badge/المصدر-PortSwigger%20Academy-blue)](https://portswigger.net/web-security/ssrf)

> هذا الشرح جزء من سلسلة [معامل تزوير الطلبات (SSRF)](./README.md) — أكاديمية PortSwigger.

---

## 1. بطاقة تعريف اللاب (Overview)

| الحقل | التفاصيل |
|---|---|
| **اسم اللاب** | Lab 05 — Blind SSRF with Out-of-Band Detection (كشف SSRF أعمى خارجي عبر هيدر Referer) |
| **تصنيف الثغرة** | Server-Side Request Forgery (Blind / OAST) |
| **معيار الضعف (CWE)** | CWE-918: Server-Side Request Forgery (SSRF) |
| **مستوى الخطورة** | عالي (High) |
| **الهدف من اللاب** | إجبار نظام التحليلات الداخلي في السيرفر على إرسال طلب HTTP إلى خادم Burp Collaborator عبر هيدر Referer. |

---

## 2. فكرة اللاب وسيناريو المشكلة (Concept & Vulnerability Mechanism)

يقوم الموقع بتشغيل نظام إحصائيات وتحليلات داخلي (Analytics Engine) يراقب الزوار. عندما يتصفح المستخدم أي منتج، يقوم نظام التحليلات بقراءة ترويسة `Referer` (التي توضح الموقع الذي جاء منه الزائر)، ويقوم السيرفر في الخلفية بإجراء طلب HTTP خادمي إلى الرابط الموجود في `Referer` لتحليله.

**الخلل الفكري هنا:** السيرفر يثق في محتوى `Referer` دون فحص! ولأن هذه العملية تتم في الخلفية بشكل أعمى تماماً (Blind)، لا يظهر أي أثر لها في صفحة المتجر. نستخدم خادم **Burp Collaborator** كفخ: نضع رابطه في هيدر `Referer`، فيقوم السيرفر بالاتصال به مسجلاً زيارة خارجية تثبت وجود الثغرة.

---

## 3. التحليل الجذري للسبب (Root Cause Analysis)

### 🔴 وجهة نظر المهاجم (Attacker Perspective):
المهاجم يضع نطاق Collaborator خارجي في هيدر Referer ليجبر برنامج التحليلات الداخلي على الاتصال به.

### 🟢 وجهة نظر المدافع والمطور (Defender Perspective):
قيام السيرفر بإجراء طلبات اتصال نحو روابط مأخوذة مباشرة من ترويسات العميل دون تصفية أو قائمة بيضاء.

---

## 4. خطوات الاستغلال والحل بالتفصيل (Proof of Concept - PoC)

1. في أداة **Burp Suite Professional**، افتح تبويب **Collaborator** واضغط **Copy to clipboard** لنسخ دومين فريد.
2. تصفح أي منتج والتقط طلب الـ `GET /product?productId=1` وأرسله إلى **Repeater**.
3. في ترويسات الطلب، ابحث عن هيدر `Referer`، واستبدل قيمته برابط الـ Collaborator المنسوخ:
`Referer: http://xyz.oastify.com`.
4. اضغط Send (سترجع الصفحة استجابة طبيعية 200 OK كالمعتاد).
5. ارجع لتبويب **Collaborator** واضغط **Poll now**.
6. ستلاحظ ظهور طلبات HTTP و DNS وردت من السيرفر الهدف إلى نطاقك، وبذلك يحل اللاب!

---

## 5. أدلة الترافيك (Evidence: Request / Response)

### نموذج طلب الاستغلال (Exploit Request):
```http
GET /product?productId=1 HTTP/1.1
Host: <lab-id>.web-security-academy.net
Referer: http://xyz.oastify.com
Cookie: session=...
```

### نموذج استجابة السيرفر (Server Response):
```http
HTTP/1.1 200 OK
...
```

---

## 6. الأثر الواقعي للثغرة (Impact)

استغلال الخادم كنقطة انطلاق لشن هجمات DDoS أو التجسس واستهداف خوادم أخرى.

---

## 7. الحماية والإغلاق البرمجي (Remediation)

عدم زيارة الروابط الواردة في ترويسات الـ Referer برمجياً من السيرفر، أو حصرها في نطاقات موثوقة حصراً.

---

## 8. المراجع الرسمية (References)

- [PortSwigger Web Security Academy — SSRF](https://portswigger.net/web-security/ssrf)
- [OWASP Top 10 — A10:2021 Server-Side Request Forgery](https://owasp.org/Top10/A10_2021-Server-Side_Request_Forgery_%28SSRF%29/)
- [CWE-918: Server-Side Request Forgery](https://cwe.mitre.org/data/definitions/918.html)

---
*تم إعداد هذا التوثيق للأغراض التعليمية واختبار الاختراق الأخلاقي المصرح به.*
