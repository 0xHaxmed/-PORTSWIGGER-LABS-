# Lab 09 — Insecure Direct Object References (ملفات ثابتة متوقعة لدردشة الدعم)

[![التصنيف](https://img.shields.io/badge/الفئة-Broken%20Access%20Control-red)]()
[![النوع](https://img.shields.io/badge/النوع-Horizontal%20Broken%20Access%20Control%20(Static%20File%20IDOR)-orange)]()
[![مستوى الخطورة](https://img.shields.io/badge/الخطورة-عالي%20(High)-critical)]()
[![المصدر](https://img.shields.io/badge/المصدر-PortSwigger%20Academy-blue)](https://portswigger.net/web-security/access-control)

> هذا الشرح جزء من سلسلة [معامل التحكم في الوصول (BAC)](./README.md) — أكاديمية PortSwigger.

---

## 1. بطاقة تعريف اللاب (Overview)

| الحقل | التفاصيل |
|---|---|
| **اسم اللاب** | Lab 09 — Insecure Direct Object References (ملفات ثابتة متوقعة لدردشة الدعم) |
| **تصنيف الثغرة** | Horizontal Broken Access Control (Static File IDOR) |
| **معيار الضعف (CWE)** | CWE-639: Authorization Bypass Through User-Controlled Key |
| **مستوى الخطورة** | عالي (High) |
| **الهدف من اللاب** | العثور على سجل محادثة قديم يحتوي على كلمة مرور carlos، ثم تسجيل الدخول بحسابه. |

---

## 2. فكرة اللاب وسيناريو المشكلة (Concept & Vulnerability Mechanism)

يوفر الموقع ميزة الدردشة الحية مع الدعم الفني (Live Chat). عندما تنتهي المحادثة، يوفر الموقع زراً لتحميل تفريغ المحادثة (Download Transcript). يقوم النظام بحفظ كل محادثة في ملف نصي ثابت على السيرفر ويسميه برقم متسلسل: `1.txt`, `2.txt`, `3.txt`... إلخ.

**الخلل الفكري هنا:** الملفات النصية مخزنة في مجلد عام على السيرفر، وتحميلها يتم مباشرة عبر رابط ملف ثابت دون أن يمر ببرمجية تفحص هل هذا الملف يخص المستخدم الحالي أم لا. إذا كانت محادثتك هي رقم `3.txt`، يمكنك ببساطة تجربة `2.txt` لتحميل محادثة عميل آخر بالكامل!

---

## 3. التحليل الجذري للسبب (Root Cause Analysis)

### 🔴 وجهة نظر المهاجم (Attacker Perspective):
المهاجم يلاحظ أن مسار تحميل سجلات المحادثة يعتمد على اسم ملف متسلسل وثابت (`/download-transcript/2.txt`)، فيقوم بالتنقل التنازلي أو التصاعدي لقراءة سجلات الآخرين.

### 🟢 وجهة نظر المدافع والمطور (Defender Perspective):
حفظ السجلات الحساسة في مجلد خادمي متاح للعموم بأسماء متسلسلة يمكن تخمينها، دون فرض بوابة صلاحيات (Access Gate) على تحميل الملفات.

---

## 4. خطوات الاستغلال والحل بالتفصيل (Proof of Concept - PoC)

1. اضغط على خيار **Live chat** في شريط القائمة الرئيسي للموقع.
2. اكتب أي رسالة تجريبية في الشات ثم اضغط على زر **View transcript**.
3. ستلاحظ أن المتصفح يفتح رابطاً لتحميل الملف بصيغة: `/download-transcript/2.txt` (أو رقم مشابه).
4. في شريط الرابط، قم بتغيير الرقم إلى رقم سابق (مثلاً: `/download-transcript/1.txt`).
5. سيفتح لك سجل محادثة خاص بالمستخدم `carlos` يطلب فيه المساعدة ويعطيه موظف الدعم كلمة مروره المؤقتة!
6. انسخ كلمة المرور من داخل المحادثة.
7. اذهب لصفحة تسجيل الدخول وسجل كـ `carlos` بكلمة المرور المسروقة لحل اللاب.

---

## 5. أدلة الترافيك (Evidence: Request / Response)

### نموذج طلب الاستغلال (Exploit Request):
```http
GET /download-transcript/1.txt HTTP/1.1
Host: <lab-id>.web-security-academy.net
```

### نموذج استجابة السيرفر (Server Response):
```http
HTTP/1.1 200 OK

Carlos: I forgot my password.
Support: Your temporary password is: pass12345
```

---

## 6. الأثر الواقعي للثغرة (Impact)

تسريب سجلات محادثات العملاء التي قد تحتوي على كلمات مرور، أرقام بطاقات ائتمان، أو بيانات شخصية.

---

## 7. الحماية والإغلاق البرمجي (Remediation)

يجب عدم حفظ ملفات الدردشة بأسماء متسلسلة في مسارات عامة. يجب تخزينها في قاعدة بيانات أو مجلد محمي خارج الـ Web Root، وتمرير تحميل الملف عبر سكربت وسيط يتحقق من هوية صاحب الجلسة قبل إرسال الملف.

---

## 8. المراجع الرسمية (References)

- [PortSwigger Web Security Academy — Access Control](https://portswigger.net/web-security/access-control)
- [OWASP Top 10 — A01:2021 Broken Access Control](https://owasp.org/Top10/A01_2021-Broken_Access_Control/)
- [CWE Database Details](https://cwe.mitre.org/)

---
*تم إعداد هذا التوثيق للأغراض التعليمية واختبار الاختراق الأخلاقي المصرح به.*
