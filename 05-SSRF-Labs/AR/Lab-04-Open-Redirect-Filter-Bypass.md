# Lab 04 — SSRF with Filter Bypass via Open Redirection (تجاوز القائمة البيضاء عبر ثغرة Open Redirect)

[![التصنيف](https://img.shields.io/badge/الفئة-SSRF-red)]()
[![النوع](https://img.shields.io/badge/النوع-SSRF%20Whitelist%20Bypass%20via%20Open%20Redirection-orange)]()
[![مستوى الخطورة](https://img.shields.io/badge/الخطورة-عالي%20(High)-critical)]()
[![المصدر](https://img.shields.io/badge/المصدر-PortSwigger%20Academy-blue)](https://portswigger.net/web-security/ssrf)

> هذا الشرح جزء من سلسلة [معامل تزوير الطلبات (SSRF)](./README.md) — أكاديمية PortSwigger.

---

## 1. بطاقة تعريف اللاب (Overview)

| الحقل | التفاصيل |
|---|---|
| **اسم اللاب** | Lab 04 — SSRF with Filter Bypass via Open Redirection (تجاوز القائمة البيضاء عبر ثغرة Open Redirect) |
| **تصنيف الثغرة** | SSRF Whitelist Bypass via Open Redirection |
| **معيار الضعف (CWE)** | CWE-918 (chained with CWE-601) |
| **مستوى الخطورة** | عالي (High) |
| **الهدف من اللاب** | الوصول لخادم الشبكة الداخلية 192.168.0.12:8080/admin باستغلال ثغرة Open Redirect في الموقع. |

---

## 2. فكرة اللاب وسيناريو المشكلة (Concept & Vulnerability Mechanism)

قام المطور بحماية باراميتر فحص المخزون بقائمة بيضاء صارمة جداً: يشترط الفلتر أن يبدأ الرابط حصراً بنفس موقع المتجر (Same-Origin): `/product/stock/check?...` ولا يسمح بأي رابط خارجي أو IP.

**فكرة الالتفاف الماكرة (Chaining with Open Redirect):**
يحتوي المتجر على ميزة في صفحة المنتج اسمها "Next Product"، تقوم بتحويل الزائر للمنتج التالي عبر الرابط:
`/product/nextProduct?path=/product?productId=2`.
هذا الرابط مصاب بثغرة **Open Redirect**؛ إذ يمكنك تغيير `path` إلى أي رابط خارجي وسيقوم السيرفر بإصدار كود تحويل `302 Redirect` إليه!
بما أن مكتبة الـ HTTP التي تفحص المخزون تتبع التحويلات تلقائياً:
نضع في باراميتر المخزون رابط الـ Open Redirect الداخلي:
`stockApi=/product/nextProduct?path=http://192.168.0.12:8080/admin/delete?username=carlos`.
1. يرى الفلتر أن الرابط يبدأ بالمسار المحلي المسموح، فيقبله!
2. عندما يطلب السيرفر هذا الرابط، يرد عليه مسار `nextProduct` بأمر تحويل `302` نحو السيرفر الداخلي.
3. يتبع خادم الويب التحويل فوراً ويتصل بالسيرفر الداخلي ويحذف المستخدم!

---

## 3. التحليل الجذري للسبب (Root Cause Analysis)

### 🔴 وجهة نظر المهاجم (Attacker Perspective):
المهاجم يدمج ثغرة Open Redirect محلية للقفز من فوق فلتر القائمة البيضاء والوصول للشبكة الخاصة.

### 🟢 وجهة نظر المدافع والمطور (Defender Perspective):
السماح بإعادة التوجيه المفتوح للمستخدمين وتفعيل ميزة تتبع التحويلات التلقائي في مكتبة الاتصال.

---

## 4. خطوات الاستغلال والحل بالتفصيل (Proof of Concept - PoC)

1. تصفح أي منتج واضغط على زر **Next product** ولاحظ الرابط في المتصفح.
2. ستجد أنه يستخدم مساراً مثل:
`GET /product/nextProduct?path=/product?productId=2`.
3. التقط طلب فحص المخزون `POST /product/stock` في Repeater.
4. استبدل قيمة `stockApi` بالمسار المحلي لتحويل المنتجات موجهاً نحو خادم الباك إند المطلوب في اللاب:
`stockApi=/product/nextProduct?path=http://192.168.0.12:8080/admin/delete?username=carlos`.
5. اضغط Send: سيقبل الفلتر الطلب، وسيتبع السيرفر التحويل ويحذف carlos بنجاح!

---

## 5. أدلة الترافيك (Evidence: Request / Response)

### نموذج طلب الاستغلال (Exploit Request):
```http
POST /product/stock HTTP/1.1
Host: <lab-id>.web-security-academy.net
Content-Type: application/x-www-form-urlencoded

stockApi=/product/nextProduct?path=http://192.168.0.12:8080/admin/delete?username=carlos
```

### نموذج استجابة السيرفر (Server Response):
```http
HTTP/1.1 302 Found
Location: /admin
...
```

---

## 6. الأثر الواقعي للثغرة (Impact)

تجاوز أصعب القوائم البيضاء والوصول لكافة الأنظمة السحابية والداخلية.

---

## 7. الحماية والإغلاق البرمجي (Remediation)

1. إغلاق ثغرة الـ Open Redirect بمنع التحويل لعناوين خارجية.
2. تعطيل ميزة تتبع التحويلات التلقائية في مكتبة الـ HTTP الخادمة.

---

## 8. المراجع الرسمية (References)

- [PortSwigger Web Security Academy — SSRF](https://portswigger.net/web-security/ssrf)
- [OWASP Top 10 — A10:2021 Server-Side Request Forgery](https://owasp.org/Top10/A10_2021-Server-Side_Request_Forgery_%28SSRF%29/)
- [CWE-918: Server-Side Request Forgery](https://cwe.mitre.org/data/definitions/918.html)

---
*تم إعداد هذا التوثيق للأغراض التعليمية واختبار الاختراق الأخلاقي المصرح به.*
