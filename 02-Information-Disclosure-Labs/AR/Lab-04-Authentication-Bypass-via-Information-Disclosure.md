# Lab 04 — Authentication Bypass via Information Disclosure (تجاوز تسجيل الدخول بتسريب هيدر الثقة عبر TRACE)

[![التصنيف](https://img.shields.io/badge/الفئة-Information%20Disclosure-blue)]()
[![النوع](https://img.shields.io/badge/النوع-Information%20Disclosure%20to%20Authentication%20Bypass-orange)]()
[![مستوى الخطورة](https://img.shields.io/badge/الخطورة-حرج%20(Critical)-critical)]()
[![المصدر](https://img.shields.io/badge/المصدر-PortSwigger%20Academy-blue)](https://portswigger.net/web-security/information-disclosure)

> هذا الشرح جزء من سلسلة [معامل تسريب المعلومات (Information Disclosure)](./README.md) — أكاديمية PortSwigger.

---

## 1. بطاقة تعريف اللاب (Overview)

| الحقل | التفاصيل |
|---|---|
| **اسم اللاب** | Lab 04 — Authentication Bypass via Information Disclosure (تجاوز تسجيل الدخول بتسريب هيدر الثقة عبر TRACE) |
| **تصنيف الثغرة** | Information Disclosure to Authentication Bypass |
| **معيار الضعف (CWE)** | CWE-290: Authentication Bypass by Spoofing |
| **مستوى الخطورة** | حرج (Critical) |
| **الهدف من اللاب** | استخدام طريقة HTTP TRACE لاكتشاف ترويسة التوجيه الداخلية، ثم تزويرها لفتح لوحة الأدمن وحذف carlos. |

---

## 2. فكرة اللاب وسيناريو المشكلة (Concept & Vulnerability Mechanism)

لوحة الإدارة `/admin` محمية بحيث ترفض أي اتصال قادم من الإنترنت وتسمح فقط لمن يتصل من الـ IP الداخلي `127.0.0.1`. عندما يأتي طلب من الخارج، يقوم البروكسي الداخلي بفحص عنوان المستخدم، وإذا كان المستخدم محلياً يضيف ترويسة HTTP سرية خاصة (مثل `X-Custom-IP-Authorization: 127.0.0.1`) ويمررها للباك إند ليثبت له أن الطلب محلي.

**الخلل الفكري هنا:** خادم الويب يتيح استخدام طريقة `TRACE` في طلبات HTTP! وظيفة `TRACE` هي أن السيرفر يأخذ الطلب كما استلمه ويعيد طباعته في الرد. عندما أرسل المهاجم طلب `TRACE /admin`:
قام البروكسي بإضافة ترويسة الثقة السرية `X-Custom-IP-Authorization: <attacker-ip>`، ثم أعاد السيرفر طباعة الطلب كاملاً في الرد للمهاجم! عرف المهاجم اسم الهيدر السري، فقام بإضافته يدوياً بقيمة `127.0.0.1` وتجاوز الحماية بالكامل!

---

## 3. التحليل الجذري للسبب (Root Cause Analysis)

### 🔴 وجهة نظر المهاجم (Attacker Perspective):
المهاجم يرسل طلب `TRACE` ليرى الترويسات التي يضيفها البروكسي الداخلي في الخفاء، ثم يزور هذه الترويسات في طلبات عادية لخداع الباك إند.

### 🟢 وجهة نظر المدافع والمطور (Defender Perspective):
السماح بطريقة `TRACE` التشخيصية في بيئة الإنتاج، والاعتماد على هيدر HTTP قابل للتزوير للتحقق من هوية وموقع المتصل.

---

## 4. خطوات الاستغلال والحل بالتفصيل (Proof of Concept - PoC)

1. حاول زيارة `/admin` في المتصفح، ستجد رسالة حظر:
`Admin interface only available to local users`.
2. افتح **Burp Suite Repeater**، وغير طريقة الطلب من `GET /admin` إلى `TRACE /admin` واضغط Send.
3. انظر إلى استجابة السيرفر: ستجد أن السيرفر أعاد طباعة الطلب بعد أن أضاف إليه البروكسي ترويسة داخلية خاصة:
`X-Custom-IP-Authorization: <your-public-ip>`.
4. الآن عرفت اسم الترويسة السرية! أعد تحويل الطلب إلى `GET /admin`.
5. أضف الترويسة التالية بنفسك:
`X-Custom-IP-Authorization: 127.0.0.1` واضغط Send.
6. ستجد أن السيرفر ظن أنك متصل من داخل السيرفر نفسه وفتح لك لوحة الإدارة `200 OK`!
7. أرسل طلب حذف المستخدم مع نفس الهيدر:
`GET /admin/delete?username=carlos`.

---

## 5. أدلة الترافيك (Evidence: Request / Response)

### نموذج طلب الاستغلال (Exploit Request):
```http
GET /admin/delete?username=carlos HTTP/1.1
Host: <lab-id>.web-security-academy.net
X-Custom-IP-Authorization: 127.0.0.1
```

### نموذج استجابة السيرفر (Server Response):
```http
HTTP/1.1 302 Found
Location: /admin
...
```

---

## 6. الأثر الواقعي للثغرة (Impact)

تجاوز كامل لنظام الصلاحيات والتحقق من الهوية واختراق لوحة التحكم الإدارية عبر تزوير ترويسات الثقة.

---

## 7. الحماية والإغلاق البرمجي (Remediation)

1. تعطيل طريقة HTTP TRACE فوراً في خادم الويب (`TraceEnable Off`).
2. تجريد وتصفية (Strip/Sanitize) أي ترويسة `X-Custom-*` قادمة من المستخدمين الخارجيين على مستوى البروكسي قبل وصولها للباك إند.

---

## 8. المراجع الرسمية (References)

- [PortSwigger Web Security Academy — Information Disclosure](https://portswigger.net/web-security/information-disclosure)
- [OWASP Top 10 — A05:2021 Security Misconfiguration](https://owasp.org/Top10/A05_2021-Security_Misconfiguration/)
- [CWE Database Details](https://cwe.mitre.org/)

---
*تم إعداد هذا التوثيق للأغراض التعليمية واختبار الاختراق الأخلاقي المصرح به.*
