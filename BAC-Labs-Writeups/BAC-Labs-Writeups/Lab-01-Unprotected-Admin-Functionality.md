# Lab 01 — Unprotected Admin Functionality

[![Category](https://img.shields.io/badge/Category-Broken%20Access%20Control-red)]()
[![Type](https://img.shields.io/badge/Type-Vertical%20Broken%20Access%20Control-orange)]()
[![Severity](https://img.shields.io/badge/Severity-High-critical)]()
[![Source](https://img.shields.io/badge/Source-PortSwigger%20Web%20Security%20Academy-blue)](https://portswigger.net/web-security/access-control)

> Part of the [Broken Access Control Labs](../README.md) series — PortSwigger Web Security Academy.

---

## 1. Overview

| Field | Detail |
|---|---|
| **Lab Name** | Unprotected Admin Functionality |
| **Vulnerability Class** | Vertical Broken Access Control |
| **Weakness (CWE)** | CWE-284: Improper Access Control |
| **Severity** | High |
| **Objective** | Access the administration panel and delete the user `carlos`. |

## 2. Summary

The application exposes an administrative panel at a predictable path without any session-based privilege check. The only barrier is a `Disallow` directive in `robots.txt`, which is a client-side hint, not a security control.

## 3. Proof of Concept (PoC)

**Step 1 — Reconnaissance:**  
Request `GET /robots.txt` on the lab root domain.

**Step 2 — Identify the hidden path:**  
Observe the disallowed entry:

```
User-agent: *
Disallow: /administrator-panel
```

**Step 3 — Direct access:**  
Navigate directly to `/administrator-panel`. No authentication or authorization prompt is enforced.

**Step 4 — Exploit:**  
Click **Delete** next to user `carlos` to confirm full administrative control over the panel.


## 4. Evidence (Request / Response)

```http
GET /administrator-panel HTTP/2
Host: <lab-id>.web-security-academy.net
Cookie: session=<low-priv-or-no-session>

HTTP/2 200 OK
...
<h1>Admin panel</h1>
<table>...carlos...delete...</table>
```

## 5. Root Cause Analysis

The server relies on `robots.txt` — a convention respected only by well-behaved crawlers — to keep the admin panel undiscoverable. No server-side authorization check (role/session validation) is performed when the `/administrator-panel` route is hit directly.

### Attacker vs. Defender Perspective

- **Attacker view:** Enumerate or guess hidden/unprotected paths and request them directly, bypassing any client-side-only gating.
- **Defender view:** Enforce server-side, deny-by-default authorization on every sensitive route, independent of how it was discovered.

## 6. Impact

Any unauthenticated or low-privileged user who discovers the path (via `robots.txt`, directory brute-forcing, or search engine caches) gains full administrative control, including destructive actions such as user deletion.

## 7. Remediation

- Never rely on `robots.txt`, obscure paths, or client-side hiding as an access control mechanism (security by obscurity).
- Enforce server-side role/permission checks (RBAC) on every administrative route, independent of how the route was reached.
- Apply the principle of *deny by default*: unauthenticated or non-admin sessions should receive `401/403` before any admin logic executes.
- Remove sensitive paths from `robots.txt`; if discoverability must be limited, use authentication at the infrastructure layer (e.g., reverse proxy ACLs) as a secondary control, not the primary one.

## 8. References

- [PortSwigger — Access Control Vulnerabilities](https://portswigger.net/web-security/access-control)
- [OWASP Top 10 2021 — A01: Broken Access Control](https://owasp.org/Top10/A01_2021-Broken_Access_Control/)
- [CWE-284: Improper Access Control](https://cwe.mitre.org/data/definitions/284.html)

---

*Writeup produced for educational and authorized lab purposes only (PortSwigger Web Security Academy environment).*
