# Lab 10 — URL-based Access Control Can Be Circumvented (تجاوز فحص URL عبر ترويسات البروكسي)

[![التصنيف](https://img.shields.io/badge/الفئة-Broken%20Access%20Control-red)]()
[![النوع](https://img.shields.io/badge/النوع-Vertical%20Broken%20Access%20Control%20(Reverse%20Proxy%20/%20Edge%20Bypass)-orange)]()
[![مستوى الخطورة](https://img.shields.io/badge/الخطورة-حرج%20(Critical)-critical)]()
[![المصدر](https://img.shields.io/badge/المصدر-PortSwigger%20Academy-blue)](https://portswigger.net/web-security/access-control)

> هذا الشرح جزء من سلسلة [معامل التحكم في الوصول (BAC)](./README.md) — أكاديمية PortSwigger.

---

## 1. بطاقة تعريف اللاب (Overview)

| الحقل | التفاصيل |
|---|---|
| **اسم اللاب** | Lab 10 — URL-based Access Control Can Be Circumvented (تجاوز فحص URL عبر ترويسات البروكسي) |
| **تصنيف الثغرة** | Vertical Broken Access Control (Reverse Proxy / Edge Bypass) |
| **معيار الضعف (CWE)** | CWE-441: Unintended Proxy or Intermediary ('Confused Deputy') |
| **مستوى الخطورة** | حرج (Critical) |
| **الهدف من اللاب** | تجاوز حظر البروكسي على /admin باستخدام هيدر X-Original-URL وحذف carlos. |

---

## 2. فكرة اللاب وسيناريو المشكلة (Concept & Vulnerability Mechanism)

في هذه المعمارية، يقف خادم وسيط (Reverse Proxy / WAF) أمام تطبيق الويب. تم ضبط البروكسي بحيث إذا طلب المستخدم أي رابط يبدأ بـ `/admin`، يقوم البروكسي فوراً بحظر الطلب وإرجاع `403 Forbidden`.

**الخلل الفكري هنا:** خادم التطبيق الداخلي يدعم ترويسة تسمى `X-Original-URL` (أو `X-Rewrite-URL`) تُستخدم لإعادة كتابة المسار داخلياً. عندما يرسل المهاجم طلباً إلى الرابط المسموح `GET /` ويضع في الترويسات `X-Original-URL: /admin/delete?username=carlos`:
1. يرى البروكسي أن الطلب ذاهب إلى `/` (وهو مسموح به لعامة الناس)، فيمرره للباك إند.
2. يستقبل الباك إند الطلب ويقرأ ترويسة `X-Original-URL`، فيعيد توجيه التنفيذ داخلياً إلى مسار الإدارة ويحذف المستخدم دون أن يدري البروكسي!

---

## 3. التحليل الجذري للسبب (Root Cause Analysis)

### 🔴 وجهة نظر المهاجم (Attacker Perspective):
المهاجم يستغل الاختلاف في فهم الطلب بين الطبقة الخارجية (البروكسي) والطبقة الداخلية (الباك إند) بتمرير هيدر يعيد كتابة المسار خلسة.

### 🟢 وجهة نظر المدافع والمطور (Defender Perspective):
الاعتماد الكامل على البروكسي الخارجي لفحص المسارات، بينما خادم الباك إند مهيأ لدعم ترويسات إعادة التوجيه الداخلي دون التحقق من الصلاحيات محلياً.

---

## 4. خطوات الاستغلال والحل بالتفصيل (Proof of Concept - PoC)

1. جرب زيارة الرابط: `/admin` مباشرة في المتصفح، ستلاحظ ظهور شاشة خطأ `403 Forbidden` من البروكسي.
2. افتح **Burp Suite Repeater** وأرسل طلباً عادياً إلى الصفحة الرئيسية:
```http
GET / HTTP/1.1
Host: <lab-id>.web-security-academy.net
```
3. أضف الترويسة التالية في الطلب:
`X-Original-URL: /admin`
واضغط Send. ستجد أن السيرفر رد بـ `200 OK` وفتح لوحة الإدارة!
4. الآن، لتنفيذ أمر الحذف، عدل الترويسة لتصبح:
```http
GET /?username=carlos HTTP/1.1
Host: <lab-id>.web-security-academy.net
X-Original-URL: /admin/delete
```
5. أرسل الطلب، سيتم حذف carlos بنجاح ويحل اللاب.

---

## 5. أدلة الترافيك (Evidence: Request / Response)

### نموذج طلب الاستغلال (Exploit Request):
```http
GET /?username=carlos HTTP/1.1
Host: <lab-id>.web-security-academy.net
X-Original-URL: /admin/delete
```

### نموذج استجابة السيرفر (Server Response):
```http
HTTP/1.1 302 Found
Location: /admin
...
```

---

## 6. الأثر الواقعي للثغرة (Impact)

تجاوز تام لجميع جدران الحماية الخارجية (WAFs) والبروكسي، والوصول المباشر للوظائف الإدارية.

---

## 7. الحماية والإغلاق البرمجي (Remediation)

تعطيل دعم ترويسات `X-Original-URL` و `X-Rewrite-URL` في خوادم التطبيقات الداخلية، وتطبيق فحص الصلاحيات داخل كود التطبيق نفسه وليس على بوابة الدخول فقط.

---

## 8. المراجع الرسمية (References)

- [PortSwigger Web Security Academy — Access Control](https://portswigger.net/web-security/access-control)
- [OWASP Top 10 — A01:2021 Broken Access Control](https://owasp.org/Top10/A01_2021-Broken_Access_Control/)
- [CWE Database Details](https://cwe.mitre.org/)

---
*تم إعداد هذا التوثيق للأغراض التعليمية واختبار الاختراق الأخلاقي المصرح به.*
