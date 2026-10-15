# Lab 03 — Source Code Disclosure via Backup Files (كشف كود المصدر عبر ملف نسخ احتياطي .bak)

[![التصنيف](https://img.shields.io/badge/الفئة-Information%20Disclosure-blue)]()
[![النوع](https://img.shields.io/badge/النوع-Information%20Disclosure%20(Backup%20File%20Exposure)-orange)]()
[![مستوى الخطورة](https://img.shields.io/badge/الخطورة-عالي%20(High)-critical)]()
[![المصدر](https://img.shields.io/badge/المصدر-PortSwigger%20Academy-blue)](https://portswigger.net/web-security/information-disclosure)

> هذا الشرح جزء من سلسلة [معامل تسريب المعلومات (Information Disclosure)](./README.md) — أكاديمية PortSwigger.

---

## 1. بطاقة تعريف اللاب (Overview)

| الحقل | التفاصيل |
|---|---|
| **اسم اللاب** | Lab 03 — Source Code Disclosure via Backup Files (كشف كود المصدر عبر ملف نسخ احتياطي .bak) |
| **تصنيف الثغرة** | Information Disclosure (Backup File Exposure) |
| **معيار الضعف (CWE)** | CWE-530: Exposure of Backup File to an Unauthorized Control Sphere |
| **مستوى الخطورة** | عالي (High) |
| **الهدف من اللاب** | العثور على ملف باك آب لكود جافا في مجلد مخفي، واستخراج كلمة مرور قاعدة البيانات المشفرة بداخله. |

---

## 2. فكرة اللاب وسيناريو المشكلة (Concept & Vulnerability Mechanism)

قام المطور بالتعديل على ملف كود جافا اسمه `ProductTemplate.java`. وقبل التعديل، قام بأخذ نسخة احتياطية يدوية وسماها `ProductTemplate.java.bak` وحفظها داخل مجلد `/backup`. ولكي يمنع محركات البحث من رؤية المجلد، وضع في `robots.txt`: `Disallow: /backup`.

**الخلل الفكري هنا:**
1. استخدام `robots.txt` لإخفاء المجلد نبه المهاجمين لوجوده.
2. خادم الويب (Apache) مهيأ لتنفيذ ملفات الكود المصدري، لكنه لا يعرف امتداد `.bak`، فيتعامل معه كملف نصي ثابت (Static Text File) ويقوم بتحميله للمتصفح صراحة دون تنفيذ، مما كشف الكود المصدري المكتوب بداخله بالكامل متضمناً كلمة مرور قاعدة البيانات المشفرة هارد كود (Hardcoded Password)!

---

## 3. التحليل الجذري للسبب (Root Cause Analysis)

### 🔴 وجهة نظر المهاجم (Attacker Perspective):
المهاجم يقرأ `robots.txt` ليكتشف مجلد `/backup`، ويستعرض محتوياته لتحميل ملف `.bak` وقراءة الباسورد من داخل الكود.

### 🟢 وجهة نظر المدافع والمطور (Defender Perspective):
حفظ ملفات النسخ الاحتياطية داخل مسارات الـ Web Root، وتضمين بيانات الاعتماد صريحة داخل الكود المصدري بدلاً من سحبها من مدير أسرار آمن.

---

## 4. خطوات الاستغلال والحل بالتفصيل (Proof of Concept - PoC)

1. افتح ملف `/robots.txt` في المتصفح.
2. ستجد سطر: `Disallow: /backup`.
3. توجه إلى الرابط: `/backup` في المتصفح.
4. ستجد أن خاصية عرض المجلدات (Directory Listing) مفعلة، ويظهر لك ملف اسمه: `ProductTemplate.java.bak`.
5. اضغط على الملف لتحميله وفتحه بأي محرر نصوص.
6. اقرأ كود الجافا، وستجد دالة الاتصال بقاعدة البيانات تحتوي على:
```java
Connection conn = DriverManager.getConnection("jdbc:postgresql:...", "postgres", "mypassword123");
```
7. انسخ كلمة مرور قاعدة البيانات وقدمها لحل اللاب.

---

## 5. أدلة الترافيك (Evidence: Request / Response)

### نموذج طلب الاستغلال (Exploit Request):
```http
GET /backup/ProductTemplate.java.bak HTTP/1.1
Host: <lab-id>.web-security-academy.net
```

### نموذج استجابة السيرفر (Server Response):
```http
HTTP/1.1 200 OK
Content-Type: text/plain

package data.product;
... postgres, "5d9a8c1f4e2b..." ...
```

---

## 6. الأثر الواقعي للثغرة (Impact)

تسريب الكود المصدري وكلمة مرور قاعدة البيانات، مما يمكن المهاجم من اختراق قاعدة البيانات مباشرة أو البحث عن ثغرات في الكود المسرب.

---

## 7. الحماية والإغلاق البرمجي (Remediation)

1. تعطيل ميزة Directory Listing.
2. حظر خادم الويب من إرسال ملفات `.bak` و `.old`.
3. حفظ النسخ الاحتياطية خارج مجلد الـ Web Root تماماً.
4. عدم وضع كلمات المرور أبداً داخل الكود المصدري واستخدام Environment Variables.

---

## 8. المراجع الرسمية (References)

- [PortSwigger Web Security Academy — Information Disclosure](https://portswigger.net/web-security/information-disclosure)
- [OWASP Top 10 — A05:2021 Security Misconfiguration](https://owasp.org/Top10/A05_2021-Security_Misconfiguration/)
- [CWE Database Details](https://cwe.mitre.org/)

---
*تم إعداد هذا التوثيق للأغراض التعليمية واختبار الاختراق الأخلاقي المصرح به.*
