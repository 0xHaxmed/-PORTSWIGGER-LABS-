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
| 06 | File Upload | 7 labs + TryHackMe walkthrough | [06-File-Upload-Labs/](./06-File-Upload-Labs/) |

> More categories (SQL Injection, XSS, SSRF, Authentication, OS Command Injection, etc.) will be added here as separate folders, following the same structure.

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
├── 06-File-Upload-Labs/                    ← File Upload
│   ├── README.md
│   ├── METHODOLOGY.md
│   ├── TOPIC-OVERVIEW.md
│   ├── THM-Upload-Vulnerabilities-Walkthrough.md
│   └── Lab-01-....md ...
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

Each category folder also includes a **METHODOLOGY.md** (pentester mindset for that vulnerability class) linked at the top of its README.

## 📚 References

- [PortSwigger Web Security Academy](https://portswigger.net/web-security)
- [OWASP Top 10](https://owasp.org/Top10/)
- [CWE — Common Weakness Enumeration](https://cwe.mitre.org/)
