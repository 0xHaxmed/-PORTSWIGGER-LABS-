# PortSwigger Web Security Academy — Writeups

Professional, structured Proof-of-Concept (PoC) writeups for PortSwigger Web Security Academy labs, organized by vulnerability category. Each category is a self-contained folder with its own index and individual lab writeups.

> All content is for educational and authorized lab purposes only (PortSwigger Web Security Academy environment).

---

## 📂 Categories

| # | Category | Labs Covered | Folder |
|---|---|---|---|
| 02 | Broken Access Control | 13 labs | [02-BAC-Labs/](./02-BAC-Labs/) |

> More categories (SQL Injection, XSS, SSRF, Authentication, etc.) will be added here as separate folders, following the same structure.

---

## 📐 Repository Structure

```
PORTSWIGGER-LABS/
├── README.md                  ← you are here (category index)
├── 02-BAC-Labs/                ← Broken Access Control
│   ├── README.md              ← index of all labs in this category
│   ├── Lab-01-....md
│   ├── Lab-02-....md
│   └── ...
├── 03-SQLi-Labs/                ← (future) SQL Injection
│   ├── README.md
│   └── Lab-01-....md
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
