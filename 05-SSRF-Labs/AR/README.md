# معامل تزوير الطلبات من جانب السيرفر (SSRF) — الفهرس الكامل

[![التصنيف](https://img.shields.io/badge/OWASP-A10%3A2021-red)]()
[![اللابات المكتملة](https://img.shields.io/badge/PortSwigger%20Labs-7%20Labs-blue)]()
[![المرجع السحابي](https://img.shields.io/badge/AWS%20IMDS-Deep%20Dive-orange)]()
[![غرف العملي](https://img.shields.io/badge/TryHackMe-Walkthrough-orange)]()

مرحباً بك في التوثيق العربي الكامل والشامل لثغرات **Server-Side Request Forgery (SSRF)**. يغطي هذا القسم السيناريوهات العملية لاستهداف الخوادم المحلية، وفحص الشبكات الداخلية، وتجاوز القوائم البيضاء والسوداء، وتتبع استعلامات الـ DNS العمياء، وسرقة مفاتيح السحاب في AWS، وحتى تنفيذ الأوامر عبر Shellshock.

---

## 📚 الملفات والوثائق الأساسية

| الوثيقة | الوصف والمحتوى |
|---|---|
| [📖 OVERVIEW.md](./OVERVIEW.md) | شرح نظري شامل: مفهوم الـ SSRF، أهدافه، وأشهر أساليب تخطي الحماية. |
| [🧠 METHODOLOGY.md](./METHODOLOGY.md) | عقلية البنتستر: استكشاف نقاط الضعف، فحص الـ Cloud Metadata، وأدوات المراقبة. |
| [🔍 ROOT-CAUSE.md](./ROOT-CAUSE.md) | التحليل الجذري لخداع الشبكات الداخلية والتناقض في تفكيك الروابط (Parser Discrepancies). |
| [🛡️ REMEDIATION.md](./REMEDIATION.md) | دليل الحماية للمطورين: التحقق الصارم من الـ IP وإلزام IMDSv2 في AWS. |
| [☁️ AWS-IMDS-Deep-Dive.md](./AWS-IMDS-Deep-Dive.md) | المرجع الشامل والعميق لخدمات الميتاداتا السحابية وسرقة اعتمادات IAM في AWS. |
| [🎯 THM-Server-Side-Request-Forgery.md](./THM-Server-Side-Request-Forgery.md) | الدليل العملي والشرح التفصيلي لغرفتي SSRF على TryHackMe. |

---

## 📋 فهرس لابات PortSwigger المشروحة بالتفصيل

| # | عنوان اللاب | سيناريو الهجوم | مستوى الخطورة | الرابط المباشر |
|---|---|---|---|---|
| 01 | Basic SSRF Against the Local Server | استهداف السيرفر المحلي (`http://localhost/admin`) | عالي (High) | [Lab-01](./Lab-01-Basic-SSRF-Local-Server.md) |
| 02 | Basic SSRF Against Another Back-End System | فحص الشبكة الداخلية (`192.168.0.X:8080`) | عالي (High) | [Lab-02](./Lab-02-Basic-SSRF-Backend-System.md) |
| 03 | SSRF with Blacklist-Based Input Filter | تجاوز القائمة السوداء بـ `127.1` والتشفير المزدوج | عالي (High) | [Lab-03](./Lab-03-Blacklist-Filter-Bypass.md) |
| 04 | SSRF with Filter Bypass via Open Redirection | تجاوز القيد عبر ثغرة Open Redirect محلية | عالي (High) | [Lab-04](./Lab-04-Open-Redirect-Filter-Bypass.md) |
| 05 | Blind SSRF with Out-of-Band Detection | فحص أعمى عبر هيدر `Referer` نحو Collaborator | عالي (High) | [Lab-05](./Lab-05-Blind-SSRF-OOB-Detection.md) |
| 06 | SSRF with Whitelist-Based Input Filter | كسر القائمة البيضاء بالتناقض (`@` و `#` المشفرة) | حرج (Critical) | [Lab-06](./Lab-06-Whitelist-Filter-Bypass.md) |
| 07 | Blind SSRF with Shellshock Exploitation | دمج SSRF الأعمى مع Shellshock لـ RCE عبر DNS | حرج (Critical) | [Lab-07](./Lab-07-Blind-SSRF-Shellshock.md) |
