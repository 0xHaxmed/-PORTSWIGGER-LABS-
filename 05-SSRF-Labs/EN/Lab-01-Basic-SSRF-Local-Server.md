# Lab 01 — Basic SSRF Against the Local Server

[![Category](https://img.shields.io/badge/Category-SSRF-red)]()
[![Type](https://img.shields.io/badge/Type-In-band%20Local%20SSRF-orange)]()
[![Difficulty](https://img.shields.io/badge/Difficulty-Apprentice-9cf)]()
[![Severity](https://img.shields.io/badge/Severity-High-critical)]()
[![Source](https://img.shields.io/badge/Source-PortSwigger%20Web%20Security%20Academy-blue)](https://portswigger.net/web-security/ssrf)

> Part of the [SSRF Labs](./README.md) series — PortSwigger Web Security Academy.

---


## 1. Overview

| Field | Detail |
|---|---|
| **Vulnerability Class** | Server-Side Request Forgery (In-band) |
| **Weakness (CWE)** | CWE-918 |
| **Difficulty** | Apprentice |
| Source | PortSwigger Web Security Academy |
| **Objective** | Access the admin interface at `http://localhost/admin` via the stock checker and delete user `carlos` |

## 2. Summary

The application's stock-check feature accepts a full URL in a `stockApi` parameter and has the server fetch that URL on the client's behalf, returning the result directly in the response. Browsing to `/admin` in a normal browser is blocked because that functionality is only meant to be reachable from the server itself. By redirecting the `stockApi` parameter to `http://localhost/admin`, the request to the admin interface appears to originate from the trusted local machine, bypassing the access control entirely.

## 3. Proof of Concept (PoC)

1. Attempt to browse directly to `/admin` — confirm this is blocked/inaccessible to external users.
2. Open any product page and click **Check stock**.
3. Capture the resulting `POST /product/stock` request in Burp Suite and send it to Repeater.
4. Replace the `stockApi` value with `http://localhost/admin` and send the request — the admin interface HTML is returned in the response.
5. Inspect the returned HTML for the delete link, which follows the pattern `/admin/delete?username=carlos`.
6. Change `stockApi` to that delete URL and send the request to execute the deletion.

## 4. Evidence

**Baseline request:**
```
POST /product/stock HTTP/1.1
Host: <lab-id>.web-security-academy.net
Content-Type: application/x-www-form-urlencoded

stockApi=http://stock.weliketoshop.net:8080/product/stock/check?productId=1&storeId=1
```

**Step 1 — reach the admin panel:**
```
stockApi=http://localhost/admin
```
Response: `200 OK`, body contains the admin panel HTML including `<a href="/admin/delete?username=carlos">Delete</a>`.

**Step 2 — delete the user:**
```
stockApi=http://localhost/admin/delete?username=carlos
```
Response: `200 OK`, the lab status flips to **Solved**.

## 5. Root Cause Analysis

**Attacker perspective:** The stock-checker functionality is essentially an unrestricted server-side URL fetcher. Because the fetch happens on the server, any destination — including the server's own loopback interface — is reachable, and because the admin panel implicitly trusts anything arriving from `localhost`, the usual authentication requirement is skipped entirely.

**Defender perspective:** Several architectural decisions combine to create this flaw: (1) accepting a full attacker-controlled URL instead of an internal identifier, (2) performing the fetch server-side with no destination restriction, and (3) an admin interface that equates "request came from localhost" with "request is authorized." Each of these assumptions is reasonable in isolation (e.g. a disaster-recovery backdoor for admins who lose credentials) but together they create a critical bypass.

## 6. Impact

An attacker gains full, unauthenticated access to sensitive administrative functionality, including the ability to perform destructive actions (user deletion demonstrated here). In real-world applications this pattern frequently extends to full administrative takeover, configuration changes, or access to internal dashboards never meant to be internet-facing.

## 7. Remediation

- Never accept a full, attacker-controlled URL for server-side fetches. Use an internal identifier (e.g. a product/store ID) and resolve it to a trusted, hardcoded destination server-side.
- Do not grant elevated trust based solely on the source IP of a request (`127.0.0.1`/`localhost`). Require proper authentication for all administrative functionality, including from the local host.
- If a local disaster-recovery bypass is genuinely required, gate it behind a separate, non-HTTP-reachable mechanism (e.g. local console access) rather than trusting any request that happens to originate from the loopback interface.
- Apply network segmentation so the application server cannot reach administrative interfaces it doesn't need.

## 8. References

- [PortSwigger — Basic SSRF against the local server](https://portswigger.net/web-security/ssrf)
- [CWE-918](https://cwe.mitre.org/data/definitions/918.html)