# Lab 06 — User ID Controlled by Request Parameter with Unpredictable IDs (IDOR بمعرفات عشوائية مسربة)

[![التصنيف](https://img.shields.io/badge/الفئة-Broken%20Access%20Control-red)]()
[![النوع](https://img.shields.io/badge/النوع-Horizontal%20Broken%20Access%20Control%20(IDOR%20with%20Enumerable%20GUID)-orange)]()
[![مستوى الخطورة](https://img.shields.io/badge/الخطورة-عالي%20(High)-critical)]()
[![المصدر](https://img.shields.io/badge/المصدر-PortSwigger%20Academy-blue)](https://portswigger.net/web-security/access-control)

> هذا الشرح جزء من سلسلة [معامل التحكم في الوصول (BAC)](./README.md) — أكاديمية PortSwigger.

---

## 1. بطاقة تعريف اللاب (Overview)

| الحقل | التفاصيل |
|---|---|
| **اسم اللاب** | Lab 06 — User ID Controlled by Request Parameter with Unpredictable IDs (IDOR بمعرفات عشوائية مسربة) |
| **تصنيف الثغرة** | Horizontal Broken Access Control (IDOR with Enumerable GUID) |
| **معيار الضعف (CWE)** | CWE-639: Authorization Bypass Through User-Controlled Key |
| **مستوى الخطورة** | عالي (High) |
| **الهدف من اللاب** | العثور على معرّف GUID الخاص بكارلوس، واستخدامه لقراءة الـ API Key الخاص به. |

---

## 2. فكرة اللاب وسيناريو المشكلة (Concept & Vulnerability Mechanism)

حاول المطور في هذا اللاب حماية المستخدمين من هجمات الـ IDOR باستبدال الأسماء والأرقام المتسلسلة بمعرفات عشوائية فريدة عالمياً (GUID / UUID) مثل `id=6c66cf1c-3b08-41db-9d0a-9d66572e9d5d`، على أمل أن المهاجم لن يستطيع تخمين هذا المعرّف الطويل.

**الخلل الفكري هنا:** المعرّف المعقد ليس بديلاً عن فحص الصلاحية! فالمعرّفات بطبيعتها يتم تسريبها في أماكن أخرى داخل التطبيق (مثل روابط كتاب المقالات في المدونة، أو التعليقات). بمجرد أن يعثر المهاجم على معرّف GUID الخاص بالضحية في مقال عام، يعود لاستخدامه في صفحة الحساب الشخصي لسرقة بياناته.

---

## 3. التحليل الجذري للسبب (Root Cause Analysis)

### 🔴 وجهة نظر المهاجم (Attacker Perspective):
المهاجم يستغل ميزة عامة في التطبيق (المدونة) لمعرفة المعرّف الفريد للضحية، ثم يستخدمه في نقطة النهاية المعرضة لـ IDOR.

### 🟢 وجهة نظر المدافع والمطور (Defender Perspective):
المطور اعتقد أن صعوبة تخمين المعرّف تغني عن فحص ملكية الحساب. معرفات الـ GUID ليست أسراراً مشفرة بل مجرد أرقام تعريفية تظهر في سياقات متعددة.

---

## 4. خطوات الاستغلال والحل بالتفصيل (Proof of Concept - PoC)

1. تصفح المدونة على الصفحة الرئيسية وابحث عن مقال كتبه المستخدم `carlos`.
2. اضغط على اسم الكاتب `carlos` في أسفل المقال أو افحص رابطه.
3. ستلاحظ أن الرابط يوجه إلى: `/blogs?userId=6c66cf1c-3b08-41db-9d0a-9d66572e9d5d`.
4. انسخ قيمة الـ GUID الخاص بـ carlos.
5. سجل دخولك بحساب `wiener:peter` وافتح صفحة **My account**.
6. في شريط الرابط، استبدل الـ GUID الخاص بحسابك بالـ GUID الخاص بكارلوس: `/my-account?id=6c66cf1c-3b08-41db-9d0a-9d66572e9d5d`.
7. ستفتح صفحة carlos وتكشف الـ API Key الخاص به، قم بنسخه وتقديمه لحل اللاب.

---

## 5. أدلة الترافيك (Evidence: Request / Response)

### نموذج طلب الاستغلال (Exploit Request):
```http
GET /my-account?id=6c66cf1c-3b08-41db-9d0a-9d66572e9d5d HTTP/1.1
Host: <lab-id>.web-security-academy.net
Cookie: session=...
```

### نموذج استجابة السيرفر (Server Response):
```http
HTTP/1.1 200 OK

Your user name is: carlos
Your API Key is: 1q2w3e4r...
```

---

## 6. الأثر الواقعي للثغرة (Impact)

تسريب كامل لحسابات المستخدمين المستهدفين بمجرد العثور على أي نشاط عام لهم على الموقع.

---

## 7. الحماية والإغلاق البرمجي (Remediation)

الـ GUID يمنع الهجمات التخمينية المتسلسلة (Enumeration)، ولكنه لا يغني عن فحص الصلاحيات. يجب أن يظل التحقق الخادمي من مطابقة المستخدم الحالي للسجل المستعلم عنه قائماً دائماً.

---

## 8. المراجع الرسمية (References)

- [PortSwigger Web Security Academy — Access Control](https://portswigger.net/web-security/access-control)
- [OWASP Top 10 — A01:2021 Broken Access Control](https://owasp.org/Top10/A01_2021-Broken_Access_Control/)
- [CWE Database Details](https://cwe.mitre.org/)

---
*تم إعداد هذا التوثيق للأغراض التعليمية واختبار الاختراق الأخلاقي المصرح به.*
