# Lab 03 — User Role Controlled by Request Parameter (التحكم بالرتبة عبر باراميتر الطلب / الكوكي)

[![التصنيف](https://img.shields.io/badge/الفئة-Broken%20Access%20Control-red)]()
[![النوع](https://img.shields.io/badge/النوع-Vertical%20Broken%20Access%20Control%20(تصعيد%20صلاحيات%20رأسي)-orange)]()
[![مستوى الخطورة](https://img.shields.io/badge/الخطورة-حرج%20(Critical)-critical)]()
[![المصدر](https://img.shields.io/badge/المصدر-PortSwigger%20Academy-blue)](https://portswigger.net/web-security/access-control)

> هذا الشرح جزء من سلسلة [معامل التحكم في الوصول (BAC)](./README.md) — أكاديمية PortSwigger.

---

## 1. بطاقة تعريف اللاب (Overview)

| الحقل | التفاصيل |
|---|---|
| **اسم اللاب** | Lab 03 — User Role Controlled by Request Parameter (التحكم بالرتبة عبر باراميتر الطلب / الكوكي) |
| **تصنيف الثغرة** | Vertical Broken Access Control (تصعيد صلاحيات رأسي) |
| **معيار الضعف (CWE)** | CWE-565: Reliance on Cookies without Validation and Integrity Checking |
| **مستوى الخطورة** | حرج (Critical) |
| **الهدف من اللاب** | الوصول للوحة الإدارة وحذف carlos بتعديل كوكي الرتبة. |

---

## 2. فكرة اللاب وسيناريو المشكلة (Concept & Vulnerability Mechanism)

في هذا السيناريو، يقوم التطبيق بتحديد صلاحية المستخدم بناءً على قيمة مخزنة في كوكي المتصفح (Cookie) اسمها `Admin`. عندما يسجل المستخدم العادي دخوله، يرسل له السيرفر كوكي `Admin=false`. وعندما يطلب المستخدم أي صفحة، يفحص السيرفر الكوكي: إذا كانت `Admin=true` يعطيه صلاحية الإدارة، وإذا كانت `false` يمنعه.

**الخلل الفكري هنا:** المطور وثق في قيمة مرسلة من جهاز العميل لتحديد الصلاحية! بما أن الكوكيز مخزنة على جهاز المستخدم، فإن المستخدم يمتلك حق تعديلها لأي قيمة يريدها عبر أدوات المطورين أو Burp Suite.

---

## 3. التحليل الجذري للسبب (Root Cause Analysis)

### 🔴 وجهة نظر المهاجم (Attacker Perspective):
المهاجم يراقب الكوكيز المرسلة مع كل طلب، ويلاحظ وجود متغير منطقي بسيط (`Admin=false`)، فيقوم بتغييره إلى `true` للحصول على رتبة الأدمن فوراً.

### 🟢 وجهة نظر المدافع والمطور (Defender Perspective):
المطور جعل جهة العميل (Client) هي مصدر الحقيقة لتحديد رتبة المستخدم، بدلاً من حفظ الرتبة في جلسة آمنة على السيرفر (Server-Side Session) مرتبطة بقاعدة البيانات.

---

## 4. خطوات الاستغلال والحل بالتفصيل (Proof of Concept - PoC)

1. سجل الدخول بالحساب المتاح: `wiener:peter`.
2. افتح أداة **Burp Suite** واذهب إلى تبويب **HTTP history** وراقب الطلبات.
3. ستلاحظ وجود هيدر `Cookie` في الطلب يحتوي على: `Cookie: Admin=false; session=...`.
4. أرسل أي طلب إلى **Burp Repeater** (`Ctrl+R`).
5. قم بتعديل الكوكي لتصبح: `Admin=true`، ثم أرسل طلباً للمسار `/admin`.
6. ستجد أن السيرفر أعاد استجابة `200 OK` وفتح لوحة الإدارة!
7. أرسل طلب حذف المستخدم: `GET /admin/delete?username=carlos` مع التأكد من إبقاء `Admin=true`.

---

## 5. أدلة الترافيك (Evidence: Request / Response)

### نموذج طلب الاستغلال (Exploit Request):
```http
GET /admin/delete?username=carlos HTTP/1.1
Host: <lab-id>.web-security-academy.net
Cookie: Admin=true; session=abc123xyz
```

### نموذج استجابة السيرفر (Server Response):
```http
HTTP/1.1 302 Found
Location: /admin
...
```

---

## 6. الأثر الواقعي للثغرة (Impact)

تصعيد صلاحيات فوري وكامل لأي مستخدم عادي ليصبح مديراً للنظام بالكامل.

---

## 7. الحماية والإغلاق البرمجي (Remediation)

يجب ألا تُحدد الصلاحيات أبداً عبر كوكيز قابلة للتعديل. يجب حفظ دور المستخدم في جدول المستخدمين بقاعدة البيانات، وتحميله في جلسة السيرفر المؤمنة (Signed/Encrypted Session) المشفرة بمفتاح سري لا يعلمه إلا الخادم.

---

## 8. المراجع الرسمية (References)

- [PortSwigger Web Security Academy — Access Control](https://portswigger.net/web-security/access-control)
- [OWASP Top 10 — A01:2021 Broken Access Control](https://owasp.org/Top10/A01_2021-Broken_Access_Control/)
- [CWE Database Details](https://cwe.mitre.org/)

---
*تم إعداد هذا التوثيق للأغراض التعليمية واختبار الاختراق الأخلاقي المصرح به.*
