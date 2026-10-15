# Server-Side Request Forgery Labs (EN)

[![Category](https://img.shields.io/badge/Category-SSRF-red)]()
[![Labs](https://img.shields.io/badge/PortSwigger%20Labs-7%20Labs-blue)]()
[![Cloud](https://img.shields.io/badge/Cloud%20Deep%20Dive-AWS%20IMDS-orange)]()
[![THM](https://img.shields.io/badge/TryHackMe-Walkthrough-orange)]()

Welcome to the English documentation for **Server-Side Request Forgery (SSRF - CWE-918)**. This directory covers the full progression of SSRF attacks: local loopback requests, internal network subnet scanning, blacklist/whitelist filter evasion, parser differentials, blind out-of-band detection, and chaining SSRF into Remote Code Execution via Shellshock.

---

## 📚 Core Documents

| Document | Description |
|---|---|
| [📖 OVERVIEW.md](./OVERVIEW.md) | Theoretical breakdown: In-band vs Blind SSRF, filter bypass techniques, and cloud risks. |
| [🧠 METHODOLOGY.md](./METHODOLOGY.md) | Pentester Mindset: Attack surface identification, URL fuzzing, and cloud metadata probing. |
| [🔍 ROOT-CAUSE.md](./ROOT-CAUSE.md) | Root cause analysis: Network perimeter trust, DNS rebinding, and URL parser differentials. |
| [🛡️ REMEDIATION.md](./REMEDIATION.md) | Secure implementation: IP address resolution validation, IMDSv2 enforcement, and egress rules. |
| [☁️ AWS-IMDS-Deep-Dive.md](./AWS-IMDS-Deep-Dive.md) | Industry-leading deep dive on AWS EC2 Metadata, IMDSv1 vs v2, and IAM theft case studies. |
| [🎯 THM-Server-Side-Request-Forgery.md](./THM-Server-Side-Request-Forgery.md) | Complete conceptual walkthrough for TryHackMe SSRF rooms. |

---

## 📋 Lab Writeup Index

| # | Lab Title | Attack Class | Severity | Link |
|---|---|---|---|---|
| 01 | Basic SSRF Against the Local Server | Localhost Loopback (`http://localhost/admin`) | High | [Lab-01](./Lab-01-Basic-SSRF-Local-Server.md) |
| 02 | Basic SSRF Against Another Back-End System | Internal Subnet Scan (`192.168.0.X:8080`) | High | [Lab-02](./Lab-02-Basic-SSRF-Backend-System.md) |
| 03 | SSRF with Blacklist-Based Input Filter | IP Obfuscation (`127.1`) & Double Encoding | High | [Lab-03](./Lab-03-Blacklist-Filter-Bypass.md) |
| 04 | SSRF with Filter Bypass via Open Redirection | Whitelist Bypass via Chained Open Redirect | High | [Lab-04](./Lab-04-Open-Redirect-Filter-Bypass.md) |
| 05 | Blind SSRF with Out-of-Band Detection | Blind SSRF via `Referer` Header to Collaborator | High | [Lab-05](./Lab-05-Blind-SSRF-OOB-Detection.md) |
| 06 | SSRF with Whitelist-Based Input Filter | URL Parser Differential (`@` and `#` Fragment) | Critical | [Lab-06](./Lab-06-Whitelist-Filter-Bypass.md) |
| 07 | Blind SSRF with Shellshock Exploitation | Blind SSRF Chained to RCE (CVE-2014-6271) | Critical | [Lab-07](./Lab-07-Blind-SSRF-Shellshock.md) |
