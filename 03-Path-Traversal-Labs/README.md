# 03 — Path Traversal Labs

Welcome to the **Path Traversal & File Inclusion** vulnerability module.
مرحباً بك في القسم الثالث: **ثغرات اجتياز المسارات وتضمين الملفات**.

> Path traversal allows attackers to read arbitrary files from the server's filesystem—exposing credentials, application code, and system records.

---

## 🌐 Choose Your Language / اختر لغة التصفح

| Language / اللغة | Folder / المجلد | Overview / نظرة عامة |
|---|---|---|
| 🇪🇬 **العربية (Arabic)** | [📂 الانتقال للنسخة العربية (AR)](./AR/README.md) | شروحات كاملة ومفصلة لجميع اللابات بالعربية، مع التركيز على فهم الفكرة وسيناريو المشكلة والحل العملي خطوة بخطوة. |
| 🇬🇧 **English (الإنجليزية)** | [📂 Browse English Version (EN)](./EN/README.md) | Complete industry-grade PoC writeups with Shields badges, root cause analysis, and remediation guides. |

---

## 📑 Core Documentation Index

| Feature / الميزة | 🇪🇬 Arabic (AR) | 🇬🇧 English (EN) |
|---|---|---|
| **Overview (نظرة عامة)** | [📖 AR/OVERVIEW.md](./AR/OVERVIEW.md) | [📖 EN/OVERVIEW.md](./EN/OVERVIEW.md) |
| **Methodology (منهجية الفحص)** | [🧠 AR/METHODOLOGY.md](./AR/METHODOLOGY.md) | [🧠 EN/METHODOLOGY.md](./EN/METHODOLOGY.md) |
| **Root Cause (أسباب الثغرة)** | [🔍 AR/ROOT-CAUSE.md](./AR/ROOT-CAUSE.md) | [🔍 EN/ROOT-CAUSE.md](./EN/ROOT-CAUSE.md) |
| **Remediation (طرق الحماية)** | [🛡️ AR/REMEDIATION.md](./AR/REMEDIATION.md) | [🛡️ EN/REMEDIATION.md](./EN/REMEDIATION.md) |
| **THM: File Inclusion** | [🎯 AR/THM-File-Inclusion.md](./AR/THM-File-Inclusion.md) | [🎯 EN/THM-File-Inclusion.md](./EN/THM-File-Inclusion.md) |
| **THM: Advanced LFI & Traversal** | [🎯 AR/THM-File-Inclusion-Path-Traversal.md](./AR/THM-File-Inclusion-Path-Traversal.md) | [🎯 EN/THM-File-Inclusion-Path-Traversal.md](./EN/THM-File-Inclusion-Path-Traversal.md) |
| **HAXMED Cheat Sheet & Intelligence** | [⚡ Cheat Sheet / Reference](./PATH-TRAVERSAL-LFI-INTELLIGENCE.md) | [⚡ Cheat Sheet / Reference](./PATH-TRAVERSAL-LFI-INTELLIGENCE.md) |

---

## 📋 Covered Labs (6 PortSwigger Labs)

| # | Lab Title | Arabic Version | English Version |
|---|---|:---:|:---:|
| 01 | File Path Traversal, Simple Case | [🇪🇬 شرح بالعربي](./AR/Lab-01-File-Path-Traversal-Simple-Case.md) | [🇬🇧 Read in EN](./EN/Lab-01-File-Path-Traversal-Simple-Case.md) |
| 02 | Traversal Sequences Blocked with Absolute Path Bypass | [🇪🇬 شرح بالعربي](./AR/Lab-02-Traversal-Sequences-Blocked-with-Absolute-Path-Bypass.md) | [🇬🇧 Read in EN](./EN/Lab-02-Traversal-Sequences-Blocked-with-Absolute-Path-Bypass.md) |
| 03 | Traversal Sequences Stripped Non-Recursively | [🇪🇬 شرح بالعربي](./AR/Lab-03-Traversal-Sequences-Stripped-Non-Recursively.md) | [🇬🇧 Read in EN](./EN/Lab-03-Traversal-Sequences-Stripped-Non-Recursively.md) |
| 04 | Traversal Sequences Stripped with Superfluous URL-Decode | [🇪🇬 شرح بالعربي](./AR/Lab-04-Traversal-Sequences-Stripped-with-Superfluous-URL-Decode.md) | [🇬🇧 Read in EN](./EN/Lab-04-Traversal-Sequences-Stripped-with-Superfluous-URL-Decode.md) |
| 05 | Validation of Start of Path | [🇪🇬 شرح بالعربي](./AR/Lab-05-Validation-of-Start-of-Path.md) | [🇬🇧 Read in EN](./EN/Lab-05-Validation-of-Start-of-Path.md) |
| 06 | Validation of File Extension with Null Byte Bypass | [🇪🇬 شرح بالعربي](./AR/Lab-06-Validation-of-File-Extension-with-Null-Byte-Bypass.md) | [🇬🇧 Read in EN](./EN/Lab-06-Validation-of-File-Extension-with-Null-Byte-Bypass.md) |
