# Lab 02 — Unprotected Admin Functionality with Unpredictable URL (لوحة إدارة برابط عشوائي غير متوقع)

[![التصنيف](https://img.shields.io/badge/الفئة-Broken%20Access%20Control-red)]()
[![النوع](https://img.shields.io/badge/النوع-Vertical%20Broken%20Access%20Control%20(تصعيد%20صلاحيات%20رأسي)-orange)]()
[![مستوى الخطورة](https://img.shields.io/badge/الخطورة-عالي%20(High)-critical)]()
[![المصدر](https://img.shields.io/badge/المصدر-PortSwigger%20Academy-blue)](https://portswigger.net/web-security/access-control)

> هذا الشرح جزء من سلسلة [معامل التحكم في الوصول (BAC)](./README.md) — أكاديمية PortSwigger.

---

## 1. بطاقة تعريف اللاب (Overview)

| الحقل | التفاصيل |
|---|---|
| **اسم اللاب** | Lab 02 — Unprotected Admin Functionality with Unpredictable URL (لوحة إدارة برابط عشوائي غير متوقع) |
| **تصنيف الثغرة** | Vertical Broken Access Control (تصعيد صلاحيات رأسي) |
| **معيار الضعف (CWE)** | CWE-284: Improper Access Control |
| **مستوى الخطورة** | عالي (High) |
| **الهدف من اللاب** | اكتشاف رابط الإدارة العشوائي، فتحه، وحذف المستخدم carlos. |

---

## 2. فكرة اللاب وسيناريو المشكلة (Concept & Vulnerability Mechanism)

أدرك المطور في هذا اللاب أن استخدام اسم مسار مشهور مثل `/admin` أو `/administrator-panel` يسهل تخمينه. فقرر استخدام مسار عشوائي طويل (مثل `/admin-xxxxxx`) ظناً منه أنه طالما الرابط مستحيل التخمين، فلا يمكن لأحد الوصول إليه.

**الخلل الفكري هنا:** قام المطور بوضع شرط في كود الجافاسكربت في الواجهة الأمامية: `if (isAdmin) { renderAdminLink('/admin-xxxxxx'); }`. ورغم أن زر الأدمن لن يظهر في المتصفح للمستخدم العادي، إلا أن ملف الجافاسكربت نفسه يتم تنزيله بالكامل إلى جهاز أي زائر للموقع! وبالتالي، مجرد فتح كود المصدر (Page Source) أو فحص أدوات المطور (DevTools) يكشف الرابط بالكامل، والسيرفر لا يفحص الصلاحيات عند زيارة هذا الرابط.

---

## 3. التحليل الجذري للسبب (Root Cause Analysis)

### 🔴 وجهة نظر المهاجم (Attacker Perspective):
المهاجم يفحص ملفات الـ JavaScript والـ Source Code المحملة في المتصفح، ويستخرج منها الروابط الإدارية المشروطة التي أخفاها المطور عن الواجهة.

### 🟢 وجهة نظر المدافع والمطور (Defender Perspective):
المطور خلط بين التحكم في عرض عناصر الواجهة (UI Rendering) والتحكم في صلاحيات الوصول للخدمة (Access Control). الأصول الثابتة التي تنزل للعميل يراها الجميع دون استثناء.

---

## 4. خطوات الاستغلال والحل بالتفصيل (Proof of Concept - PoC)

1. افتح الصفحة الرئيسية للموقع في المتصفح.
2. اضغط بزر الفأرة الأيمن واختر **View Page Source** (أو اضغط `Ctrl+U`).
3. ابحث في كود الـ HTML أو داخل كتل الـ `<script>` المضمنة عن أي إشارة لـ `admin`.
4. ستجد كود جافاسكربت يحتوي على نص شبيه بـ:
```javascript
if (user.isAdmin) {
   document.getElementById('admin-link').setAttribute('href', '/admin-1h9qf2');
}
```
5. انسخ المسار العشوائي (مثال: `/admin-1h9qf2`) وضعه في شريط المتصفح.
6. ستفتح لك لوحة الإدارة فوراً. اضغط على خيار حذف المستخدم `carlos` لإتمام الحل.

---

## 5. أدلة الترافيك (Evidence: Request / Response)

### نموذج طلب الاستغلال (Exploit Request):
```http
GET /admin-1h9qf2/delete?username=carlos HTTP/1.1
Host: <lab-id>.web-security-academy.net
```

### نموذج استجابة السيرفر (Server Response):
```http
HTTP/1.1 302 Found
Location: /admin-1h9qf2
...
```

---

## 6. الأثر الواقعي للثغرة (Impact)

الوصول لوظائف إدارية وتخريب حسابات المستخدمين بالاعتماد الزائف على سرية الروابط.

---

## 7. الحماية والإغلاق البرمجي (Remediation)

الاعتماد على عشوائية الرابط هو شكل خطير من أشكال (Security through Obscurity). يجب حماية المسار من جهة السيرفر بفحص الصلاحيات ورفض الطلب `403 Forbidden` لأي شخص لا يحمل دور الإدارة، مع تجريد كود الواجهة العامة من الروابط الإدارية.

---

## 8. المراجع الرسمية (References)

- [PortSwigger Web Security Academy — Access Control](https://portswigger.net/web-security/access-control)
- [OWASP Top 10 — A01:2021 Broken Access Control](https://owasp.org/Top10/A01_2021-Broken_Access_Control/)
- [CWE Database Details](https://cwe.mitre.org/)

---
*تم إعداد هذا التوثيق للأغراض التعليمية واختبار الاختراق الأخلاقي المصرح به.*
