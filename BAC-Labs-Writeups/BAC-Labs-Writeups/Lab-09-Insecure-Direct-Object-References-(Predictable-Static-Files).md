# Lab 09 — Insecure Direct Object References (Predictable Static Files)

[![Category](https://img.shields.io/badge/Category-Broken%20Access%20Control-red)]()
[![Type](https://img.shields.io/badge/Type-Horizontal%20Broken%20Access%20Control-orange)]()
[![Severity](https://img.shields.io/badge/Severity-High-critical)]()
[![Source](https://img.shields.io/badge/Source-PortSwigger%20Web%20Security%20Academy-blue)](https://portswigger.net/web-security/access-control)

> Part of the [Broken Access Control Labs](../README.md) series — PortSwigger Web Security Academy.

---

## 1. Overview

| Field | Detail |
|---|---|
| **Lab Name** | Insecure Direct Object References (Predictable Static Files) |
| **Vulnerability Class** | Horizontal Broken Access Control (IDOR via Predictable Filenames) |
| **Weakness (CWE)** | CWE-639: Authorization Bypass Through User-Controlled Key |
| **Severity** | High |
| **Objective** | Retrieve Carlos's password from a leaked support chat transcript. |

## 2. Summary

Chat transcripts are stored as static files with sequential, predictable names and served without any ownership check tying the requesting session to the transcript owner.

## 3. Proof of Concept (PoC)

**Step 1 — Generate a reference object:**  
Start a live chat session with support to obtain a legitimate transcript.

**Step 2 — Identify the pattern:**  
Click **View transcript** and note the predictable path: `/download-transcript?file=3.txt`.

**Step 3 — Enumerate:**  
Request neighboring predictable filenames: `/download-transcript?file=1.txt`, `/download-transcript?file=2.txt`, etc. (ideal candidate for Burp Intruder with a numeric range).

**Step 4 — Exploit:**  
Identify a transcript in which the support bot discloses `carlos`'s login credentials in plaintext, then log in as `carlos` and delete `wiener`.


## 4. Evidence (Request / Response)

```http
GET /download-transcript?file=1.txt HTTP/2
Host: <lab-id>.web-security-academy.net
Cookie: session=<any-session>

HTTP/2 200 OK
...
Support: Your username is carlos and your password is <leaked>...
```

## 5. Root Cause Analysis

File retrieval is keyed purely on a sequential filename with no association check between the requesting user's session and the actual owner of that transcript record.

### Attacker vs. Defender Perspective

- **Attacker view:** Identify an object reference (ID/GUID/filename) in a request and systematically substitute values belonging to other users to access data outside the attacker's own scope.
- **Defender view:** Never trust a client-supplied identifier; always verify server-side that the authenticated session owns (or is authorized for) the specific object being requested.

## 6. Impact

Automatable, low-effort enumeration (e.g., Burp Intruder with a numeric payload list) exposes an unbounded set of historical support conversations, which frequently contain credentials or other sensitive PII shared during troubleshooting.

## 7. Remediation

- Never expose internal resources via sequential, guessable filenames; use random, non-enumerable identifiers (UUIDv4) at minimum.
- Enforce server-side ownership checks on every file-serving endpoint — validate that `session.user` is authorized to access the specific `file` requested.
- Avoid ever transmitting credentials in plaintext through support channels; train/automate support bots to never repeat full passwords back to users in any persisted medium.
- Rate-limit and monitor sequential access patterns to file-download endpoints to detect enumeration attempts.

## 8. References

- [PortSwigger — Access Control Vulnerabilities](https://portswigger.net/web-security/access-control)
- [OWASP Top 10 2021 — A01: Broken Access Control](https://owasp.org/Top10/A01_2021-Broken_Access_Control/)
- [CWE-639: Authorization Bypass Through User-Controlled Key](https://cwe.mitre.org/data/definitions/639.html)

---

*Writeup produced for educational and authorized lab purposes only (PortSwigger Web Security Academy environment).*
