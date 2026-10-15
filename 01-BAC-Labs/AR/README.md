# معامل التحكم المكسور في الوصول (Broken Access Control) — الفهرس الكامل

[![التصنيف](https://img.shields.io/badge/OWASP-A01%3A2021-red)]()
[![اللابات المكتملة](https://img.shields.io/badge/PortSwigger%20Labs-13%20Labs-blue)]()
[![غرف العملي](https://img.shields.io/badge/TryHackMe-Walkthrough-orange)]()

مرحباً بك في التوثيق العربي الكامل والشامل لثغرات **Broken Access Control**. يحتوي هذا المجلد على شروحات تفصيلية لكافة معامل أكاديمية PortSwigger الـ 13، مع التركيز على فهم الفكرة وسيناريو المشكلة خلف الكواليس، وتحليل أسباب الثغرة وطرق استغلالها وإغلاقها.

---

## 📚 الملفات والوثائق الأساسية

| الوثيقة | الوصف والمحتوى |
|---|---|
| [📖 OVERVIEW.md](./OVERVIEW.md) | شرح نظري ومعماري شامل لنماذج التحكم بالوصول (DAC, MAC, RBAC, ABAC) وأنواع الثغرة. |
| [🧠 METHODOLOGY.md](./METHODOLOGY.md) | عقلية البنتستر: خطوات فحص الصلاحيات، استراتيجية الجلستين، وأدوات الاختبار (Autorize). |
| [🔍 ROOT-CAUSE.md](./ROOT-CAUSE.md) | التحليل الجذري للأخطاء البرمجية الخمسة التي تسبب انهيار التحكم بالوصول. |
| [🛡️ REMEDIATION.md](./REMEDIATION.md) | دليل الحماية للمطورين: نماذج كود برمجية محصنة وإعدادات السيرفر الآمنة. |
| [🎯 THM-Broken-Access-Control.md](./THM-Broken-Access-Control.md) | دليل ومنهجية حل غرفة Broken Access Control في TryHackMe. |
| [🎯 THM-IDOR.md](./THM-IDOR.md) | دليل ومنهجية حل غرفة Insecure Direct Object Reference (IDOR) في TryHackMe. |

---

## 📋 فهرس لابات PortSwigger المشروحة بالتفصيل

| # | عنوان اللاب | نوع الثغرة | مستوى الخطورة | الرابط المباشر |
|---|---|---|---|---|
| 01 | Unprotected Admin Functionality | تصعيد رأسي (لوحة مكشوفة بـ robots.txt) | عالي (High) | [Lab-01](./Lab-01-Unprotected-Admin-Functionality.md) |
| 02 | Unprotected Admin Functionality with Unpredictable URL | تصعيد رأسي (رابط مخفي مسرب بـ JS) | عالي (High) | [Lab-02](./Lab-02-Unprotected-Admin-Functionality-with-Unpredictable-URL.md) |
| 03 | User Role Controlled by Request Parameter | تصعيد رأسي (تزوير كوكي Admin) | حرج (Critical) | [Lab-03](./Lab-03-User-Role-Controlled-by-Request-Parameter.md) |
| 04 | User Role Can Be Modified in User Profile | تصعيد رأسي (ثغرة Mass Assignment) | حرج (Critical) | [Lab-04](./Lab-04-User-Role-Can-Be-Modified-in-User-Profile.md) |
| 05 | User ID Controlled by Request Parameter | تصعيد أفقي (IDOR مباشر بالأرقام) | عالي (High) | [Lab-05](./Lab-05-User-ID-Controlled-by-Request-Parameter.md) |
| 06 | User ID Controlled by Request Parameter with Unpredictable IDs | تصعيد أفقي (IDOR بـ GUID مسرب في مقال) | عالي (High) | [Lab-06](./Lab-06-User-ID-Controlled-by-Request-Parameter-with-Unpredictable-User-IDs.md) |
| 07 | User ID Controlled by Request Parameter with Data Leakage in Redirect | تصعيد أفقي (تسريب البيانات داخل رد 302) | عالي (High) | [Lab-07](./Lab-07-User-ID-Controlled-by-Request-Parameter-with-Data-Leakage-in-Redirect.md) |
| 08 | User ID Controlled by Request Parameter with Password Disclosure | تصعيد أفقي إلى رأسي (تسريب كلمة مرور الأدمن) | حرج (Critical) | [Lab-08](./Lab-08-User-ID-Controlled-by-Request-Parameter-with-Password-Disclosure.md) |
| 09 | Insecure Direct Object References (Predictable Static Files) | تصعيد أفقي (ملفات شات الدعم الفني 1.txt) | عالي (High) | [Lab-09](./Lab-09-Insecure-Direct-Object-References-(Predictable-Static-Files).md) |
| 10 | URL-based Access Control Can Be Circumvented | تصعيد رأسي (تجاوز حظر البروكسي بـ X-Original-URL) | حرج (Critical) | [Lab-10](./Lab-10-URL-based-Access-Control-Can-Be-Circumvented.md) |
| 11 | Method-based Access Control Can Be Circumvented | تصعيد رأسي (تجاوز فحص HTTP Method) | عالي (High) | [Lab-11](./Lab-11-Method-based-Access-Control-Can-Be-Circumvented.md) |
| 12 | Multi-step Process with No Access Control on One Step | تصعيد رأسي (القفز لخطوة التأكيد في الـ Workflow) | حرج (Critical) | [Lab-12](./Lab-12-Multi-step-Process-with-No-Access-Control-on-One-Step.md) |
| 13 | Referer-based Access Control | تصعيد رأسي (الثقة العمياء في هيدر Referer) | عالي (High) | [Lab-13](./Lab-13-Referer-based-Access-Control.md) |
