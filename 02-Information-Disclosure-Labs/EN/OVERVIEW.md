# Information Disclosure — Comprehensive Overview

## 1. What is Information Disclosure?
Information disclosure (also known as information leakage) occurs when a website unintentionally reveals sensitive, confidential, or technical information to unauthorized users. This can include:
- Technical stack details (framework versions, database types, internal server paths).
- Secrets and credentials (API keys, database passwords, private encryption keys).
- Business-sensitive information (source code, customer financial records, internal employee usernames).

In the OWASP Top 10, Information Disclosure frequently maps to **A05:2021 — Security Misconfiguration** and **A01/A04**, often serving as the initial reconnaissance vector for severe multi-stage attacks.

## 2. Common Information Disclosure Vectors

| Vector | Description | Example Risk |
|---|---|---|
| **Verbose Error Messages** | Uncaught exceptions showing stack traces and library versions. | Reveals exact Apache Struts or Spring version for 1-day exploit lookup. |
| **Debug Interfaces** | Development dashboards (phpinfo, Django debug mode, Swagger) exposed in prod. | Exposes all environment variables, AWS keys, and session tokens. |
| **Backup Files** | Old file copies left behind by text editors (.bak, .old, ~). | Exposes raw backend source code without server-side execution. |
| **Exposed Git Repositories** | `.git` folders accessible via HTTP (`/.git/`). | Complete codebase extraction including previous git commit history. |
| **HTTP Diagnostic Methods** | HTTP TRACE or OPTIONS echoing internal headers. | Leaks custom proxy trust headers used for IP-restricted admin bypass. |

---

## 3. The Attacker's Perspective: The Stepping Stone Flaw
Information disclosure is rarely the end goal itself; it is the **catalyst**. A database password leaked in a backup file leads to database compromise; a leaked internal header leads to authentication bypass.
