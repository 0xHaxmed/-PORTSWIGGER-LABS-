# معامل ثغرات رفع الملفات (File Upload) — الفهرس الكامل

[![التصنيف](https://img.shields.io/badge/CWE-CWE--434-red)]()
[![اللابات المكتملة](https://img.shields.io/badge/PortSwigger%20Labs-7%20Labs-blue)]()
[![غرف العملي](https://img.shields.io/badge/TryHackMe-Walkthrough-orange)]()

مرحباً بك في التوثيق العربي الكامل والشامل لثغرات **File Upload**. يغطي هذا القسم السيناريوهات المتقدمة لرفع وتشغيل الـ Web Shells، وتخطي فلاتر الـ MIME، والتلاعب بامتدادات أباتشي `.htaccess`، وحقن الصور الهجينة (Polyglot)، واستغلال ثغرات السباق الزمني (Race Conditions).

---

## 📚 الملفات والوثائق الأساسية

| الوثيقة | الوصف والمحتوى |
|---|---|
| [📖 OVERVIEW.md](./OVERVIEW.md) | شرح نظري شامل: طبقات فحص الملفات، جدول أساليب الالتفاف، ونماذج التنفيذ. |
| [🧠 METHODOLOGY.md](./METHODOLOGY.md) | عقلية البنتستر: خطوات اختبار الرفع، سلم تجربة الامتدادات، واستخراج الـ RCE. |
| [🔍 ROOT-CAUSE.md](./ROOT-CAUSE.md) | التحليل الجذري لمشاكل مجلدات الويب التنفيذية والسباق الزمني (TOCTOU). |
| [🛡️ REMEDIATION.md](./REMEDIATION.md) | دليل الحماية للمطورين: التخزين خارج الـ Web Root، تعطيل محرك PHP، وإعادة رسم الصور. |
| [🎯 THM-Upload-Vulnerabilities.md](./THM-Upload-Vulnerabilities.md) | الدليل الكامل والشامل لغرفة Upload Vulnerabilities على منصة TryHackMe. |

---

## 📋 فهرس لابات PortSwigger المشروحة بالتفصيل

| # | عنوان اللاب | سيناريو الالتفاف | مستوى الخطورة | الرابط المباشر |
|---|---|---|---|---|
| 01 | Remote Code Execution via Web Shell Upload | لا توجد حماية (رفع ويب شل مباشر) | حرج (Critical) | [Lab-01](./Lab-01-Remote-Code-Execution-via-Web-Shell-Upload.md) |
| 02 | Web Shell Upload via Content-Type Restriction Bypass | تزوير نوع المحتوى (`Content-Type: image/jpeg`) | حرج (Critical) | [Lab-02](./Lab-02-Web-Shell-Upload-via-Content-Type-Restriction-Bypass.md) |
| 03 | Web Shell Upload via Path Traversal | دمج Path Traversal لرفع الشل خارج المجلد | حرج (Critical) | [Lab-03](./Lab-03-Web-Shell-Upload-via-Path-Traversal.md) |
| 04 | Web Shell Upload via Extension Blacklist Bypass | رفع ملف `.htaccess` لتعريف امتداد تنفيذي جديد | حرج (Critical) | [Lab-04](./Lab-04-Web-Shell-Upload-via-Extension-Blacklist-Bypass.md) |
| 05 | Web Shell Upload via Obfuscated File Extension | حقن البايت الفارغ لاقتطاع الامتداد المزيف | حرج (Critical) | [Lab-05](./Lab-05-Web-Shell-Upload-via-Obfuscated-File-Extension.md) |
| 06 | Remote Code Execution via Polyglot Web Shell Upload | حقن الشل في صورة سليمة 100% بـ exiftool | حرج (Critical) | [Lab-06](./Lab-06-Remote-Code-Execution-via-Polyglot-Web-Shell-Upload.md) |
| 07 | Web Shell Upload via Race Condition | استغلال السباق الزمني للتشغيل قبل الحذف | حرج (Critical) | [Lab-07](./Lab-07-Web-Shell-Upload-via-Race-Condition.md) |
