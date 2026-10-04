# Lab 02 — Unprotected Admin Functionality with Unpredictable URL

[![Category](https://img.shields.io/badge/Category-Broken%20Access%20Control-red)]()
[![Type](https://img.shields.io/badge/Type-Vertical%20Broken%20Access%20Control-orange)]()
[![Severity](https://img.shields.io/badge/Severity-High-critical)]()
[![Source](https://img.shields.io/badge/Source-PortSwigger%20Web%20Security%20Academy-blue)](https://portswigger.net/web-security/access-control)

> Part of the [Broken Access Control Labs](../README.md) series — PortSwigger Web Security Academy.

---

## 1. Overview

| Field | Detail |
|---|---|
| **Lab Name** | Unprotected Admin Functionality with Unpredictable URL |
| **Vulnerability Class** | Vertical Broken Access Control |
| **Weakness (CWE)** | CWE-284: Improper Access Control |
| **Severity** | High |
| **Objective** | Access the admin panel at an unpredictable URL and delete user `carlos`. |

## 2. Summary

The admin panel URL is randomized, but the path is leaked in client-side JavaScript that conditionally renders a link to it. Since all JS served to the browser is inherently public, the obfuscation provides no real protection.

## 3. Proof of Concept (PoC)

**Step 1 — Inspect client-side assets:**  
Open browser DevTools (or view page source) on the homepage and locate the loaded `.js` file(s).

**Step 2 — Locate the conditional route:**  
Search the script for conditional admin-link logic, e.g.:

```javascript
if (isAdmin) {
    // render link to '/admin-lmn62t'
}
```

**Step 3 — Direct access:**  
Navigate directly to the disclosed path `/admin-lmn62t`, bypassing the `isAdmin` client-side gate entirely.

**Step 4 — Exploit:**  
Delete user `carlos` from the now-accessible admin panel.


## 4. Evidence (Request / Response)

```http
GET /admin-lmn62t HTTP/2
Host: <lab-id>.web-security-academy.net
Cookie: session=<low-priv-session>

HTTP/2 200 OK
...
<h1>Admin panel</h1>
```

## 5. Root Cause Analysis

Client-side conditional rendering (`if (isAdmin) {...}`) is a UI/UX convenience, not a security boundary. The JavaScript bundle — including the hardcoded unpredictable path — is downloaded by every visitor regardless of role, so the 'secret' URL is never actually secret.

### Attacker vs. Defender Perspective

- **Attacker view:** Enumerate or guess hidden/unprotected paths and request them directly, bypassing any client-side-only gating.
- **Defender view:** Enforce server-side, deny-by-default authorization on every sensitive route, independent of how it was discovered.

## 6. Impact

Identical to Lab 01: any user who inspects client-side code can bypass intended UI restrictions and reach privileged functionality directly, since the backend performs no authorization check of its own.

## 7. Remediation

- Treat all client-side JavaScript, including conditionally rendered UI logic, as fully readable by any user — never embed sensitive routes or secrets in it.
- Enforce authorization decisions exclusively on the server for every request to a sensitive endpoint.
- Use randomized URLs only as defense-in-depth (to reduce casual discovery), never as the sole access control mechanism.
- Add automated tests (e.g., in CI) that assert protected routes return `403` for non-privileged sessions.

## 8. References

- [PortSwigger — Access Control Vulnerabilities](https://portswigger.net/web-security/access-control)
- [OWASP Top 10 2021 — A01: Broken Access Control](https://owasp.org/Top10/A01_2021-Broken_Access_Control/)
- [CWE-284: Improper Access Control](https://cwe.mitre.org/data/definitions/284.html)

---

*Writeup produced for educational and authorized lab purposes only (PortSwigger Web Security Academy environment).*
