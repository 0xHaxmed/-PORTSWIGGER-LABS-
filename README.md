# PortSwigger Web Security Academy — Writeups

Professional, structured Proof-of-Concept (PoC) writeups for PortSwigger Web Security Academy labs, organized by vulnerability category. Each category is a self-contained folder with its own index and individual lab writeups.

> All content is for educational and authorized lab purposes only (PortSwigger Web Security Academy environment).

---

## 📂 Categories

| # | Category | Labs Covered | Folder |
|---|---|---|---|
| 02 | Broken Access Control | 13 labs | [02-BAC-Labs/](./02-BAC-Labs/) |
| 03 | Information Disclosure | 5 labs | [03-Information-Disclosure-Labs/](./03-Information-Disclosure-Labs/) |
| 04 | Path Traversal | 6 labs | [04-Path-Traversal-Labs/](./04-Path-Traversal-Labs/) |
| 05 | OS Command Injection | 5 labs (+ TryHackMe practical) | [05-OS-Command-Injection-Labs/](./05-OS-Command-Injection-Labs/) |
| 06 | Server-Side Request Forgery (SSRF) | 7 labs | [06-SSRF-Labs/](./06-SSRF-Labs/) |

> More categories (SQL Injection, XSS, Authentication, etc.) will be added here as separate folders, following the same structure.

---

## 📐 Repository Structure

```
PORTSWIGGER-LABS/
├── README.md                              ← you are here (category index)
├── 02-BAC-Labs/                            ← Broken Access Control
│   ├── README.md
│   └── Lab-01-....md ...
├── 03-Information-Disclosure-Labs/         ← Information Disclosure
│   ├── README.md
│   └── Lab-01-....md ...
├── 04-Path-Traversal-Labs/                 ← Path Traversal
│   ├── README.md
│   └── Lab-01-....md ...
├── 05-OS-Command-Injection-Labs/           ← OS Command Injection
│   ├── README.md
│   ├── Lab-01-Simple-Case.md
│   ├── Lab-02-Blind-Time-Delays.md
│   ├── Lab-03-Blind-Output-Redirection.md
│   ├── Lab-04-Blind-OOB-Interaction.md
│   ├── Lab-05-Blind-OOB-Data-Exfiltration.md
│   ├── THM-Command-Injection-Practical.md
│   └── METHODOLOGY.md
├── 06-SSRF-Labs/                           ← Server-Side Request Forgery
│   ├── README.md
│   ├── Lab-01-Basic-SSRF-Local-Server.md
│   ├── Lab-02-Basic-SSRF-Backend-System.md
│   ├── Lab-03-Blacklist-Filter-Bypass.md
│   ├── Lab-04-Open-Redirect-Filter-Bypass.md
│   ├── Lab-05-Blind-SSRF-OOB-Detection.md
│   ├── Lab-06-Whitelist-Filter-Bypass.md
│   ├── Lab-07-Blind-SSRF-Shellshock.md
│   └── METHODOLOGY.md
└── ...
```

## ✍️ Writeup Format

Every individual lab writeup follows the same professional structure for consistency:

1. **Overview** — vulnerability class, CWE, severity, objective
2. **Summary** — plain-language explanation of the flaw
3. **Proof of Concept (PoC)** — numbered, reproducible steps
4. **Evidence** — request/response samples
5. **Root Cause Analysis** — technical explanation + Attacker vs. Defender perspective
6. **Impact** — real-world consequences
7. **Remediation** — concrete fixes for developers/defenders
8. **References** — CWE, OWASP, and official PortSwigger links

## 📚 References

- [PortSwigger Web Security Academy](https://portswigger.net/web-security)
- [OWASP Top 10](https://owasp.org/Top10/)
- [CWE — Common Weakness Enumeration](https://cwe.mitre.org/)
