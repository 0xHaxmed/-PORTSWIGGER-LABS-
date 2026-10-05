# Lab 03 — Source Code Disclosure via Backup Files

[![Category](https://img.shields.io/badge/Category-Information%20Disclosure-blue)]()
[![Type](https://img.shields.io/badge/Type-Information%20Disclosure%20—%20Backup%20File%20Exposure-orange)]()
[![Severity](https://img.shields.io/badge/Severity-High-critical)]()
[![Source](https://img.shields.io/badge/Source-PortSwigger%20Web%20Security%20Academy-blue)](https://portswigger.net/web-security/information-disclosure)

> Part of the [Information Disclosure Labs](./README.md) series — PortSwigger Web Security Academy.

---

## 1. Overview

| Field | Detail |
|---|---|
| **Lab Name** | Source Code Disclosure via Backup Files |
| **Vulnerability Class** | Information Disclosure — Backup File Exposure |
| **Weakness (CWE)** | CWE-530: Exposure of Backup File to an Unauthorized Control Sphere |
| **Severity** | High |
| **Objective** | Locate a leaked source-code backup file in a hidden directory and extract a hard-coded database password from it. |

## 2. Summary

A directory disclosed via `robots.txt` contains a `.bak` backup copy of a Java source file. Because the server does not recognize `.bak` as an executable/processable extension, it serves the file as raw static text instead of compiling/executing it — exposing the full source code, including a hard-coded database credential.

## 3. Proof of Concept (PoC)

**Step 1 — Check robots.txt:**  
Request `/robots.txt` and note the disallowed path:

```
User-agent: *
Disallow: /backup
```

**Step 2 — Browse the directory:**  
Navigate to `/backup` — directory listing is enabled, revealing `ProductTemplate.java.bak`.

**Step 3 — Request the backup file:**  
`GET /backup/ProductTemplate.java.bak` returns the raw Java source instead of being executed.

**Step 4 — Extract the hard-coded secret:**  
Inspect the `ConnectionBuilder.from(...)` call in the source — the Postgres database password is hard-coded as a literal string argument.


## 4. Evidence (Request / Response)

```http
GET /backup/ProductTemplate.java.bak HTTP/2
Host: <lab-id>.web-security-academy.net

HTTP/2 200 OK
...
ConnectionBuilder connectionBuilder = ConnectionBuilder.from(
    "org.postgresql.Driver", "postgresql", "localhost", 5432,
    "postgres", "postgres",
    "9t6oup5jqmtjadn9srbvmgmuhzu9cf1m"   // <-- hard-coded password
).withAutoCommit();
```

## 5. Root Cause Analysis

Two compounding misconfigurations: (1) `robots.txt` was used to "hide" a sensitive directory instead of actually restricting access to it, turning it into a reconnaissance map; (2) the web server has directory listing enabled and will serve any file extension it doesn't explicitly map to a script handler (like `.bak`) as plain static text, exposing source code verbatim — including hard-coded credentials.

### Attacker vs. Defender Perspective

- **Attacker view:** Check robots.txt and try common backup extensions (.bak, .old, ~) against any discovered source file path.
- **Defender view:** Keep backups out of the web root entirely and disable directory listing on production servers.

## 6. Impact

Full source code disclosure dramatically accelerates an attacker's ability to find further vulnerabilities (the leaked class here also reveals string-concatenated SQL — a SQL Injection indicator — and uses `Serializable`, a potential Insecure Deserialization vector). The hard-coded database password alone can allow direct, unauthenticated access to the backend datastore.

## 7. Remediation

- Never store backup, temporary, or editor-generated files (`.bak`, `.old`, `~`) inside the web-accessible document root.
- Disable directory listing (`Options -Indexes` in Apache, `autoindex off` in Nginx) on all production web servers.
- Never hard-code credentials, API keys, or secrets in source code; use environment variables backed by a secrets manager.
- Treat any path listed in `robots.txt` as a prime target for manual review, not as a reliably hidden location.

## 8. References

- [PortSwigger — Information Disclosure](https://portswigger.net/web-security/information-disclosure)
- [OWASP Top 10 2021 — A01/A05: Broken Access Control / Security Misconfiguration](https://owasp.org/Top10/)
- [CWE-530: Exposure of Backup File to an Unauthorized Control Sphere](https://cwe.mitre.org/data/definitions/530.html)

---

*Writeup produced for educational and authorized lab purposes only (PortSwigger Web Security Academy environment).*
