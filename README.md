# 🛡️ WEB PENTEST LABS — Comprehensive Bilingual Reference

[![Labs](https://img.shields.io/badge/PortSwigger%20Labs-43%20Labs%20Covered-blue?style=for-the-badge&logo=portswigger)]()
[![Languages](https://img.shields.io/badge/Languages-Arabic%20%7C%20English-green?style=for-the-badge)]()
[![Standard](https://img.shields.io/badge/Standard-OWASP%20Top%2010%20%26%20CWE-red?style=for-the-badge)]()
[![Rooms](https://img.shields.io/badge/TryHackMe-Walkthroughs%20Included-orange?style=for-the-badge&logo=tryhackme)]()

> مستودع ومرجع أمني احترافي متكامل ثنائي اللغة (عربي / إنجليزي) يغطي معامل **أكاديمية PortSwigger** و **غرف TryHackMe**، مع التركيز على **فهم آلية وسيناريو كل ثغرة**، وأسبابها البرمجية المعمارية، وخطوات استغلالها بالتفصيل مع أدلة الترافيك، وكيفية إغلاقها وحمايتها.

---

## 🌐 Choose Language / اختر لغة التصفح

داخل كل مجلد ثغرة، ستجد مجلدين أساسيين:
- 🇪🇬 **`AR/`** — النسخة العربية: شروحات مفاهيمية عميقة، لغة عربية سلسة واحترافية، فهم دقيق لكواليس كل لاب وسيناريو حدوثه خطوة بخطوة.
- 🇬🇧 **`EN/`** — English Version: Full industry-grade writeups, Shields badges, request/response evidence, root cause analysis, and remediation.

---

## 📂 خريطة وفهرس الأقسام (01 إلى 06)

| # | فئة الثغرة / Category | عدد اللابات | 🇪🇬 النسخة العربية | 🇬🇧 English Version | 🎯 تطبيق TryHackMe | ⚡ HAXMED Intelligence |
|---|---|:---:|:---:|:---:|:---:|:---:|
| **01** | **Broken Access Control** | 13 | [📂 تصفح بالعربي](./01-BAC-Labs/AR/README.md) | [📂 Browse in EN](./01-BAC-Labs/EN/README.md) | [BAC Guide](./01-BAC-Labs/AR/THM-Broken-Access-Control.md) • [IDOR Guide](./01-BAC-Labs/AR/THM-IDOR.md) | [⚡ BAC Sheet](./01-BAC-Labs/BROKEN-ACCESS-CONTROL-INTELLIGENCE.md) |
| **02** | **Information Disclosure** | 5 | [📂 تصفح بالعربي](./02-Information-Disclosure-Labs/AR/README.md) | [📂 Browse in EN](./02-Information-Disclosure-Labs/EN/README.md) | [Recon Guide](./02-Information-Disclosure-Labs/AR/THM-Information-Disclosure.md) | [⚡ Info Sheet](./02-Information-Disclosure-Labs/INFORMATION-DISCLOSURE-INTELLIGENCE.md) |
| **03** | **Path Traversal & LFI** | 6 | [📂 تصفح بالعربي](./03-Path-Traversal-Labs/AR/README.md) | [📂 Browse in EN](./03-Path-Traversal-Labs/EN/README.md) | [File Inc Guide](./03-Path-Traversal-Labs/AR/THM-File-Inclusion.md) • [Adv LFI Guide](./03-Path-Traversal-Labs/AR/THM-File-Inclusion-Path-Traversal.md) | [⚡ Traversal Sheet](./03-Path-Traversal-Labs/PATH-TRAVERSAL-LFI-INTELLIGENCE.md) |
| **04** | **OS Command Injection** | 5 | [📂 تصفح بالعربي](./04-OS-Command-Injection-Labs/AR/README.md) | [📂 Browse in EN](./04-OS-Command-Injection-Labs/EN/README.md) | [THM Guide](./04-OS-Command-Injection-Labs/AR/THM-OS-Command-Injection.md) | [⚡ Command Sheet](./04-OS-Command-Injection-Labs/COMMAND-INJECTION-INTELLIGENCE.md) |
| **05** | **Server-Side Request Forgery** | 7 | [📂 تصفح بالعربي](./05-SSRF-Labs/AR/README.md) | [📂 Browse in EN](./05-SSRF-Labs/EN/README.md) | [THM Guide](./05-SSRF-Labs/AR/THM-Server-Side-Request-Forgery.md) | [⚡ SSRF Sheet](./05-SSRF-Labs/SSRF-INTELLIGENCE.md) |
| **06** | **File Upload Vulnerabilities** | 7 | [📂 تصفح بالعربي](./06-File-Upload-Labs/AR/README.md) | [📂 Browse in EN](./06-File-Upload-Labs/EN/README.md) | [THM Guide](./06-File-Upload-Labs/AR/THM-Upload-Vulnerabilities.md) | [⚡ Upload Sheet](./06-File-Upload-Labs/FILE-UPLOAD-INTELLIGENCE.md) |

---

## 🏛️ الهيكل المعماري الموحد لكل قسم (Directory Structure)

يحتوي كل قسم على نفس الهيكل المنتظم لسهولة التصفح والدراسة:

```text
Category-Folder/
├── README.md                          ← بوابة القسم مع محول اللغة وجدول اللابات
├── AR/                                ← النسخة العربية الشاملة
│   ├── README.md                      ← فهرس القسم ومحتوياته
│   ├── OVERVIEW.md                    ← نظرة عامة وشرح نظري معماري للثغرة
│   ├── METHODOLOGY.md                 ← منهجية الفحص وعقلية البنتستر (Mindset)
│   ├── ROOT-CAUSE.md                  ← أسباب الثغرة الجوهرية والخلل البرمجي
│   ├── REMEDIATION.md                 ← دليل الحماية للمطورين وأكواد الإغلاق
│   ├── THM-<Topic>.md                 ← شرح وتوجيه غرفة TryHackMe العملية
│   ├── Lab-01-...md                   ← شرح اللاب الأول (فكرة وسيناريو وحل كامل)
│   └── ...                            ← باقي لابات القسم
└── EN/                                ← The English Version
    ├── README.md                      ← English Category Index & Lab Matrix
    ├── OVERVIEW.md                    ← Full Vulnerability Theoretical Breakdown
    ├── METHODOLOGY.md                 ← Systematic Testing Workflows & Tools
    ├── ROOT-CAUSE.md                  ← Technical Root Cause & Flawed Logic
    ├── REMEDIATION.md                 ← Developer Hardening & Secure Code
    ├── THM-<Topic>.md                 ← Unified THM Practical Walkthrough
    ├── Lab-01-...md                   ← Lab Writeup with Badges & Raw Evidence
    └── ...                            ← Remaining Category Labs
```

---

## 📝 المعيار الموحد لشرح كل لاب (8-Section Standard)

تمت صياغة كل معمل في النسختين العربية والإنجليزية ليضمن **الفهم المفاهيمي العميق وليس مجرد حرق الحل**:

1. **بطاقة تعريف اللاب (Overview):** اسم اللاب، تصنيف الثغرة، رمز CWE، مستوى الصعوبة، والهدف المطلوب.
2. **فكرة اللاب وسيناريو المشكلة (Concept & Vulnerability Mechanism):** كيف يعمل التطبيق، وما هو التصميم البرمجي الخاطئ الذي أدى لوجود الثغرة خلف الكواليس.
3. **التحليل الجذري للسبب (Root Cause Analysis):** مقارنة تفصيلية بين:
   - 🔴 **منظور المهاجم (Attacker Perspective):** كيف يفكر المهاجم ويتلاعب بالمدخلات؟
   - 🟢 **منظور المدافع والمطور (Defender Perspective):** أين يكمن الخلل البرمجي في الكود؟
4. **خطوات الاستغلال والحل بالتفصيل (Proof of Concept - PoC):** خطوات عملية دقيقة وقابلة لإعادة الإنتاج باستخدام متصفح الويب و Burp Suite.
5. **أدلة الترافيك (Evidence: Request / Response):** نصوص الطلبات والاستجابات الأصلية من السيرفر.
6. **الأثر الواقعي للثغرة (Impact):** العواقب الفعلية لاستغلال الثغرة في بيئات الشركات والإنتاج.
7. **الحماية والإغلاق البرمجي (Remediation):** أمثلة برمجية وإعدادات خوادم لإغلاق الثغرة نهائياً.
8. **المراجع الرسمية (References):** روابط رسمية لأكاديمية PortSwigger و OWASP و CWE.

---

## 📚 مراجع وروابط مهمة

- [PortSwigger Web Security Academy](https://portswigger.net/web-security)
- [OWASP Top 10 Project](https://owasp.org/Top10/)
- [Common Weakness Enumeration (CWE)](https://cwe.mitre.org/)
- [TryHackMe Security Rooms](https://tryhackme.com/)

---
*تم إعداد وتطوير هذا المستودع ليكون دليلاً مرجعياً وتعليمياً متقدماً لمختبري اختراق تطبيقات الويب ومهندسي أمان التطبيقات.*
