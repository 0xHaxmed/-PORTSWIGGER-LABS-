# Lab 02 — Traversal Sequences Blocked with Absolute Path Bypass

[![Category](https://img.shields.io/badge/Category-Path%20Traversal-purple)]()
[![Type](https://img.shields.io/badge/Type-Path%20Traversal%20-%20Blacklist%20Bypass-orange)]()
[![Severity](https://img.shields.io/badge/Severity-High-critical)]()
[![Source](https://img.shields.io/badge/Source-PortSwigger%20Web%20Security%20Academy-blue)](https://portswigger.net/web-security/file-path-traversal)

> Part of the [Path Traversal Labs](./README.md) series — PortSwigger Web Security Academy.

---

## 1. Overview

| Field | Detail |
|---|---|
| **Lab Name** | Traversal Sequences Blocked with Absolute Path Bypass |
| **Vulnerability Class** | Path Traversal — Blacklist Bypass (Absolute Path) |
| **Weakness (CWE)** | CWE-22 / CWE-23: Relative Path Traversal |
| **Severity** | High |
| **Objective** | Bypass a filter that blocks relative `../` sequences by supplying an absolute path directly. |

## 2. Summary

The application blocks input containing `../`, but the underlying path-joining function treats an absolute path (one starting with `/`) as a complete override — discarding the intended base directory entirely rather than appending to it.

## 3. Proof of Concept (PoC)

**Step 1 — Capture the image request:**  
Intercept a normal image request, e.g. `GET /image?filename=42.jpg`.

**Step 2 — Confirm the blacklist:**  
Try the classic payload `filename=../../../etc/passwd` — the server rejects it (e.g. `400 Bad Request`) since `../` is detected and blocked.

**Step 3 — Bypass with an absolute path:**  
Replace the value with the target file's full absolute path, with no traversal sequences at all:

```http
GET /image?filename=/etc/passwd HTTP/1.1
```

**Step 4 — Confirm the read:**  
The server's path-joining logic (e.g. `os.path.join(base, input)` in Python, or equivalent) treats the absolute input as authoritative and discards the base directory, reading `/etc/passwd` directly.


## 4. Evidence (Request / Response)

```http
GET /image?filename=/etc/passwd HTTP/1.1
Host: <lab-id>.web-security-academy.net

HTTP/1.1 200 OK
...
root:x:0:0:root:/root:/bin/bash
...
```

## 5. Root Cause Analysis

Many path-joining utilities across languages (Python's `os.path.join`, Node's `path.join` in some configurations, etc.) special-case an absolute second argument: when the user-supplied component already starts with `/`, the function returns it as-is, silently discarding the base directory it was supposed to be anchored to.

### Attacker vs. Defender Perspective

- **Attacker view:** When ../ is blocked, try supplying an absolute path directly — many path-join functions silently discard the base directory when given one.
- **Defender view:** Never trust blacklist-only filters; always canonicalize the final path and verify containment within the base directory, which also defeats absolute-path overrides.

## 6. Impact

Same as simple-case traversal — full arbitrary file read — but demonstrates that blacklisting the `../` string alone is insufficient; the underlying path-handling semantics must also be understood and defended against.

## 7. Remediation

- Never rely on blacklisting specific substrings (`../`) as the sole defense; this is a weak, bypassable control.
- After concatenation, always resolve to a canonical absolute path and explicitly verify it is still contained within the intended base directory — this single check also defeats absolute-path bypass.
- Be aware of language/framework-specific path-joining quirks (absolute-path override behavior) when choosing how to combine user input with a base path.

## 8. References

- [PortSwigger — Path Traversal](https://portswigger.net/web-security/file-path-traversal)
- [OWASP — Path Traversal](https://owasp.org/www-community/attacks/Path_Traversal)
- [CWE-22: Improper Limitation of a Pathname to a Restricted Directory](https://cwe.mitre.org/data/definitions/22.html)

---

*Writeup produced for educational and authorized lab purposes only (PortSwigger Web Security Academy environment).*
