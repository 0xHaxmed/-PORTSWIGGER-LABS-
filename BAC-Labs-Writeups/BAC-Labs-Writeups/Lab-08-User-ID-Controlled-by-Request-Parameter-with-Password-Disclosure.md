# Lab 08 — User ID Controlled by Request Parameter with Password Disclosure

[![Category](https://img.shields.io/badge/Category-Broken%20Access%20Control-red)]()
[![Type](https://img.shields.io/badge/Type-Horizontal-to-Vertical%20Privilege%20Escalation-orange)]()
[![Severity](https://img.shields.io/badge/Severity-Critical-critical)]()
[![Source](https://img.shields.io/badge/Source-PortSwigger%20Web%20Security%20Academy-blue)](https://portswigger.net/web-security/access-control)

> Part of the [Broken Access Control Labs](../README.md) series — PortSwigger Web Security Academy.

---

## 1. Overview

| Field | Detail |
|---|---|
| **Lab Name** | User ID Controlled by Request Parameter with Password Disclosure |
| **Vulnerability Class** | Horizontal-to-Vertical Privilege Escalation (Password Disclosure) |
| **Weakness (CWE)** | CWE-522: Insufficiently Protected Credentials |
| **Severity** | Critical |
| **Objective** | Retrieve the administrator's password and delete user `carlos`. |

## 2. Summary

The same unguarded `id` parameter from Lab 05 now exposes a far more critical field: the target account's plaintext password, pre-filled into a password input for user convenience — turning a horizontal IDOR into full vertical privilege escalation.

## 3. Proof of Concept (PoC)

**Step 1 — Authenticate:**  
Log in with credentials `wiener:peter`.

**Step 2 — Exploit IDOR:**  
Navigate to `/my-account?id=administrator`, reusing the IDOR pattern from previous labs.

**Step 3 — Inspect page source:**  
View the page's HTML source and locate the password `<input>` field, which is pre-populated:

```html
<input type="password" value="admin-password-here">
```

**Step 4 — Escalate:**  
Log out, then log back in using the recovered `administrator` credentials.

**Step 5 — Exploit:**  
Access `/admin` and delete user `carlos` as a fully privileged administrator.


## 4. Evidence (Request / Response)

```http
GET /my-account?id=administrator HTTP/2
Host: <lab-id>.web-security-academy.net
Cookie: session=<wiener-session>

HTTP/2 200 OK
...
<input type="password" value="S3cr3tAdm1nPW!">
```

## 5. Root Cause Analysis

Two compounding failures: (1) the persistent lack of ownership verification on `/my-account`, and (2) a UX decision to pre-fill the password field with the stored plaintext value instead of leaving it blank — exposing credentials that should never be rendered back to the client at all.

### Attacker vs. Defender Perspective

- **Attacker view:** Identify an object reference (ID/GUID/filename) in a request and systematically substitute values belonging to other users to access data outside the attacker's own scope.
- **Defender view:** Never trust a client-supplied identifier; always verify server-side that the authenticated session owns (or is authorized for) the specific object being requested.

## 6. Impact

Escalates a mid-severity information disclosure into full account takeover of the highest-privilege account in the application, demonstrating how a single unresolved IDOR can cascade into complete compromise.

## 7. Remediation

- Fix the root IDOR (see Lab 05 remediation) — this is the single most impactful control for Labs 05–08.
- Never pre-fill or return a stored password (plaintext or otherwise) to the client under any circumstances; password fields should always render empty.
- Store passwords exclusively as salted, strong one-way hashes (e.g., bcrypt/argon2) — a correctly hashed password cannot be 'disclosed' in a usable form even on a logic failure.
- Apply defense-in-depth: even if one layer fails, credential fields should never be a renderable attribute of a user profile API/template.

## 8. References

- [PortSwigger — Access Control Vulnerabilities](https://portswigger.net/web-security/access-control)
- [OWASP Top 10 2021 — A01: Broken Access Control](https://owasp.org/Top10/A01_2021-Broken_Access_Control/)
- [CWE-522: Insufficiently Protected Credentials](https://cwe.mitre.org/data/definitions/522.html)

---

*Writeup produced for educational and authorized lab purposes only (PortSwigger Web Security Academy environment).*
