# Lab 13 — Referer-based Access Control

[![Category](https://img.shields.io/badge/Category-Broken%20Access%20Control-red)]()
[![Type](https://img.shields.io/badge/Type-Vertical%20Broken%20Access%20Control-orange)]()
[![Severity](https://img.shields.io/badge/Severity-High-critical)]()
[![Source](https://img.shields.io/badge/Source-PortSwigger%20Web%20Security%20Academy-blue)](https://portswigger.net/web-security/access-control)

> Part of the [Broken Access Control Labs](../README.md) series — PortSwigger Web Security Academy.

---

## 1. Overview

| Field | Detail |
|---|---|
| **Lab Name** | Referer-based Access Control |
| **Vulnerability Class** | Vertical Broken Access Control (Referer Header Spoofing) |
| **Weakness (CWE)** | CWE-293: Using Referer Field for Authentication |
| **Severity** | High |
| **Objective** | Promote user `wiener` to administrator by forging the `Referer` header. |

## 2. Summary

The application uses the client-supplied, trivially forgeable `Referer` HTTP header as an authorization signal to infer that a request originated from the (supposedly admin-only) admin panel page.

## 3. Proof of Concept (PoC)

**Step 1 — Authenticate:**  
Log in as the low-privileged user `wiener`.

**Step 2 — Trigger the target endpoint:**  
Attempt the promotion request directly: `GET /admin-roles?username=wiener&action=upgrade` — observe that the server rejects it because the request did not originate from the admin panel.

**Step 3 — Forge the Referer header:**  
Intercept the request in Burp and inject a `Referer` header pointing to the admin panel:

```http
GET /admin-roles?username=wiener&action=upgrade HTTP/2
Referer: https://<lab-id>.web-security-academy.net/admin
```

**Step 4 — Exploit:**  
The server grants the request based solely on the spoofed `Referer` value, executing the privilege escalation.


## 4. Evidence (Request / Response)

```http
GET /admin-roles?username=wiener&action=upgrade HTTP/2
Host: <lab-id>.web-security-academy.net
Referer: https://<lab-id>.web-security-academy.net/admin
Cookie: session=<wiener-session>

HTTP/2 302 Found
Location: /admin-roles
```

## 5. Root Cause Analysis

The `Referer` header is entirely client-controlled and can be freely set (or omitted/stripped by proxies and privacy tools) by any HTTP client, including Burp Repeater, curl, or a browser extension. It was never designed as — and must never be used as — an authentication or authorization signal.

### Attacker vs. Defender Perspective

- **Attacker view:** Forge the client-controlled Referer header to make a request appear as though it originated from a trusted, privileged page.
- **Defender view:** Never use the Referer header (or any client-controlled header) as an authorization signal; rely solely on server-side session/role state.

## 6. Impact

Any user capable of crafting raw HTTP requests (i.e., anyone with basic tooling) can bypass this control entirely, since the 'proof' of having come from the admin panel is self-asserted and unverifiable.

## 7. Remediation

- Never use the `Referer` (or any other client-controlled) header as an authorization or authentication signal.
- Enforce authorization strictly via server-side, tamper-proof session state tied to the authenticated user's verified role.
- If origin-of-request validation is genuinely needed (e.g., CSRF protection), use purpose-built mechanisms such as CSRF tokens or the `Origin`/`Sec-Fetch-Site` headers in combination with proper server-side session checks — not as a substitute for role-based authorization.
- Conduct security code reviews specifically flagging any authorization logic that branches on request headers rather than session/role data.

## 8. References

- [PortSwigger — Access Control Vulnerabilities](https://portswigger.net/web-security/access-control)
- [OWASP Top 10 2021 — A01: Broken Access Control](https://owasp.org/Top10/A01_2021-Broken_Access_Control/)
- [CWE-293: Using Referer Field for Authentication](https://cwe.mitre.org/data/definitions/293.html)

---

*Writeup produced for educational and authorized lab purposes only (PortSwigger Web Security Academy environment).*
