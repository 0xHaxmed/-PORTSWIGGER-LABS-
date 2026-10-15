# Lab 13 — Referer-based Access Control (التحقق الخاطئ بالاعتماد على هيدر Referer)

[![التصنيف](https://img.shields.io/badge/الفئة-Broken%20Access%20Control-red)]()
[![النوع](https://img.shields.io/badge/النوع-Vertical%20Broken%20Access%20Control%20(Referer%20Header%20Spoofing)-orange)]()
[![مستوى الخطورة](https://img.shields.io/badge/الخطورة-عالي%20(High)-critical)]()
[![المصدر](https://img.shields.io/badge/المصدر-PortSwigger%20Academy-blue)](https://portswigger.net/web-security/access-control)

> هذا الشرح جزء من سلسلة [معامل التحكم في الوصول (BAC)](./README.md) — أكاديمية PortSwigger.

---

## 1. بطاقة تعريف اللاب (Overview)

| الحقل | التفاصيل |
|---|---|
| **اسم اللاب** | Lab 13 — Referer-based Access Control (التحقق الخاطئ بالاعتماد على هيدر Referer) |
| **تصنيف الثغرة** | Vertical Broken Access Control (Referer Header Spoofing) |
| **معيار الضعف (CWE)** | CWE-293: Using Referer Field for Authentication |
| **مستوى الخطورة** | عالي (High) |
| **الهدف من اللاب** | ترقية حساب wiener إلى رتبة أدمن بتزوير هيدر Referer في طلب الترقية. |

---

## 2. فكرة اللاب وسيناريو المشكلة (Concept & Vulnerability Mechanism)

يقوم السيرفر بمعالجة طلبات ترقية المستخدمين عبر الرابط: `GET /admin-roles?username=wiener&action=upgrade`. أراد المطور التأكد من أن هذا الطلب جاء من لوحة الإدارة `/admin` فقط، فقام بفحص ترويسة `Referer` في الطلب: إذا كانت تحتوي على `/admin` يسمح بالترقية، وإذا لم تكن كذلك يرفض الطلب.

**الخلل الفكري هنا:** ترويسة `Referer` هي ترويسة يرسلها المتصفح، وهي تقع بالكامل تحت سيطرة المستخدم والعميل! يستطيع أي مهاجم كتابة أي قيمة يريدها داخل هيدر `Referer` عبر Burp Suite أو أي أداة أتمتة، وبالتالي فإن جعلها شرطاً للأمان هو خطأ تصميمي فادح.

---

## 3. التحليل الجذري للسبب (Root Cause Analysis)

### 🔴 وجهة نظر المهاجم (Attacker Perspective):
المهاجم يضيف أو يعدل هيدر `Referer: .../admin` في طلبه بجلسة حسابه العادي ليخدع خادم التطبيق ويجعله يعتقد أن الطلب صدر من داخل لوحة الإدارة.

### 🟢 وجهة نظر المدافع والمطور (Defender Perspective):
المطور استخدم هيدر توجيهي مخصص للمعلومات والإحصائيات (`Referer`) كأداة للتحقق من هوية وصلاحية مصدر الطلب.

---

## 4. خطوات الاستغلال والحل بالتفصيل (Proof of Concept - PoC)

1. سجل دخولك بحساب `administrator:admin` وقم بترقية أي حساب لمراقبة الطلب.
2. ستلاحظ أن الطلب يرسل هيدر:
`Referer: https://<lab-id>.web-security-academy.net/admin`
3. أرسل الطلب إلى **Burp Repeater**.
4. قم بتغيير اسم المستخدم إلى `wiener`، واستبدل كوكي الـ Session بجلسة المستخدم العادي `wiener`.
5. تأكد من وجود ترويسة الـ Referer كاملة تشير إلى رابط الـ `/admin`:
`Referer: https://<lab-id>.web-security-academy.net/admin`
6. اضغط Send، ستجد أن السيرفر اعتمد الترويسة ونفذ عملية الترقية بنجاح!

---

## 5. أدلة الترافيك (Evidence: Request / Response)

### نموذج طلب الاستغلال (Exploit Request):
```http
GET /admin-roles?username=wiener&action=upgrade HTTP/1.1
Host: <lab-id>.web-security-academy.net
Cookie: session=wiener_session
Referer: https://<lab-id>.web-security-academy.net/admin
```

### نموذج استجابة السيرفر (Server Response):
```http
HTTP/1.1 302 Found
Location: /
...
```

---

## 6. الأثر الواقعي للثغرة (Impact)

تجاوز تام لضوابط الوصول وترقية الحسابات عبر تزوير ترويسات HTTP عادية.

---

## 7. الحماية والإغلاق البرمجي (Remediation)

لا تعتمد أبداً على ترويسة `Referer` في اتخاذ أي قرار أمني أو إداري. التحقق من الصلاحيات يجب أن يستند حصراً إلى بيانات جلسة المستخدم الموثقة في السيرفر.

---

## 8. المراجع الرسمية (References)

- [PortSwigger Web Security Academy — Access Control](https://portswigger.net/web-security/access-control)
- [OWASP Top 10 — A01:2021 Broken Access Control](https://owasp.org/Top10/A01_2021-Broken_Access_Control/)
- [CWE Database Details](https://cwe.mitre.org/)

---
*تم إعداد هذا التوثيق للأغراض التعليمية واختبار الاختراق الأخلاقي المصرح به.*
