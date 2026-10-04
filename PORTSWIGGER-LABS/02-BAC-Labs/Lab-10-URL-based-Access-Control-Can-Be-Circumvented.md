# Lab 10 — URL-based Access Control Can Be Circumvented

[![Category](https://img.shields.io/badge/Category-Broken%20Access%20Control-red)]()
[![Type](https://img.shields.io/badge/Type-Vertical%20Broken%20Access%20Control-orange)]()
[![Severity](https://img.shields.io/badge/Severity-Critical-critical)]()
[![Source](https://img.shields.io/badge/Source-PortSwigger%20Web%20Security%20Academy-blue)](https://portswigger.net/web-security/access-control)

> Part of the [Broken Access Control Labs](../README.md) series — PortSwigger Web Security Academy.

---

## 1. Overview

| Field | Detail |
|---|---|
| **Lab Name** | URL-based Access Control Can Be Circumvented |
| **Vulnerability Class** | Vertical Broken Access Control (Reverse Proxy / Edge-Layer Bypass) |
| **Weakness (CWE)** | CWE-441: Unintended Proxy or Intermediary ('Confused Deputy') |
| **Severity** | Critical |
| **Objective** | Access `/admin` and delete user `carlos` by bypassing front-end URL-based restrictions. |

## 2. Summary

Access control is enforced only at the edge (reverse proxy), which pattern-matches on the literal request path. The backend application separately trusts a routing-override header (`X-Original-URL`) intended for internal rewriting, creating a mismatch an attacker can exploit by targeting a path the proxy allows while redirecting the backend's actual routing decision.

## 3. Proof of Concept (PoC)

**Step 1 — Confirm the block:**  
Request `/admin` directly — the front-end reverse proxy returns `403 Forbidden` before the request reaches the backend application.

**Step 2 — Identify the bypass surface:**  
Note that the reverse proxy permits requests to `/`, since its block rule is scoped only to the literal `/admin` path.

**Step 3 — Craft the bypass:**  
Send a request to the permitted root path while injecting a routing header the backend trusts:

```http
GET / HTTP/2
X-Original-URL: /admin/delete?username=carlos
```

**Step 4 — Exploit:**  
The backend reads `X-Original-URL` and internally routes the request to the privileged `/admin/delete` handler, which executes successfully.


## 4. Evidence (Request / Response)

```http
GET / HTTP/2
Host: <lab-id>.web-security-academy.net
X-Original-URL: /admin/delete?username=carlos
Cookie: session=<low-priv-session>

HTTP/2 302 Found
Location: /admin/delete?username=carlos
```

## 5. Root Cause Analysis

A classic 'confused deputy' / edge-layer trust mismatch: the proxy enforces access control based solely on the externally visible path, while the backend application honors an internal rewrite header without re-validating authorization after the effective route changes.

### Attacker vs. Defender Perspective

- **Attacker view:** Target a path permitted by the edge layer while using an internal routing-override header to redirect the backend's actual route resolution.
- **Defender view:** Re-run full authorization on the effective (post-rewrite) route and strip untrusted routing headers at the perimeter.

## 6. Impact

Completely defeats perimeter-based access control. Any path-based restriction enforced only at the proxy layer (and not re-validated by the application itself) can potentially be bypassed via routing-override headers (`X-Original-URL`, `X-Rewrite-URL`, `X-Forwarded-*`, etc.).

## 7. Remediation

- Never rely solely on an edge/proxy layer for authorization decisions — the application itself must independently enforce access control on the *effective* route it processes, not just the externally requested path.
- Strip or ignore internal routing-override headers (`X-Original-URL`, `X-Rewrite-URL`) at the perimeter unless they originate from a fully trusted, isolated internal network segment.
- If such headers are operationally necessary, re-run the full authorization pipeline *after* the rewrite is applied, using the final resolved path.
- Perform regular configuration audits comparing proxy ACL rules against all headers the backend trusts for routing.

## 8. References

- [PortSwigger — Access Control Vulnerabilities](https://portswigger.net/web-security/access-control)
- [OWASP Top 10 2021 — A01: Broken Access Control](https://owasp.org/Top10/A01_2021-Broken_Access_Control/)
- [CWE-441: Unintended Proxy or Intermediary](https://cwe.mitre.org/data/definitions/441.html)

---

*Writeup produced for educational and authorized lab purposes only (PortSwigger Web Security Academy environment).*
