# Lab 07 — User ID Controlled by Request Parameter with Data Leakage in Redirect

[![Category](https://img.shields.io/badge/Category-Broken%20Access%20Control-red)]()
[![Type](https://img.shields.io/badge/Type-Horizontal%20Broken%20Access%20Control-orange)]()
[![Severity](https://img.shields.io/badge/Severity-High-critical)]()
[![Source](https://img.shields.io/badge/Source-PortSwigger%20Web%20Security%20Academy-blue)](https://portswigger.net/web-security/access-control)

> Part of the [Broken Access Control Labs](../README.md) series — PortSwigger Web Security Academy.

---

## 1. Overview

| Field | Detail |
|---|---|
| **Lab Name** | User ID Controlled by Request Parameter with Data Leakage in Redirect |
| **Vulnerability Class** | Horizontal Broken Access Control (Sensitive Data Leak via 302 Response Body) |
| **Weakness (CWE)** | CWE-201: Insertion of Sensitive Information Into Sent Data |
| **Severity** | High |
| **Objective** | Retrieve Carlos's API key despite an apparent redirect-based access control. |

## 2. Summary

The server correctly decides to deny access and issues a `302` redirect — but only *after* fully rendering the page template (including sensitive data) server-side. The redirect header is set, yet the already-generated HTML body is still transmitted and visible to anyone inspecting raw HTTP traffic.

## 3. Proof of Concept (PoC)

**Step 1 — Intercept the request:**  
Using Burp Suite, intercept a request to `/my-account?id=carlos` while authenticated as a low-privileged user.

**Step 2 — Observe the redirect:**  
Note that the browser follows a `302 Found` response redirecting to `/login`, which normally hides the page content from the user.

**Step 3 — Inspect the raw response:**  
In Burp (not the browser, which auto-follows redirects), examine the **body** of the `302` response rather than just the `Location` header.

**Step 4 — Exploit:**  
Locate Carlos's API key, which is already rendered inside the HTML body of the redirect response, despite the user never actually reaching that page in the browser.


## 4. Evidence (Request / Response)

```http
GET /my-account?id=carlos HTTP/2
Host: <lab-id>.web-security-academy.net
Cookie: session=<low-priv-session>

HTTP/2 302 Found
Location: /login
Content-Length: 3184

<html>...API Key: 4e8a1...c90d...</html>
```

## 5. Root Cause Analysis

The authorization check and the response-generation logic are not properly sequenced: the access-denial decision (redirect) is appended to the response *after* the full page — including the sensitive data query — has already executed and been serialized into the body.

### Attacker vs. Defender Perspective

- **Attacker view:** Identify an object reference (ID/GUID/filename) in a request and systematically substitute values belonging to other users to access data outside the attacker's own scope.
- **Defender view:** Never trust a client-supplied identifier; always verify server-side that the authenticated session owns (or is authorized for) the specific object being requested.

## 6. Impact

Browsers hide this leak because they auto-follow redirects without rendering the body, creating a false sense of security. Any tool or attacker that inspects raw HTTP traffic (proxies, curl, Burp) can trivially extract the leaked data.

## 7. Remediation

- Perform the authorization check *first*, before any sensitive data is queried or rendered — halt execution immediately and return an empty/minimal redirect body.
- Never assume a `3xx` status code alone is sufficient; audit the actual response body content for every redirect from a sensitive endpoint.
- Adopt a 'fail closed, fail fast' pattern: return `403`/`302` with no body as early as possible in the request-handling pipeline.
- Include automated response-body assertions in security regression tests for all redirect-protected routes.

## 8. References

- [PortSwigger — Access Control Vulnerabilities](https://portswigger.net/web-security/access-control)
- [OWASP Top 10 2021 — A01: Broken Access Control](https://owasp.org/Top10/A01_2021-Broken_Access_Control/)
- [CWE-201: Insertion of Sensitive Information Into Sent Data](https://cwe.mitre.org/data/definitions/201.html)

---

*Writeup produced for educational and authorized lab purposes only (PortSwigger Web Security Academy environment).*
