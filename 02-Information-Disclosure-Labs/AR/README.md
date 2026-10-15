# معامل تسريب المعلومات (Information Disclosure) — الفهرس الكامل

[![التصنيف](https://img.shields.io/badge/OWASP-A05%3A2021-orange)]()
[![اللابات المكتملة](https://img.shields.io/badge/PortSwigger%20Labs-5%20Labs-blue)]()

مرحباً بك في التوثيق العربي الكامل لثغرات **Information Disclosure**. يغطي هذا القسم السيناريوهات العملية لتسريب أسرار البرمجيات وإصداراتها وملفات النسخ الاحتياطي ومستودعات Git واستغلالها لتنفيذ اختراقات متقدمة.

---

## 📚 الملفات والوثائق الأساسية

| الوثيقة | الوصف والمحتوى |
|---|---|
| [📖 OVERVIEW.md](./OVERVIEW.md) | شرح نظري شامل لأنواع التسريب، الأثر الأمني، وسلاسل الهجمات المركبة. |
| [🧠 METHODOLOGY.md](./METHODOLOGY.md) | عقلية البنتستر: استراتيجيات إثارة الأخطاء، صيد ملفات الباك آب، وفحص الـ Git. |
| [🔍 ROOT-CAUSE.md](./ROOT-CAUSE.md) | التحليل الجذري لأخطاء وضع التصحيح والتهيئة الخاطئة لخوادم الويب. |
| [🛡️ REMEDIATION.md](./REMEDIATION.md) | دليل الحماية للمطورين: قواعد Apache و Nginx ومعالجة الأخطاء البرمجية الموحدة. |
| [🎯 THM-Information-Disclosure.md](./THM-Information-Disclosure.md) | دليل عملي لمنهجيات جمع المعلومات واستخراج مستودعات Git. |

---

## 📋 فهرس لابات PortSwigger المشروحة بالتفصيل

| # | عنوان اللاب | نوع الثغرة | مستوى الخطورة | الرابط المباشر |
|---|---|---|---|---|
| 01 | Information Disclosure in Error Messages | تسريب إصدار الـ Framework في الـ Stack Trace | منخفض (Low) | [Lab-01](./Lab-01-Information-Disclosure-in-Error-Messages.md) |
| 02 | Information Disclosure on Debug Page | واجهة تصحيح مكشوفة (`phpinfo()`) وسرقة المفتاح | متوسط (Medium) | [Lab-02](./Lab-02-Information-Disclosure-on-Debug-Page.md) |
| 03 | Source Code Disclosure via Backup Files | كشف كود المصدر عبر ملف `.bak` وسرقة باسورد الداتابيز | عالي (High) | [Lab-03](./Lab-03-Source-Code-Disclosure-via-Backup-Files.md) |
| 04 | Authentication Bypass via Information Disclosure | تسريب هيدر الثقة الداخلي بطلب `TRACE` وتخطي الدخول | حرج (Critical) | [Lab-04](./Lab-04-Authentication-Bypass-via-Information-Disclosure.md) |
| 05 | Information Disclosure in Version Control History | استخراج كلمة مرور الأدمن من تاريخ مستودع `.git` | عالي (High) | [Lab-05](./Lab-05-Information-Disclosure-in-Version-Control-History.md) |
