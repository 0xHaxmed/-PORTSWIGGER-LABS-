# Lab 01 — Information Disclosure in Error Messages (تسريب إصدار الفريم وورك في رسائل الخطأ)

[![التصنيف](https://img.shields.io/badge/الفئة-Information%20Disclosure-blue)]()
[![النوع](https://img.shields.io/badge/النوع-Information%20Disclosure%20(Stack%20Trace%20&%20Version%20Leak)-orange)]()
[![مستوى الخطورة](https://img.shields.io/badge/الخطورة-منخفض%20(Low)%20/%20تمهيدي-critical)]()
[![المصدر](https://img.shields.io/badge/المصدر-PortSwigger%20Academy-blue)](https://portswigger.net/web-security/information-disclosure)

> هذا الشرح جزء من سلسلة [معامل تسريب المعلومات (Information Disclosure)](./README.md) — أكاديمية PortSwigger.

---

## 1. بطاقة تعريف اللاب (Overview)

| الحقل | التفاصيل |
|---|---|
| **اسم اللاب** | Lab 01 — Information Disclosure in Error Messages (تسريب إصدار الفريم وورك في رسائل الخطأ) |
| **تصنيف الثغرة** | Information Disclosure (Stack Trace & Version Leak) |
| **معيار الضعف (CWE)** | CWE-209: Generation of Error Message Containing Sensitive Information |
| **مستوى الخطورة** | منخفض (Low) / تمهيدي |
| **الهدف من اللاب** | استخراج رقم الإصدار الدقيق للـ Framework من خلال رسالة خطأ وتسليمه لحل اللاب. |

---

## 2. فكرة اللاب وسيناريو المشكلة (Concept & Vulnerability Mechanism)

يقوم تطبيق الويب بعرض المنتجات بناءً على رقم المنتج الممرر في الرابط `GET /product?productId=1`. في الباك إند، يقوم التطبيق بتحويل القيمة المدخلة إلى رقم صحيح (Integer) باستخدام دالة برمجية مثل `Integer.parseInt()`.

**الخلل الفكري هنا:** لم يقم المطور بوضع كود لمعالجة الأخطاء (Exception Handling). عندما يرسل المستخدم قيمة نصية بدلاً من رقم (مثل `productId=abc`)، يفشل البرنامج ويحدث استثناء غير معالج (Unhandled Exception). ونظراً لأن وضع التصحيح (Debug) مفعل، يقوم السيرفر بطباعة الـ Stack Trace بالكامل في المتصفح، متضمناً اسم الفريم وورك وإصداره الدقيق (Apache Struts 2 2.3.31).

---

## 3. التحليل الجذري للسبب (Root Cause Analysis)

### 🔴 وجهة نظر المهاجم (Attacker Perspective):
المهاجم يغير نوع المدخل المتوقع لإجبار النظام على الانهيار واستخراج الـ Stack Trace لمعرفة الإصدار ومطابقته بثغرات 1-day.

### 🟢 وجهة نظر المدافع والمطور (Defender Perspective):
غياب الـ try-catch على تحويل البيانات وعدم وجود صفحة أخطاء عامة مخصصة لبيئة الإنتاج.

---

## 4. خطوات الاستغلال والحل بالتفصيل (Proof of Concept - PoC)

1. افتح أي منتج في المتصفح ولاحظ الرابط: `/product?productId=1`.
2. اعترض الطلب أو أرسله إلى **Burp Repeater**.
3. غيّر قيمة الباراميتر `productId` من رقم `1` إلى نص عشوائي غير متوقع: `productId=test`.
4. أرسل الطلب، ولاحظ أن السيرفر أعاد استجابة `500 Internal Server Error`.
5. افحص جسم الاستجابة: ستجد رسالة خطأ تفصيلية تحتوي على السطر التالي:
`Apache Struts 2 2.3.31`.
6. انسخ رقم الإصدار `2.3.31` واضغط على زر **Submit solution** في اللاب وقدمه كحل.

---

## 5. أدلة الترافيك (Evidence: Request / Response)

### نموذج طلب الاستغلال (Exploit Request):
```http
GET /product?productId=test HTTP/1.1
Host: <lab-id>.web-security-academy.net
```

### نموذج استجابة السيرفر (Server Response):
```http
HTTP/1.1 500 Internal Server Error

java.lang.NumberFormatException: For input string: "test"
  at java.lang.NumberFormatException.forInputString
  ... Apache Struts 2 2.3.31 ...
```

---

## 6. الأثر الواقعي للثغرة (Impact)

تمكين المهاجمين من معرفة الحزمة البرمجية وإصدارها، مما يفتح الباب لاستخدام ثغرات RCE شهيرة مطابقة للإصدار.

---

## 7. الحماية والإغلاق البرمجي (Remediation)

تغليف جميع عمليات التحويل والمدخلات بكتل `try-catch` آمنة، وضبط السيرفر ليعرض صفحات أخطاء عامة (Generic 500 Error Page) لا تحتوي على أي تفاصيل فنية.

---

## 8. المراجع الرسمية (References)

- [PortSwigger Web Security Academy — Information Disclosure](https://portswigger.net/web-security/information-disclosure)
- [OWASP Top 10 — A05:2021 Security Misconfiguration](https://owasp.org/Top10/A05_2021-Security_Misconfiguration/)
- [CWE Database Details](https://cwe.mitre.org/)

---
*تم إعداد هذا التوثيق للأغراض التعليمية واختبار الاختراق الأخلاقي المصرح به.*
