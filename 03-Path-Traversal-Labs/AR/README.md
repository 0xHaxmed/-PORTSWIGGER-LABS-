# معامل اجتياز المسارات وتضمين الملفات (Path Traversal) — الفهرس الكامل

[![التصنيف](https://img.shields.io/badge/CWE-CWE--22-red)]()
[![اللابات المكتملة](https://img.shields.io/badge/PortSwigger%20Labs-6%20Labs-blue)]()
[![غرف العملي](https://img.shields.io/badge/TryHackMe-Walkthrough-orange)]()

مرحباً بك في التوثيق العربي الكامل والشامل لثغرات **Path Traversal & LFI**. يغطي هذا القسم الطرق المنهجية لتجاوز الفلاتر وقراءة ملفات النظام الحساسة، وتحويل LFI إلى تنفيذ أوامر كامل (RCE).

---

## 📚 الملفات والوثائق الأساسية

| الوثيقة | الوصف والمحتوى |
|---|---|
| [📖 OVERVIEW.md](./OVERVIEW.md) | شرح نظري شامل: الفرق بين Path Traversal و LFI و RFI وأنواع الفلاتر. |
| [🧠 METHODOLOGY.md](./METHODOLOGY.md) | عقلية البنتستر: سلم التدرج في تجربة البيلودات وأماكن البحث في التطبيق. |
| [🔍 ROOT-CAUSE.md](./ROOT-CAUSE.md) | التحليل الجذري لأخطاء دمج المسارات والفلترة غير التكرارية. |
| [🛡️ REMEDIATION.md](./REMEDIATION.md) | دليل الحماية للمطورين: التحقق عبر الـ Canonical Path في جافا وبايثون. |
| [🎯 THM-File-Inclusion-Path-Traversal.md](./THM-File-Inclusion-Path-Traversal.md) | الدليل العربي الشامل لغرفة TryHackMe مع تقنيات LFI to RCE (اللوج والسيشن). |

---

## 📋 فهرس لابات PortSwigger المشروحة بالتفصيل

| # | عنوان اللاب | نوع الفلتر والالتفاف | مستوى الخطورة | الرابط المباشر |
|---|---|---|---|---|
| 01 | File Path Traversal, Simple Case | لا توجد حماية (`../../../etc/passwd`) | عالي (High) | [Lab-01](./Lab-01-File-Path-Traversal-Simple-Case.md) |
| 02 | Traversal Sequences Blocked with Absolute Path Bypass | تجاوز فلتر النسبي بالمسار المطلق (`/etc/passwd`) | عالي (High) | [Lab-02](./Lab-02-Traversal-Sequences-Blocked-with-Absolute-Path-Bypass.md) |
| 03 | Traversal Sequences Stripped Non-Recursively | تجاوز الفلتر الأحادي بالتداخل (`....//`) | عالي (High) | [Lab-03](./Lab-03-Traversal-Sequences-Stripped-Non-Recursively.md) |
| 04 | Traversal Sequences Stripped with Superfluous URL-Decode | التشفير المزدوج (`%252e%252e%252f`) | عالي (High) | [Lab-04](./Lab-04-Traversal-Sequences-Stripped-with-Superfluous-URL-Decode.md) |
| 05 | Validation of Start of Path | تجاوز شرط بداية المسار (`/var/www/images/...`) | عالي (High) | [Lab-05](./Lab-05-Validation-of-Start-of-Path.md) |
| 06 | Validation of File Extension with Null Byte Bypass | حقن البايت الفارغ لكسر الامتداد (`%00.jpg`) | عالي (High) | [Lab-06](./Lab-06-Validation-of-File-Extension-with-Null-Byte-Bypass.md) |
