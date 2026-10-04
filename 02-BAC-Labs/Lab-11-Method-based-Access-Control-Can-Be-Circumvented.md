# Lab 11 — Method-based Access Control Can Be Circumvented

[![Category](https://img.shields.io/badge/Category-Broken%20Access%20Control-red)]()
[![Type](https://img.shields.io/badge/Type-Vertical%20Broken%20Access%20Control-orange)]()
[![Severity](https://img.shields.io/badge/Severity-High-critical)]()
[![Source](https://img.shields.io/badge/Source-PortSwigger%20Web%20Security%20Academy-blue)](https://portswigger.net/web-security/access-control)

> Part of the [Broken Access Control Labs](../README.md) series — PortSwigger Web Security Academy.

---

## 1. Overview

| Field | Detail |
|---|---|
| **Lab Name** | Method-based Access Control Can Be Circumvented |
| **Vulnerability Class** | Vertical Broken Access Control (HTTP Method Bypass) |
| **Weakness (CWE)** | CWE-650: Trusting HTTP Permission Methods on the Server Side |
| **Severity** | High |
| **Objective** | Elevate the privileges of user `wiener` by bypassing method-specific authorization. |

## 2. Summary

The authorization middleware is bound specifically to the `POST` verb for `/admin-roles`, but the application's router maps the same handler logic to both `POST` and `GET` — and only the `POST` path passes through the authorization check.

## 3. Proof of Concept (PoC)

**Step 1 — Capture the legitimate flow:**  
Log in as `administrator`, promote a test user, and capture the request in Burp proxy history:

```http
POST /admin-roles HTTP/2
...
username=carlos&action=upgrade
```

**Step 2 — Confirm the control:**  
Log in as `wiener` and replay the identical `POST /admin-roles` request — the server correctly returns `403 Forbidden`.

**Step 3 — Switch HTTP method:**  
Resend the same parameters using `GET` instead of `POST`:

```http
GET /admin-roles?username=wiener&action=upgrade HTTP/2
```

**Step 4 — Exploit:**  
The `GET` variant of the route is not covered by the authorization middleware and executes the privilege upgrade successfully.


## 4. Evidence (Request / Response)

```http
GET /admin-roles?username=wiener&action=upgrade HTTP/2
Host: <lab-id>.web-security-academy.net
Cookie: session=<wiener-session>

HTTP/2 302 Found
Location: /admin-roles
```

## 5. Root Cause Analysis

Authorization logic is attached at the route-method level (e.g., `@app.post('/admin-roles', requires_admin)`) rather than at the underlying handler/controller level, so any alternate route binding to the same business logic (`GET`, `PUT`, `HEAD`) silently skips the check.

### Attacker vs. Defender Perspective

- **Attacker view:** Probe the same endpoint with alternate HTTP verbs to find one the authorization middleware does not cover.
- **Defender view:** Attach authorization checks to the handler logic itself, and explicitly restrict/reject unsupported methods.

## 6. Impact

Any endpoint whose authorization is wired per-HTTP-method rather than per-handler is vulnerable to trivial bypass by simply switching the verb — a common and easily overlooked pattern in frameworks that implicitly register multiple methods.

## 7. Remediation

- Attach authorization checks to the underlying business logic/handler function itself, not to a specific route-method binding, so every verb that can reach it is equally protected.
- Explicitly restrict sensitive endpoints to a single, minimal HTTP method (typically `POST`/`PUT`/`DELETE`) and reject all others with `405 Method Not Allowed`.
- Use centralized authorization middleware/decorators applied globally to route groups (e.g., all `/admin-*` routes) rather than ad hoc per-method wiring.
- Include method-fuzzing (`GET`, `PUT`, `DELETE`, `HEAD`, `OPTIONS`) in access-control test suites for every sensitive route.

## 8. References

- [PortSwigger — Access Control Vulnerabilities](https://portswigger.net/web-security/access-control)
- [OWASP Top 10 2021 — A01: Broken Access Control](https://owasp.org/Top10/A01_2021-Broken_Access_Control/)
- [CWE-650: Trusting HTTP Permission Methods on the Server Side](https://cwe.mitre.org/data/definitions/650.html)

---

*Writeup produced for educational and authorized lab purposes only (PortSwigger Web Security Academy environment).*
