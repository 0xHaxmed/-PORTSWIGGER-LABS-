# Lab 02 — Basic SSRF Against Another Back-End System (فحص واستهداف أنظمة الشبكة الداخلية)

[![التصنيف](https://img.shields.io/badge/الفئة-SSRF-red)]()
[![النوع](https://img.shields.io/badge/النوع-Server-Side%20Request%20Forgery%20(Internal%20Subnet%20Scan)-orange)]()
[![مستوى الخطورة](https://img.shields.io/badge/الخطورة-عالي%20(High)-critical)]()
[![المصدر](https://img.shields.io/badge/المصدر-PortSwigger%20Academy-blue)](https://portswigger.net/web-security/ssrf)

> هذا الشرح جزء من سلسلة [معامل تزوير الطلبات (SSRF)](./README.md) — أكاديمية PortSwigger.

---

## 1. بطاقة تعريف اللاب (Overview)

| الحقل | التفاصيل |
|---|---|
| **اسم اللاب** | Lab 02 — Basic SSRF Against Another Back-End System (فحص واستهداف أنظمة الشبكة الداخلية) |
| **تصنيف الثغرة** | Server-Side Request Forgery (Internal Subnet Scan) |
| **معيار الضعف (CWE)** | CWE-918: Server-Side Request Forgery (SSRF) |
| **مستوى الخطورة** | عالي (High) |
| **الهدف من اللاب** | استكشاف الشبكة الداخلية في نطاق 192.168.0.X على المنفذ 8080 للعثور على لوحة الإدارة وحذف carlos. |

---

## 2. فكرة اللاب وسيناريو المشكلة (Concept & Vulnerability Mechanism)

في هذا السيناريو، تم حظر الوصول إلى `localhost`. ولكن خادم الويب يقع داخل شبكة داخلية خاصة (Private Subnet) تضم خوادم أخرى لا يمكن الوصول إليها من الإنترنت إطلاقاً وتبدأ عناوينها بـ `192.168.0.X`. أحد هذه الخوادم يعمل على المنفذ `8080` ويحتوي على لوحة إدارة.

**الخلل الفكري هنا:** خادم الويب يملك صلاحية الاتصال بأي IP داخل الشبكة الخاصة. يستطيع المهاجم استخدام أداة فحص آلية (Burp Intruder) لتجربة جميع الأرقام من `1` إلى `254` في الرابط: `stockApi=http://192.168.0.§X§:8080/admin`. السيرفرات التي لا تعمل ستستغرق وقتاً طويلاً أو تعيد خطأ، بينما السيرفر النشط سيرد فوراً بـ `200 OK`!

---

## 3. التحليل الجذري للسبب (Root Cause Analysis)

### 🔴 وجهة نظر المهاجم (Attacker Perspective):
المهاجم يستخدم خادم الويب كـ Proxy داخلي لعمل مسح شامل (Port/Host Scanning) لأجهزة الشبكة الخاصة واختراق لوحة الأدمن.

### 🟢 وجهة نظر المدافع والمطور (Defender Perspective):
عدم عزل خادم الويب العام عن شبكة الخوادم الإدارية الداخلية وعدم تقييد اتصالاته الصادرة بجدار ناري.

---

## 4. خطوات الاستغلال والحل بالتفصيل (Proof of Concept - PoC)

1. التقط طلب فحص المخزون `POST /product/stock` في Burp وأرسله إلى **Intruder** (`Ctrl+I`).
2. في تبويب Positions، عدل باراميتر `stockApi` وضع علامة Payload على الرقم الأخير من الـ IP:
`stockApi=http://192.168.0.§1§:8080/admin`.
3. في تبويب **Payloads**:
- اختر Payload type: **Numbers**.
- From: 1, To: 255, Step: 1.
4. اضغط **Start attack** وراقب عمود **Status Code**.
5. ستلاحظ أن معظم الطلبات تعيد `500`، ولكن طلباً واحداً عند IP محدد (مثال: `192.168.0.142`) سيعيد رمز `200 OK`!
6. أرسل هذا الطلب الناجح إلى **Repeater** وعدله لتنفيذ الحذف:
`stockApi=http://192.168.0.142:8080/admin/delete?username=carlos` واضغط Send.

---

## 5. أدلة الترافيك (Evidence: Request / Response)

### نموذج طلب الاستغلال (Exploit Request):
```http
POST /product/stock HTTP/1.1
Host: <lab-id>.web-security-academy.net
Content-Type: application/x-www-form-urlencoded

stockApi=http://192.168.0.142:8080/admin/delete?username=carlos
```

### نموذج استجابة السيرفر (Server Response):
```http
HTTP/1.1 302 Found
Location: /admin
...
```

---

## 6. الأثر الواقعي للثغرة (Impact)

كشف هيكل الشبكة الداخلية بالكامل والوصول لقواعد البيانات وخوادم الإدارة غير المحمية بكلمات مرور.

---

## 7. الحماية والإغلاق البرمجي (Remediation)

حظر الاتصال بنطاقات الـ Private IP (`RFC 1918`) على مستوى السيرفر، وتفعيل المصادقة القوية (Mutual TLS / Auth) بين الخوادم الداخلية.

---

## 8. المراجع الرسمية (References)

- [PortSwigger Web Security Academy — SSRF](https://portswigger.net/web-security/ssrf)
- [OWASP Top 10 — A10:2021 Server-Side Request Forgery](https://owasp.org/Top10/A10_2021-Server-Side_Request_Forgery_%28SSRF%29/)
- [CWE-918: Server-Side Request Forgery](https://cwe.mitre.org/data/definitions/918.html)

---
*تم إعداد هذا التوثيق للأغراض التعليمية واختبار الاختراق الأخلاقي المصرح به.*
