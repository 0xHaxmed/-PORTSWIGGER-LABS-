# Lab 02 — Information Disclosure on Debug Page (كشف صفحة تصحيح أخطاء phpinfo)

[![التصنيف](https://img.shields.io/badge/الفئة-Information%20Disclosure-blue)]()
[![النوع](https://img.shields.io/badge/النوع-Information%20Disclosure%20(Exposed%20Debug%20Interface)-orange)]()
[![مستوى الخطورة](https://img.shields.io/badge/الخطورة-متوسط%20(Medium)-critical)]()
[![المصدر](https://img.shields.io/badge/المصدر-PortSwigger%20Academy-blue)](https://portswigger.net/web-security/information-disclosure)

> هذا الشرح جزء من سلسلة [معامل تسريب المعلومات (Information Disclosure)](./README.md) — أكاديمية PortSwigger.

---

## 1. بطاقة تعريف اللاب (Overview)

| الحقل | التفاصيل |
|---|---|
| **اسم اللاب** | Lab 02 — Information Disclosure on Debug Page (كشف صفحة تصحيح أخطاء phpinfo) |
| **تصنيف الثغرة** | Information Disclosure (Exposed Debug Interface) |
| **معيار الضعف (CWE)** | CWE-215: Insertion of Sensitive Information Into Debugging Code |
| **مستوى الخطورة** | متوسط (Medium) |
| **الهدف من اللاب** | العثور على رابط صفحة الـ Debug واستخراج المفتاح السري SECRET_KEY. |

---

## 2. فكرة اللاب وسيناريو المشكلة (Concept & Vulnerability Mechanism)

أثناء مرحلة برمجة وتطوير الموقع، قام أحد المطورين بإنشاء صفحة تشخيصية تستدعي دالة `phpinfo()`، ووضع رابطها داخل تعليق في كود الـ HTML في الصفحة الرئيسية لتسهيل الوصول إليها أثناء التجارب: `<!-- <a href="/cgi-bin/phpinfo.php">Debug</a> -->`.

**الخلل الفكري هنا:** نسي المطور حذف التعليق عند نشر الموقع في بيئة الإنتاج الحية! دالة `phpinfo()` بطبيعتها تعرض كل شيء عن السيرفر: مسارات النظام، إعدادات PHP، والأخطر من ذلك: جميع متغيرات البيئة (Environment Variables) التي تحتوي على المفاتيح السرية الحساسة مثل `SECRET_KEY`.

---

## 3. التحليل الجذري للسبب (Root Cause Analysis)

### 🔴 وجهة نظر المهاجم (Attacker Perspective):
المهاجم يفحص كود المصدر (Page Source) بحثاً عن تعليقات المطورين المنسية، ويعثر على رابط واجهة الـ Debug ويفتحه لاستخراج الأسرار.

### 🟢 وجهة نظر المدافع والمطور (Defender Perspective):
نشر ملفات تشخيصية في بيئة الإنتاج وترك إشارات إليها في تعليقات الـ HTML بدلاً من حذفها تماماً قبل مرحلة الإطلاق.

---

## 4. خطوات الاستغلال والحل بالتفصيل (Proof of Concept - PoC)

1. افتح الصفحة الرئيسية للموقع في المتصفح.
2. افتح كود المصدر بالضغط على `Ctrl+U` (View Page Source).
3. ابحث داخل كود الـ HTML عن كلمة `Debug` أو `phpinfo`.
4. ستجد تعليق HTML يحتوي على رابط شبيه بـ:
`<!-- <a href="/cgi-bin/phpinfo.php">Debug</a> -->`
5. انسخ المسار `/cgi-bin/phpinfo.php` وضعه في المتصفح بعد رابط اللاب.
6. ستفتح لك صفحة `phpinfo()` الكاملة الخاصة بالسيرفر.
7. اضغط `Ctrl+F` وابحث عن متغير البيئة `SECRET_KEY`.
8. انسخ قيمة المفتاح وقدمها في زر **Submit solution**.

---

## 5. أدلة الترافيك (Evidence: Request / Response)

### نموذج طلب الاستغلال (Exploit Request):
```http
GET /cgi-bin/phpinfo.php HTTP/1.1
Host: <lab-id>.web-security-academy.net
```

### نموذج استجابة السيرفر (Server Response):
```http
HTTP/1.1 200 OK

PHP Version 7.4.3
...
Environment
SECRET_KEY => "x7a9k2l1m4p8q0..."
```

---

## 6. الأثر الواقعي للثغرة (Impact)

تسريب المفاتيح السرية المستخدمة في توقيع الجلسات أو التشفير، مما يؤدي غالباً إلى تزوير الجلسات واختراق السيرفر.

---

## 7. الحماية والإغلاق البرمجي (Remediation)

حذف جميع ملفات وواجهات الـ Debug (مثل phpinfo ومسارات Swagger الداخلية) قبل رفع الكود لبيئة الإنتاج، واستخدام أدوات Linter لفحص وحذف تعليقات الـ HTML تلقائياً.

---

## 8. المراجع الرسمية (References)

- [PortSwigger Web Security Academy — Information Disclosure](https://portswigger.net/web-security/information-disclosure)
- [OWASP Top 10 — A05:2021 Security Misconfiguration](https://owasp.org/Top10/A05_2021-Security_Misconfiguration/)
- [CWE Database Details](https://cwe.mitre.org/)

---
*تم إعداد هذا التوثيق للأغراض التعليمية واختبار الاختراق الأخلاقي المصرح به.*
