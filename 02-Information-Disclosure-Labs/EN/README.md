# Information Disclosure Labs (EN)

[![Category](https://img.shields.io/badge/Category-Information%20Disclosure-blue)]()
[![Labs](https://img.shields.io/badge/PortSwigger%20Labs-5%20Labs-blue)]()

Welcome to the English documentation for **Information Disclosure (OWASP A05)**. This module covers the full spectrum of unintended data leaks: verbose error messages, exposed debug pages, backup source code files, reverse proxy trust header leaks via TRACE, and version control (.git) exposure.

---

## 📚 Core Documents

| Document | Description |
|---|---|
| [📖 OVERVIEW.md](./OVERVIEW.md) | Theoretical overview of information leakage, classification, and attack chains. |
| [🧠 METHODOLOGY.md](./METHODOLOGY.md) | Pentester mindset: Where leaks hide, how to trigger exceptions, and severity assessment. |
| [🔍 ROOT-CAUSE.md](./ROOT-CAUSE.md) | Root cause analysis: Debug flags, unhandled exceptions, and deployment packaging hygiene. |
| [🛡️ REMEDIATION.md](./REMEDIATION.md) | Hardening guides: Web server rules (Apache/Nginx), generic error pages, and CI/CD sanitation. |
| [🎯 THM-Information-Disclosure.md](./THM-Information-Disclosure.md) | Reconnaissance and web enumeration study guide. |

---

## 📋 Lab Writeups Index

| # | Lab Title | Vulnerability Type | Severity | Link |
|---|---|---|---|---|
| 01 | Information Disclosure in Error Messages | Stack Trace / Framework Version Leak | Low | [Lab-01](./Lab-01-Information-Disclosure-in-Error-Messages.md) |
| 02 | Information Disclosure on Debug Page | Exposed Diagnostic Interface (phpinfo) | Medium | [Lab-02](./Lab-02-Information-Disclosure-on-Debug-Page.md) |
| 03 | Source Code Disclosure via Backup Files | Exposed Source Code Backup (.bak) | High | [Lab-03](./Lab-03-Source-Code-Disclosure-via-Backup-Files.md) |
| 04 | Authentication Bypass via Information Disclosure | HTTP TRACE Trust Header Leak | Critical | [Lab-04](./Lab-04-Authentication-Bypass-via-Information-Disclosure.md) |
| 05 | Information Disclosure in Version Control History | Exposed .git Repository History | High | [Lab-05](./Lab-05-Information-Disclosure-in-Version-Control-History.md) |
