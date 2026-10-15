# 05 — Server-Side Request Forgery (SSRF) Labs

Welcome to the **Server-Side Request Forgery (SSRF)** vulnerability module.
مرحباً بك في القسم الخامس: **ثغرات تزوير الطلبات من جانب السيرفر**.

> SSRF allows an attacker to induce the server-side application to make HTTP requests to an arbitrary domain—exposing internal networks, loopback services, and cloud metadata (AWS IMDS).

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
| **AWS IMDS Deep Dive** | [☁️ AR/AWS-IMDS-Deep-Dive.md](./AR/AWS-IMDS-Deep-Dive.md) | [☁️ EN/AWS-IMDS-Deep-Dive.md](./EN/AWS-IMDS-Deep-Dive.md) |
| **TryHackMe Walkthrough** | [🎯 AR/THM-Server-Side-Request-Forgery.md](./AR/THM-Server-Side-Request-Forgery.md) | [🎯 EN/THM-Server-Side-Request-Forgery.md](./EN/THM-Server-Side-Request-Forgery.md) |

---

## 📋 Covered Labs (7 PortSwigger Labs)

| # | Lab Title | Arabic Version | English Version |
|---|---|:---:|:---:|
| 01 | Basic SSRF Against the Local Server | [🇪🇬 شرح بالعربي](./AR/Lab-01-Basic-SSRF-Local-Server.md) | [🇬🇧 Read in EN](./EN/Lab-01-Basic-SSRF-Local-Server.md) |
| 02 | Basic SSRF Against Another Back-End System | [🇪🇬 شرح بالعربي](./AR/Lab-02-Basic-SSRF-Backend-System.md) | [🇬🇧 Read in EN](./EN/Lab-02-Basic-SSRF-Backend-System.md) |
| 03 | SSRF with Blacklist-Based Input Filter | [🇪🇬 شرح بالعربي](./AR/Lab-03-Blacklist-Filter-Bypass.md) | [🇬🇧 Read in EN](./EN/Lab-03-Blacklist-Filter-Bypass.md) |
| 04 | SSRF with Filter Bypass via Open Redirection | [🇪🇬 شرح بالعربي](./AR/Lab-04-Open-Redirect-Filter-Bypass.md) | [🇬🇧 Read in EN](./EN/Lab-04-Open-Redirect-Filter-Bypass.md) |
| 05 | Blind SSRF with Out-of-Band Detection | [🇪🇬 شرح بالعربي](./AR/Lab-05-Blind-SSRF-OOB-Detection.md) | [🇬🇧 Read in EN](./EN/Lab-05-Blind-SSRF-OOB-Detection.md) |
| 06 | SSRF with Whitelist-Based Input Filter | [🇪🇬 شرح بالعربي](./AR/Lab-06-Whitelist-Filter-Bypass.md) | [🇬🇧 Read in EN](./EN/Lab-06-Whitelist-Filter-Bypass.md) |
| 07 | Blind SSRF with Shellshock Exploitation | [🇪🇬 شرح بالعربي](./AR/Lab-07-Blind-SSRF-Shellshock.md) | [🇬🇧 Read in EN](./EN/Lab-07-Blind-SSRF-Shellshock.md) |
