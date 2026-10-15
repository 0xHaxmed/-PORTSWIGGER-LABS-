# Lab 05 — User ID Controlled by Request Parameter (ثغرة IDOR مباشرة عبر معرف المستخدم)

[![التصنيف](https://img.shields.io/badge/الفئة-Broken%20Access%20Control-red)]()
[![النوع](https://img.shields.io/badge/النوع-Horizontal%20Broken%20Access%20Control%20(Classic%20IDOR)-orange)]()
[![مستوى الخطورة](https://img.shields.io/badge/الخطورة-عالي%20(High)-critical)]()
[![المصدر](https://img.shields.io/badge/المصدر-PortSwigger%20Academy-blue)](https://portswigger.net/web-security/access-control)

> هذا الشرح جزء من سلسلة [معامل التحكم في الوصول (BAC)](./README.md) — أكاديمية PortSwigger.

---

## 1. بطاقة تعريف اللاب (Overview)

| الحقل | التفاصيل |
|---|---|
| **اسم اللاب** | Lab 05 — User ID Controlled by Request Parameter (ثغرة IDOR مباشرة عبر معرف المستخدم) |
| **تصنيف الثغرة** | Horizontal Broken Access Control (Classic IDOR) |
| **معيار الضعف (CWE)** | CWE-639: Authorization Bypass Through User-Controlled Key |
| **مستوى الخطورة** | عالي (High) |
| **الهدف من اللاب** | سرقة الـ API Key الخاص بالمستخدم carlos عن طريق استبدال اسم المستخدم في الرابط. |

---

## 2. فكرة اللاب وسيناريو المشكلة (Concept & Vulnerability Mechanism)

عند زيارة صفحة الحساب الشخصي، يقوم التطبيق بطلب الرابط: `GET /my-account?id=wiener`. في السيرفر، يقوم الكود بأخذ قيمة `id` مباشرة وعمل استعلام لقاعدة البيانات: `SELECT * FROM users WHERE username = :id` وعرض النتيجة في الصفحة بما فيها الـ API Key الحساس.

**الخلل الفكري هنا:** السيرفر لا يقوم بأي مطابقة بين المستخدم المسجل في الجلسة (Session) والمستخدم المطلوب في الباراميتر `id`. المطور افترض أن المستخدم لن يطلب إلا حسابه الشخصي! فإذا استبدل المهاجم `id=wiener` بـ `id=carlos`، يعرض له السيرفر بيانات كارلوس بالكامل دون أدنى اعتراض.

---

## 3. التحليل الجذري للسبب (Root Cause Analysis)

### 🔴 وجهة نظر المهاجم (Attacker Perspective):
المهاجم يلاحظ وجود معرّف مباشر في الرابط (`id=wiener`)، فيقوم بتعديله لمعرف مستخدم آخر مستهدف لقراءة بياناته الحساسة.

### 🟢 وجهة نظر المدافع والمطور (Defender Perspective):
غياب التحقق من الملكية (Ownership Check): السيرفر يثق في الباراميتر بدلاً من استخراج هوية المستخدم المطلوب من الـ Session الموثوقة الخاصة به.

---

## 4. خطوات الاستغلال والحل بالتفصيل (Proof of Concept - PoC)

1. سجل الدخول بحساب `wiener:peter`.
2. اضغط على **My account**، ولاحظ في شريط المتصفح أن الرابط هو: `/my-account?id=wiener`.
3. لاحظ أن الصفحة تعرض مفتاح الـ API الخاص بـ wiener.
4. عدّل شريط الرابط مباشرة ليكون: `/my-account?id=carlos` واضغط Enter.
5. ستفتح صفحة حساب carlos بالكامل ويظهر لك الـ API Key الخاص به!
6. انسخ الـ API Key وقدمه في زر **Submit solution** في أعلى الصفحة لحل اللاب.

---

## 5. أدلة الترافيك (Evidence: Request / Response)

### نموذج طلب الاستغلال (Exploit Request):
```http
GET /my-account?id=carlos HTTP/1.1
Host: <lab-id>.web-security-academy.net
Cookie: session=...
```

### نموذج استجابة السيرفر (Server Response):
```http
HTTP/1.1 200 OK

Your user name is: carlos
Your API Key is: d8f3a9e2...
```

---

## 6. الأثر الواقعي للثغرة (Impact)

تسريب معلومات وبيانات شخصية حساسة ومفاتيح واجهات برمجية خاصة بأي مستخدم مسجل على المنصة.

---

## 7. الحماية والإغلاق البرمجي (Remediation)

يجب ألا تعتمد صفحة الحساب على باراميتر خارجي يُرسله المستخدم لتحديد حسابه. بل يجب أن تجلب البيانات حصراً باستخدام معرّف المستخدم المخزن في الجلسة: `SELECT * FROM users WHERE id = $_SESSION['user_id']`.

---

## 8. المراجع الرسمية (References)

- [PortSwigger Web Security Academy — Access Control](https://portswigger.net/web-security/access-control)
- [OWASP Top 10 — A01:2021 Broken Access Control](https://owasp.org/Top10/A01_2021-Broken_Access_Control/)
- [CWE Database Details](https://cwe.mitre.org/)

---
*تم إعداد هذا التوثيق للأغراض التعليمية واختبار الاختراق الأخلاقي المصرح به.*
