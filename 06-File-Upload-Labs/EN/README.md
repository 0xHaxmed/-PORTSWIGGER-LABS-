# File Upload Labs (EN)

[![Category](https://img.shields.io/badge/Category-File%20Upload-indigo)]()
[![Labs](https://img.shields.io/badge/PortSwigger%20Labs-7%20Labs-blue)]()
[![THM](https://img.shields.io/badge/TryHackMe-Walkthrough-orange)]()

Welcome to the English documentation for **File Upload Vulnerabilities (CWE-434)**. This module covers the full progression of file upload exploits: direct web shell execution, MIME-type spoofing, directory traversal in filenames, server config overrides (.htaccess), obfuscated extensions, image polyglot shells (exiftool), and race condition (TOCTOU) exploitation.

---

## 📚 Core Documents

| Document | Description |
|---|---|
| [📖 OVERVIEW.md](./OVERVIEW.md) | Comprehensive theory: Validation layers, bypass cheat sheet, and execution models. |
| [🧠 METHODOLOGY.md](./METHODOLOGY.md) | Pentester Mindset: Probing validation layers, extension testing, and execution paths. |
| [🔍 ROOT-CAUSE.md](./ROOT-CAUSE.md) | Root cause analysis: Executable web roots, client MIME trust, and race conditions. |
| [🛡️ REMEDIATION.md](./REMEDIATION.md) | Secure architecture: External storage, randomized filenames, and web server lockdowns. |
| [🎯 THM-Upload-Vulnerabilities.md](./THM-Upload-Vulnerabilities.md) | Complete conceptual walkthrough for TryHackMe Upload Vulnerabilities room. |

---

## 📋 Lab Writeup Index

| # | Lab Title | Defense & Bypass Class | Severity | Link |
|---|---|---|---|---|
| 01 | Remote Code Execution via Web Shell Upload | No Validation (Direct Web Shell) | Critical | [Lab-01](./Lab-01-Remote-Code-Execution-via-Web-Shell-Upload.md) |
| 02 | Web Shell Upload via Content-Type Restriction Bypass | MIME Type Spoofing (`image/jpeg`) | Critical | [Lab-02](./Lab-02-Web-Shell-Upload-via-Content-Type-Restriction-Bypass.md) |
| 03 | Web Shell Upload via Path Traversal | Filename Traversal (`..%2fexploit.php`) | Critical | [Lab-03](./Lab-03-Web-Shell-Upload-via-Path-Traversal.md) |
| 04 | Web Shell Upload via Extension Blacklist Bypass | Apache `.htaccess` MIME Override | Critical | [Lab-04](./Lab-04-Web-Shell-Upload-via-Extension-Blacklist-Bypass.md) |
| 05 | Web Shell Upload via Obfuscated File Extension | Null Byte Injection (`exploit.php%00.jpg`) | Critical | [Lab-05](./Lab-05-Web-Shell-Upload-via-Obfuscated-File-Extension.md) |
| 06 | Remote Code Execution via Polyglot Web Shell Upload | Valid Image Polyglot (`exiftool`) | Critical | [Lab-06](./Lab-06-Remote-Code-Execution-via-Polyglot-Web-Shell-Upload.md) |
| 07 | Web Shell Upload via Race Condition | TOCTOU Race Condition (Turbo Intruder) | Critical | [Lab-07](./Lab-07-Web-Shell-Upload-via-Race-Condition.md) |
