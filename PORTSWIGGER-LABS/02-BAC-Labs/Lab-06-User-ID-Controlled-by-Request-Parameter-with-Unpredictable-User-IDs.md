# Lab 06 — User ID Controlled by Request Parameter with Unpredictable User IDs

[![Category](https://img.shields.io/badge/Category-Broken%20Access%20Control-red)]()
[![Type](https://img.shields.io/badge/Type-Horizontal%20Broken%20Access%20Control-orange)]()
[![Severity](https://img.shields.io/badge/Severity-High-critical)]()
[![Source](https://img.shields.io/badge/Source-PortSwigger%20Web%20Security%20Academy-blue)](https://portswigger.net/web-security/access-control)

> Part of the [Broken Access Control Labs](../README.md) series — PortSwigger Web Security Academy.

---

## 1. Overview

| Field | Detail |
|---|---|
| **Lab Name** | User ID Controlled by Request Parameter with Unpredictable User IDs |
| **Vulnerability Class** | Horizontal Broken Access Control (IDOR with Enumerable GUID) |
| **Weakness (CWE)** | CWE-639: Authorization Bypass Through User-Controlled Key |
| **Severity** | High |
| **Objective** | Discover Carlos's GUID, access his account page, and retrieve his API key. |

## 2. Summary

Unpredictable GUIDs are used in place of sequential IDs, but the same ownership-check flaw from Lab 05 persists — and the GUIDs are inadvertently disclosed through an unrelated feature (blog author links), defeating the unpredictability defense.

## 3. Proof of Concept (PoC)

**Step 1 — Locate leaked identifier:**  
Navigate to a blog post authored by `carlos`.

**Step 2 — Extract the GUID:**  
Inspect the author hyperlink to recover Carlos's unpredictable identifier:

```html
<a href="/author?id=85a12d...4a">carlos</a>
```

**Step 3 — Reuse in vulnerable endpoint:**  
Navigate to `/my-account?id=85a12d...4a`, substituting the GUID into the previously identified IDOR-prone parameter.

**Step 4 — Exploit:**  
Retrieve Carlos's API key from the rendered account page.


## 4. Evidence (Request / Response)

```http
GET /my-account?id=85a12d...4a HTTP/2
Host: <lab-id>.web-security-academy.net
Cookie: session=<wiener-session>

HTTP/2 200 OK
...
API Key: 7bd90...e21f
```

## 5. Root Cause Analysis

Unpredictability (GUIDs) was used as a substitute for proper authorization, but (1) the core access-control check is still missing, and (2) the 'secret' identifiers are leaked through a separate, lower-sensitivity feature, nullifying the protection entirely.

### Attacker vs. Defender Perspective

- **Attacker view:** Identify an object reference (ID/GUID/filename) in a request and systematically substitute values belonging to other users to access data outside the attacker's own scope.
- **Defender view:** Never trust a client-supplied identifier; always verify server-side that the authenticated session owns (or is authorized for) the specific object being requested.

## 6. Impact

Demonstrates that obscurity-based mitigations (hard-to-guess IDs) collapse the moment the identifier leaks anywhere else in the application — underscoring that proper ownership validation is non-negotiable.

## 7. Remediation

- Do not treat unpredictable identifiers (GUIDs/UUIDs) as a substitute for authorization checks — they mitigate brute-forcing, not missing access control.
- Audit all application surfaces (APIs, HTML, metadata, error messages, sitemaps) for inadvertent disclosure of internal identifiers.
- Enforce the same ownership/role check described in Lab 05 regardless of how unguessable the identifier is.
- Consider using different identifiers for public-facing content (e.g., blog author slugs) and internal account lookups, so a leak in one context doesn't compromise another.

## 8. References

- [PortSwigger — Access Control Vulnerabilities](https://portswigger.net/web-security/access-control)
- [OWASP Top 10 2021 — A01: Broken Access Control](https://owasp.org/Top10/A01_2021-Broken_Access_Control/)
- [CWE-639: Authorization Bypass Through User-Controlled Key](https://cwe.mitre.org/data/definitions/639.html)

---

*Writeup produced for educational and authorized lab purposes only (PortSwigger Web Security Academy environment).*
