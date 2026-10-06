# Lab 06 — Validation of File Extension with Null Byte Bypass

[![Category](https://img.shields.io/badge/Category-Path%20Traversal-purple)]()
[![Type](https://img.shields.io/badge/Type-Path%20Traversal%20-%20Null%20Byte%20Injection-orange)]()
[![Severity](https://img.shields.io/badge/Severity-High-critical)]()
[![Source](https://img.shields.io/badge/Source-PortSwigger%20Web%20Security%20Academy-blue)](https://portswigger.net/web-security/file-path-traversal)

> Part of the [Path Traversal Labs](./README.md) series — PortSwigger Web Security Academy.

---

## 1. Overview

| Field | Detail |
|---|---|
| **Lab Name** | Validation of File Extension with Null Byte Bypass |
| **Vulnerability Class** | Path Traversal — Null Byte Injection |
| **Weakness (CWE)** | CWE-626: Null Byte Interaction Error |
| **Severity** | High |
| **Objective** | Bypass a required-file-extension check by injecting a null byte that truncates the string before the extension, as interpreted by the underlying C-based filesystem API. |

## 2. Summary

The application-level check validates that the filename *ends with* an approved extension (e.g. `.png`), operating on the full string including anything after a null byte. However, the lower-level filesystem API (implemented in C, which treats strings as null-terminated) stops reading the string the instant it encounters a `%00` (null byte), silently truncating everything that follows — including the extension the high-level check saw.

## 3. Proof of Concept (PoC)

**Step 1 — Capture the image request:**  
Note the extension requirement by observing normal requests, e.g. `GET /image?filename=47.jpg`.

**Step 2 — Confirm the extension check:**  
Submitting a traversal payload without a valid trailing extension (e.g. `../../../etc/passwd`) is rejected with something like "Only PNG images are allowed!".

**Step 3 — Append a null byte before a valid extension:**  
Craft a payload where the real target is followed by a URL-encoded null byte and a fake, approved extension:

```http
GET /image?filename=../../../etc/passwd%00.png HTTP/1.1
```

**Step 4 — Observe the truncation:**  
The application's high-level check sees the full string ending in `.png` and approves it. The underlying C-based file-read function treats `%00` (decoded to a literal null byte) as the end of the string, truncating everything from `.png` onward and opening `/etc/passwd` instead.


## 4. Evidence (Request / Response)

```http
GET /image?filename=../../../etc/passwd%00.png HTTP/1.1
Host: <lab-id>.web-security-academy.net

HTTP/1.1 200 OK
...
root:x:0:0:root:/root:/bin/bash
...
```

## 5. Root Cause Analysis

A mismatch between how the high-level application code interprets strings (full-length, up to the actual end of the string) and how the underlying C-based filesystem API interprets them (null-terminated — stopping at the first `\0` byte). The extension check operates on the "safe" full string while the actual file open operates on the truncated one, so the two layers disagree about what the real filename is.

### Attacker vs. Defender Perspective

- **Attacker view:** When an extension suffix is enforced, append a URL-encoded null byte (%00) followed by a fake valid extension to truncate the string at the C-API level.
- **Defender view:** Reject any null bytes outright before processing, and never rely on extension checks as the sole access boundary.

## 6. Impact

Complete bypass of extension-based allow-listing, enabling arbitrary file read regardless of what extension restriction was intended to enforce. This specific technique is largely historical (patched in PHP ≥ 5.3.4 and most modern language runtimes) but can still surface in legacy systems, C/C++-based services, or applications that shell out to system calls.

## 7. Remediation

- Upgrade to modern language runtimes/frameworks where this specific null-byte truncation behavior has been patched at the string-handling level.
- Never rely on extension suffix checks as a security boundary for file access; combine with full canonical-path validation as described in the other labs in this series.
- Reject any input containing null bytes (`%00`, `\0`) outright before any further processing, as there is no legitimate use case for them in a filename parameter.

## 8. References

- [PortSwigger — Path Traversal](https://portswigger.net/web-security/file-path-traversal)
- [OWASP — Path Traversal](https://owasp.org/www-community/attacks/Path_Traversal)
- [CWE-626: Null Byte Interaction Error](https://cwe.mitre.org/data/definitions/626.html)

---

*Writeup produced for educational and authorized lab purposes only (PortSwigger Web Security Academy environment).*
