# Lab 05 — Validation of Start of Path

[![Category](https://img.shields.io/badge/Category-Path%20Traversal-purple)]()
[![Type](https://img.shields.io/badge/Type-Path%20Traversal%20-%20Prefix%20Validation%20Bypass-orange)]()
[![Severity](https://img.shields.io/badge/Severity-High-critical)]()
[![Source](https://img.shields.io/badge/Source-PortSwigger%20Web%20Security%20Academy-blue)](https://portswigger.net/web-security/file-path-traversal)

> Part of the [Path Traversal Labs](./README.md) series — PortSwigger Web Security Academy.

---

## 1. Overview

| Field | Detail |
|---|---|
| **Lab Name** | Validation of Start of Path |
| **Vulnerability Class** | Path Traversal — Prefix Validation Bypass |
| **Weakness (CWE)** | CWE-22: Improper Limitation of a Pathname to a Restricted Directory |
| **Severity** | High |
| **Objective** | Bypass a check that only verifies the supplied path starts with the expected base folder, without resolving the final canonical path. |

## 2. Summary

The server requires the full file path to be submitted and validates only that it begins with the expected base directory string (e.g. `/var/www/images/`). It never resolves `../` sequences appearing later in the same string before performing the file read, so the required prefix can simply be followed by traversal sequences that walk back out.

## 3. Proof of Concept (PoC)

**Step 1 — Capture the image request:**  
Note that the full path is submitted as the parameter, e.g. `GET /image?filename=/var/www/images/22.jpg`.

**Step 2 — Confirm the prefix check:**  
The server rejects any value that does not literally start with `/var/www/images/`.

**Step 3 — Satisfy the prefix, then traverse:**  
Keep the required prefix intact, then append traversal sequences to walk back out and reach the target file:

```http
GET /image?filename=/var/www/images/../../../etc/passwd HTTP/1.1
```

**Step 4 — Confirm the read:**  
The prefix check passes (`str_starts_with()` returns true), and the filesystem read API then resolves the full path — including the `../` sequences — down to `/etc/passwd`.


## 4. Evidence (Request / Response)

```http
GET /image?filename=/var/www/images/../../../etc/passwd HTTP/1.1
Host: <lab-id>.web-security-academy.net

HTTP/1.1 200 OK
...
root:x:0:0:root:/root:/bin/bash
...
```

## 5. Root Cause Analysis

The validation logic checks only the literal starting substring of the raw input (`str_starts_with(input, BASE_DIR)`), not the canonical/resolved path that will actually be used for the file operation. Because the string technically starts with the required prefix, it passes — even though everything after the prefix walks the resolved path straight back out of the intended directory.

### Attacker vs. Defender Perspective

- **Attacker view:** When a full path is required to start with an allowed folder, keep that prefix intact and simply append ../ sequences afterward.
- **Defender view:** Validate the canonical, resolved path — never the raw string's prefix — against the allowed base directory.

## 6. Impact

Full arbitrary file read, identical to the simple-case lab, despite the presence of what looks like directory-scoping validation — illustrating that prefix checks on raw strings provide no real security guarantee without canonicalization.

## 7. Remediation

- Never validate directory containment on the raw, unresolved input string; always resolve the canonical path first.
- After building the full path, call a canonicalization API (`File.getCanonicalPath()` in Java, `realpath()` in PHP/C, `os.path.realpath()` in Python) and verify the *resolved* path starts with the base directory.
- Reject the request if canonicalization fails or the resolved path falls outside the allowed directory.

## 8. References

- [PortSwigger — Path Traversal](https://portswigger.net/web-security/file-path-traversal)
- [OWASP — Path Traversal](https://owasp.org/www-community/attacks/Path_Traversal)
- [CWE-22: Improper Limitation of a Pathname to a Restricted Directory](https://cwe.mitre.org/data/definitions/22.html)

---

*Writeup produced for educational and authorized lab purposes only (PortSwigger Web Security Academy environment).*
