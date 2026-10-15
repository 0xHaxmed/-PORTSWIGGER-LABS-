# معامل حقن أوامر نظام التشغيل (OS Command Injection) — الفهرس الكامل

[![التصنيف](https://img.shields.io/badge/CWE-CWE--78-red)]()
[![اللابات المكتملة](https://img.shields.io/badge/PortSwigger%20Labs-5%20Labs-blue)]()
[![غرف العملي](https://img.shields.io/badge/TryHackMe-Walkthrough-orange)]()

مرحباً بك في التوثيق العربي الكامل والشامل لثغرات **OS Command Injection**. يغطي هذا القسم السيناريوهات العملية لتنفيذ الأوامر، والتعامل مع الثغرات العمياء (Blind) عبر تقنيات التأخير الزمني وإعادة توجيه المخرجات وسحب البيانات عبر Burp Collaborator.

---

## 📚 الملفات والوثائق الأساسية

| الوثيقة | الوصف والمحتوى |
|---|---|
| [📖 OVERVIEW.md](./OVERVIEW.md) | شرح نظري شامل: فواصل الشل، قنوات الاستجابة، وسلاسل استغلال الأوامر. |
| [🧠 METHODOLOGY.md](./METHODOLOGY.md) | عقلية البنتستر: استكشاف نقاط الحقن، فحص الثغرات العمياء، وتحويل الأعمى لظاهر. |
| [🔍 ROOT-CAUSE.md](./ROOT-CAUSE.md) | التحليل الجذري لمفهوم تشغيل الشل (Spawning Shells) والدمج النصي للمدخلات. |
| [🛡️ REMEDIATION.md](./REMEDIATION.md) | دليل الحماية للمطورين: تنفيذ العمليات بدون شل في بايثون ونود والقوائم البيضاء. |
| [🎯 THM-OS-Command-Injection.md](./THM-OS-Command-Injection.md) | دليل حل المعمل العملي في غرفة Command Injection على TryHackMe. |

---

## 📋 فهرس لابات PortSwigger المشروحة بالتفصيل

| # | عنوان اللاب | أسلوب وتقنية الحقن | مستوى الخطورة | الرابط المباشر |
|---|---|---|---|---|
| 01 | OS Command Injection, Simple Case | حقن مباشر ظاهر في فحص المخزون (`1\|whoami`) | منخفض (Low) | [Lab-01](./Lab-01-Simple-Case.md) |
| 02 | Blind OS Command Injection with Time Delays | حقن أعمى بالتأخير الزمني (`email=x\|\|ping...`) | عالي (High) | [Lab-02](./Lab-02-Blind-Time-Delays.md) |
| 03 | Blind OS Command Injection with Output Redirection | حقن أعمى بإعادة التوجيه لملف (`whoami>out.txt`) | عالي (High) | [Lab-03](./Lab-03-Blind-Output-Redirection.md) |
| 04 | Blind OS Command Injection with Out-of-Band Interaction | حقن أعمى خارجي عبر استعلام DNS بـ nslookup | عالي (High) | [Lab-04](./Lab-04-Blind-OOB-Interaction.md) |
| 05 | Blind OS Command Injection with Out-of-Band Data Exfiltration | سحب البيانات خارجياً بدمج whoami في طلب DNS | حرج (Critical) | [Lab-05](./Lab-05-Blind-OOB-Data-Exfiltration.md) |
