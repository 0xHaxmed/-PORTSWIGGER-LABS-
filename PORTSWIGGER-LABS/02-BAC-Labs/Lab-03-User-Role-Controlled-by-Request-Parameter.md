# Lab 03 — User Role Controlled by Request Parameter

[![Category](https://img.shields.io/badge/Category-Broken%20Access%20Control-red)]()
[![Type](https://img.shields.io/badge/Type-Vertical%20Broken%20Access%20Control-orange)]()
[![Severity](https://img.shields.io/badge/Severity-Critical-critical)]()
[![Source](https://img.shields.io/badge/Source-PortSwigger%20Web%20Security%20Academy-blue)](https://portswigger.net/web-security/access-control)

> Part of the [Broken Access Control Labs](../README.md) series — PortSwigger Web Security Academy.

---

## 1. Overview

| Field | Detail |
|---|---|
| **Lab Name** | User Role Controlled by Request Parameter |
| **Vulnerability Class** | Vertical Broken Access Control |
| **Weakness (CWE)** | CWE-565: Reliance on Cookies without Validation and Integrity Checking |
| **Severity** | Critical |
| **Objective** | Access `/admin` and delete user `carlos` by tampering with a client-controlled cookie. |

## 2. Summary

The application determines administrative privilege based on a plaintext, client-writable cookie (`Admin=true/false`) instead of deriving role information from server-side session state.

## 3. Proof of Concept (PoC)

**Step 1 — Authenticate:**  
Log in with standard credentials `wiener:peter`.

**Step 2 — Intercept traffic:**  
Using Burp Suite, intercept any authenticated request and observe the request cookies:

```
Cookie: Admin=false; session=<...>
```

**Step 3 — Tamper with the parameter:**  
Modify the cookie value from `Admin=false` to `Admin=true` using Burp Repeater or match-and-replace.

**Step 4 — Access privileged route:**  
Navigate to `/admin` with the modified cookie attached.

**Step 5 — Exploit:**  
Delete user `carlos` from the admin panel.


## 4. Evidence (Request / Response)

```http
GET /admin HTTP/2
Host: <lab-id>.web-security-academy.net
Cookie: Admin=true; session=<wiener-session>

HTTP/2 200 OK
...
<h1>Admin panel</h1>
```

## 5. Root Cause Analysis

Authorization state (`Admin` flag) is stored and trusted directly from client-supplied cookie data. The server never cross-checks this value against the authenticated user's actual role stored server-side.

### Attacker vs. Defender Perspective

- **Attacker view:** Enumerate or guess hidden/unprotected paths and request them directly, bypassing any client-side-only gating.
- **Defender view:** Enforce server-side, deny-by-default authorization on every sensitive route, independent of how it was discovered.

## 6. Impact

Trivial privilege escalation: any authenticated user, regardless of actual role, can self-promote to administrator by editing a single cookie value.

## 7. Remediation

- Never store authorization-critical flags (roles, privilege levels) in client-controlled cookies or any client-writable storage.
- Derive role/permission on every request from server-side session state (e.g., a session ID mapped to a role in a server-side store or a signed/encrypted token).
- If cookies must carry role claims, sign and encrypt them (e.g., JWT with server-verified signature) and validate the signature on every request.
- Apply RBAC middleware centrally so authorization cannot be bypassed by forging individual request attributes.

## 8. References

- [PortSwigger — Access Control Vulnerabilities](https://portswigger.net/web-security/access-control)
- [OWASP Top 10 2021 — A01: Broken Access Control](https://owasp.org/Top10/A01_2021-Broken_Access_Control/)
- [CWE-565: Reliance on Cookies without Validation](https://cwe.mitre.org/data/definitions/565.html)

---

*Writeup produced for educational and authorized lab purposes only (PortSwigger Web Security Academy environment).*
