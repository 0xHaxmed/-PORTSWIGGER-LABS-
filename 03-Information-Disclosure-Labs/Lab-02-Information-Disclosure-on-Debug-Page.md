# Lab 02 — Information Disclosure on Debug Page

[![Category](https://img.shields.io/badge/Category-Information%20Disclosure-blue)]()
[![Type](https://img.shields.io/badge/Type-Information%20Disclosure%20—%20Exposed%20Debug%20Interface-orange)]()
[![Severity](https://img.shields.io/badge/Severity-Medium-critical)]()
[![Source](https://img.shields.io/badge/Source-PortSwigger%20Web%20Security%20Academy-blue)](https://portswigger.net/web-security/information-disclosure)

> Part of the [Information Disclosure Labs](./README.md) series — PortSwigger Web Security Academy.

---

## 1. Overview

| Field | Detail |
|---|---|
| **Lab Name** | Information Disclosure on Debug Page |
| **Vulnerability Class** | Information Disclosure — Exposed Debug Interface |
| **Weakness (CWE)** | CWE-215: Insertion of Sensitive Information Into Debugging Code |
| **Severity** | Medium |
| **Objective** | Locate a debug page left enabled in production and extract the SECRET_KEY environment variable from it. |

## 2. Summary

A diagnostic page (PHP's built-in `phpinfo()` output) was left accessible in the production environment. The link to it was embedded in an HTML comment on the homepage rather than removed. This page dumps the entire PHP configuration and environment, including sensitive environment variables.

## 3. Proof of Concept (PoC)

**Step 1 — Inspect page source / HTML comments:**  
View the homepage's source (`Ctrl+U`) or use Burp's content tools to look for HTML comments. A comment reveals the hidden link:

```html
<!-- <a href="/cgi-bin/phpinfo.php">Debug</a> -->
```

**Step 2 — Request the debug page directly:**  
Navigate to the disclosed path:

```http
GET /cgi-bin/phpinfo.php HTTP/2
```

**Step 3 — Search the output:**  
Use `Ctrl+F` (browser) or Burp Repeater's response search to look for `SECRET_KEY`, which appears in both the **Environment** and **PHP Variables** sections of the dump.

**Step 4 — Extract and submit:**  
Copy the key's value and submit it to solve the lab.


## 4. Evidence (Request / Response)

```http
GET /cgi-bin/phpinfo.php HTTP/2
Host: <lab-id>.web-security-academy.net

HTTP/2 200 OK
...
Environment
  SECRET_KEY    0ldc6tw3hejjhcrnw5c393ih2dh1y1uc
PHP Variables
  $_SERVER['SECRET_KEY']    0ldc6tw3hejjhcrnw5c393ih2dh1y1uc
```

## 5. Root Cause Analysis

A diagnostic/debug tool (`phpinfo()`) intended strictly for local development was deployed to and left reachable in production, and a developer referenced it via an HTML comment instead of deleting the reference entirely. `phpinfo()` by design dumps the complete server environment — including any environment variable the process can see, regardless of sensitivity.

### Attacker vs. Defender Perspective

- **Attacker view:** Mine HTML comments and common diagnostic paths (phpinfo.php, /debug, /status) for forgotten developer tooling.
- **Defender view:** Strip debug endpoints and developer comments from every production build as a release-pipeline gate.

## 6. Impact

Directly exposes application secrets (encryption/signing keys, credentials) that can be used to forge session tokens, decrypt sensitive data, or pivot to further attacks, alongside detailed server fingerprinting information (PHP version, loaded modules, file paths) useful for follow-on exploitation.

## 7. Remediation

- Never deploy diagnostic/debug endpoints (`phpinfo.php`, debug consoles, admin diagnostic panels) to production environments.
- Remove developer comments and dead links from production markup as a standard part of the build/release pipeline.
- Store secrets (API keys, signing keys) in a dedicated secrets manager, not as raw environment variables accessible to any diagnostic tool.
- Periodically run automated crawls/content-discovery scans against production to catch accidentally-exposed debug endpoints.

## 8. References

- [PortSwigger — Information Disclosure](https://portswigger.net/web-security/information-disclosure)
- [OWASP Top 10 2021 — A01/A05: Broken Access Control / Security Misconfiguration](https://owasp.org/Top10/)
- [CWE-215: Insertion of Sensitive Information Into Debugging Code](https://cwe.mitre.org/data/definitions/215.html)

---

*Writeup produced for educational and authorized lab purposes only (PortSwigger Web Security Academy environment).*
