# Lab 07 — User ID Controlled by Request Parameter with Data Leakage in Redirect (تسريب البيانات داخل رد التحويل 302)

[![التصنيف](https://img.shields.io/badge/الفئة-Broken%20Access%20Control-red)]()
[![النوع](https://img.shields.io/badge/النوع-Horizontal%20Broken%20Access%20Control%20(302%20Response%20Body%20Leak)-orange)]()
[![مستوى الخطورة](https://img.shields.io/badge/الخطورة-عالي%20(High)-critical)]()
[![المصدر](https://img.shields.io/badge/المصدر-PortSwigger%20Academy-blue)](https://portswigger.net/web-security/access-control)

> هذا الشرح جزء من سلسلة [معامل التحكم في الوصول (BAC)](./README.md) — أكاديمية PortSwigger.

---

## 1. بطاقة تعريف اللاب (Overview)

| الحقل | التفاصيل |
|---|---|
| **اسم اللاب** | Lab 07 — User ID Controlled by Request Parameter with Data Leakage in Redirect (تسريب البيانات داخل رد التحويل 302) |
| **تصنيف الثغرة** | Horizontal Broken Access Control (302 Response Body Leak) |
| **معيار الضعف (CWE)** | CWE-201: Insertion of Sensitive Information Into Sent Data |
| **مستوى الخطورة** | عالي (High) |
| **الهدف من اللاب** | استخراج الـ API Key الخاص بكارلوس من جسم استجابة التحويل 302 بعد محاولة الوصول لحسابه. |

---

## 2. فكرة اللاب وسيناريو المشكلة (Concept & Vulnerability Mechanism)

وضع المطور هنا كوداً لفحص الصلاحيات: إذا حاول المستخدم `wiener` فتح صفحة `carlos`، يقوم السيرفر بإصدار أمر تحويل (HTTP 302 Redirect) إلى صفحة تسجيل الدخول `/login`. عندما يرى المتصفح رمز `302`، فإنه ينفذ أمر التحويل فوراً دون أن يعرض محتوى الصفحة للمستخدم.

**الخلل الفكري هنا:** في لغات البرمجة (مثل PHP أو ASP)، عندما تكتب دالة التحويل `header("Location: /login");` ولم تتبعها فوراً بأمر إيقاف السكربت `exit();` أو `die();`، فإن السيرفر يستمر في تنفيذ باقي الكود! فيقوم بتوليد صفحة حساب كارلوس وكتابة الـ API Key داخل الـ Response Body، ثم يرسله مع هيدر 302. المتصفح العادي يخفي هذا المحتوى، ولكن أدوات مثل Burp Suite تلتقطه بالكامل!

---

## 3. التحليل الجذري للسبب (Root Cause Analysis)

### 🔴 وجهة نظر المهاجم (Attacker Perspective):
المهاجم يفحص استجابات السيرفر الخام في Burp Suite ولا يعتمد على ما يعرضه المتصفح فقط، فيكتشف أن السيرفر ولد محتوى الصفحة الحساسة قبل أمر التحويل.

### 🟢 وجهة نظر المدافع والمطور (Defender Perspective):
المطور نسي إنهاء دورة حياة الطلب (`exit;`) بعد إرسال ترويسة التحويل `302`، مما أدى لتنفيذ باقي السكربت وطباعة البيانات في جسم الرد.

---

## 4. خطوات الاستغلال والحل بالتفصيل (Proof of Concept - PoC)

1. سجل دخولك بحساب `wiener:peter` وافتح أداة **Burp Suite**.
2. قم بزيارة الرابط: `/my-account?id=carlos`.
3. في المتصفح، ستلاحظ أنه قام فوراً بإعادة توجيهك إلى صفحة تسجيل الدخول `/login` وتظن أن الصفحة محمية.
4. افتح **Burp Suite** واذهب إلى **Proxy -> HTTP history**.
5. ابحث عن الطلب الأصلي: `GET /my-account?id=carlos`.
6. افحص الـ **Response** الخاصة بهذا الطلب: ستجد أن الكود هو `302 Found`، ولكن إذا نظرت إلى أسفل الاستجابة في الـ Body، ستجد كود الـ HTML الخاص بصفحة carlos ومعه الـ API Key!
7. انسخ الـ API Key وقدّمه لحل اللاب.

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
HTTP/1.1 302 Found
Location: /login
Content-Type: text/html

<div>Your user name is: carlos</div>
<div>Your API Key is: leak_api_key_123</div>
```

---

## 6. الأثر الواقعي للثغرة (Impact)

تسريب بيانات المستخدمين المحمية ظاهرياً نتيجة أخطاء برمجية في ترتيب معالجة الاستجابات.

---

## 7. الحماية والإغلاق البرمجي (Remediation)

يجب دائماً إنهاء تنفيذ السكربت فورياً عند إصدار ردود التحويل أو المنع، والتأكد من عدم توليد أو كتابة أي بيانات في جسم الـ Response:
```php
header('Location: /login');
exit(); // ضروري جداً لمنع تسريب المحتوى
```

---

## 8. المراجع الرسمية (References)

- [PortSwigger Web Security Academy — Access Control](https://portswigger.net/web-security/access-control)
- [OWASP Top 10 — A01:2021 Broken Access Control](https://owasp.org/Top10/A01_2021-Broken_Access_Control/)
- [CWE Database Details](https://cwe.mitre.org/)

---
*تم إعداد هذا التوثيق للأغراض التعليمية واختبار الاختراق الأخلاقي المصرح به.*
