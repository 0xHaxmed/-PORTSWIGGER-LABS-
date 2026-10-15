# Lab 08 — User ID Controlled by Request Parameter with Password Disclosure (تسريب كلمة مرور الأدمن عبر IDOR)

[![التصنيف](https://img.shields.io/badge/الفئة-Broken%20Access%20Control-red)]()
[![النوع](https://img.shields.io/badge/النوع-Horizontal-to-Vertical%20Privilege%20Escalation%20(Password%20Disclosure)-orange)]()
[![مستوى الخطورة](https://img.shields.io/badge/الخطورة-حرج%20(Critical)-critical)]()
[![المصدر](https://img.shields.io/badge/المصدر-PortSwigger%20Academy-blue)](https://portswigger.net/web-security/access-control)

> هذا الشرح جزء من سلسلة [معامل التحكم في الوصول (BAC)](./README.md) — أكاديمية PortSwigger.

---

## 1. بطاقة تعريف اللاب (Overview)

| الحقل | التفاصيل |
|---|---|
| **اسم اللاب** | Lab 08 — User ID Controlled by Request Parameter with Password Disclosure (تسريب كلمة مرور الأدمن عبر IDOR) |
| **تصنيف الثغرة** | Horizontal-to-Vertical Privilege Escalation (Password Disclosure) |
| **معيار الضعف (CWE)** | CWE-522: Insufficiently Protected Credentials |
| **مستوى الخطورة** | حرج (Critical) |
| **الهدف من اللاب** | استخراج كلمة مرور administrator المسربة داخل صفحة حسابه، ثم تسجيل الدخول كأدمن وحذف carlos. |

---

## 2. فكرة اللاب وسيناريو المشكلة (Concept & Vulnerability Mechanism)

في صفحة الحساب الشخصي `/my-account?id=administrator`، توجد استمارة لتغيير كلمة المرور. قام المطور بقرار تصميمي خاطئ (UX Flaw) بجعل حقل كلمة المرور ممتلئاً مسبقاً (Pre-filled) بكلمة المرور الحالية للمستخدم المشفرة داخل الـ HTML بصيغة: `<input type="password" name="password" value="s3cr3tP@ss">`.

**الخلل الفكري هنا:** اجتمع خطآن قاتلان: الأول هو وجود ثغرة IDOR تسمح لـ wiener بطلب حساب `id=administrator`. والخطأ الثاني هو أن السيرفر يرسل كلمة المرور الحقيقية في كود الـ HTML. ورغم أن المتصفح يعرضها على شكل نجوم (••••••)، إلا أن المهاجم يمكنه رؤيتها صريحة في كود الـ Source Code بمجرد فحص العنصر (Inspect Element)!

---

## 3. التحليل الجذري للسبب (Root Cause Analysis)

### 🔴 وجهة نظر المهاجم (Attacker Perspective):
المهاجم يدمج ثغرة IDOR للوصول لحساب الأدمن، ثم يفحص كود المصدر ليستخرج كلمة المرور الصريحة المضمنة في خاصية `value` لحقل الإدخال.

### 🟢 وجهة نظر المدافع والمطور (Defender Perspective):
المطور قام بتغذية حقول كلمات المرور بالقيمة المخزنة لتسهيل التعديل على المستخدم، متجاهلاً أن كلمات المرور لا يجب إرجاعها للمتصفح مطلقاً بعد إدخالها لأول مرة.

---

## 4. خطوات الاستغلال والحل بالتفصيل (Proof of Concept - PoC)

1. سجل دخولك بحساب `wiener:peter`.
2. اذهب للرابط: `/my-account?id=administrator`.
3. ستفتح لك صفحة حساب المدير وفيها حقول تغيير كلمة المرور ممتلئة بنجوم.
4. اضغط بزر الفأرة الأيمن على حقل كلمة المرور واختر **Inspect Element** (أو افحص الـ Source Code).
5. ستجد وسم الـ HTML كالتالي:
```html
<input type="password" name="password" value="super_secret_admin_pass">
```
6. انسخ كلمة المرور الحقيقية للأدمن الظاهرة في `value`.
7. سجل خروجك، ثم سجل دخولك بالحساب: `administrator` وكلمة المرور التي نسختها.
8. توجه إلى لوحة الإدارة `/admin` واحذف المستخدم `carlos`.

---

## 5. أدلة الترافيك (Evidence: Request / Response)

### نموذج طلب الاستغلال (Exploit Request):
```http
GET /my-account?id=administrator HTTP/1.1
Host: <lab-id>.web-security-academy.net
Cookie: session=...
```

### نموذج استجابة السيرفر (Server Response):
```http
HTTP/1.1 200 OK

<input type="password" name="password" value="ab12cd34ef56">
```

---

## 6. الأثر الواقعي للثغرة (Impact)

تصعيد صلاحيات كامل والسيطرة على الحساب الإداري الرئيسي للموقع عبر تسريب الاعتمادات.

---

## 7. الحماية والإغلاق البرمجي (Remediation)

1. إصلاح ثغرة الـ IDOR بربط صفحة الحساب بجلسة المستخدم الحالي فقط.
2. حقول كلمات المرور يجب ألا تُملأ مسبقاً بقيمة كلمة المرور إطلاقاً، ويجب أن تترك فارغة دائماً.

---

## 8. المراجع الرسمية (References)

- [PortSwigger Web Security Academy — Access Control](https://portswigger.net/web-security/access-control)
- [OWASP Top 10 — A01:2021 Broken Access Control](https://owasp.org/Top10/A01_2021-Broken_Access_Control/)
- [CWE Database Details](https://cwe.mitre.org/)

---
*تم إعداد هذا التوثيق للأغراض التعليمية واختبار الاختراق الأخلاقي المصرح به.*
