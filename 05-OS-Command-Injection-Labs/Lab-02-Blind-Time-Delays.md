# Lab 02 — Blind OS Command Injection with Time Delays

## 1. Overview

| Field | Value |
|---|---|
| Vulnerability Class | OS Command Injection (Blind / Time-based) |
| CWE | CWE-78 |
| Difficulty | Practitioner |
| Source | PortSwigger Web Security Academy |
| Objective | Trigger a 10-second delay to prove blind command injection in the feedback function |

## 2. Summary

The feedback submission feature passes user-supplied details into a shell command that invokes `mail` to notify a site administrator. The command's output is never returned in the HTTP response, so verbose detection techniques (like in Lab 01) do not apply. Because there is no visible output channel, the vulnerability must be confirmed **blindly**, using a time-delay payload (`ping`) and measuring the server's response time.

## 3. Proof of Concept (PoC)

1. Go to the **Submit feedback** page, fill in dummy values, and submit the form.
2. Capture the `POST /feedback/submit` request in Burp Suite and send it to Repeater.
3. Replace the `email` parameter with a payload that chains a 10-second `ping`:
   ```
   email=x||ping+-c+10+127.0.0.1||
   ```
4. Send the request and observe the response timer in Repeater.

## 4. Evidence

**Request (Repeater):**
```
POST /feedback/submit HTTP/2
Host: <lab-id>.web-security-academy.net
Content-Type: application/x-www-form-urlencoded

csrf=<token>&name=test&email=x||ping+-c+10+127.0.0.1||&subject=test&message=test
```

**Observed behaviour:** the response that would normally return in a few hundred milliseconds instead takes approximately **10,000 ms** before returning `200 OK`, confirming server-side execution of the injected `ping` command.

## 5. Root Cause Analysis

**Attacker perspective:** The backend builds a command similar to:
```
mail -s "Subject" -aFrom:<email> feedback@vulnerable-website.com
```
Submitting `email=x||ping+-c+10+127.0.0.1||` produces:
```
mail -s "Subject" -aFrom:x||ping -c 10 127.0.0.1|| feedback@vulnerable-website.com
```
The first `mail` invocation (with the malformed `-aFrom:x`) fails, so the shell's `||` operator (run-on-failure) executes the injected `ping -c 10 127.0.0.1`, which sends 10 ICMP packets to the loopback adapter — each taking roughly one second — forcing the server to pause for ~10 seconds before continuing. The trailing `||` isolates the injected command from the rest of the original command line so it doesn't break the surrounding syntax.

**Defender perspective:** User input flows unsanitised into a shell-invoked mail command, and because no output is surfaced to the client, the team likely assumed this path was "safe" — a common but incorrect assumption that blind injection surfaces are lower risk.

**Troubleshooting notes:**
- If `||` doesn't trigger (i.e. the first command doesn't fail), try `&` or `;` instead, since `&` executes in the background regardless of the first command's exit status.
- On Windows targets, the equivalent payload uses `-n` instead of `-c`: `& ping -n 10 127.0.0.1 &`.

## 6. Impact

Even without visible output, blind command injection still grants arbitrary command execution on the server. An attacker can escalate from mere detection to full exploitation (file read/write, reverse shells, data exfiltration via OAST), as demonstrated in the subsequent labs in this series.

## 7. Remediation

- Avoid shelling out to `mail`/`sendmail`; use a language-native mail library (e.g. an SMTP client) instead.
- If a shell call is unavoidable, pass arguments as a parameter array (not an interpolated string) via an API that does not invoke `/bin/sh`.
- Strictly validate the `email` field server-side (proper email format via a regex/library, not a shell pass-through).
- Monitor for anomalous response-time spikes, which can indicate blind injection attempts in production.

## 8. References

- [PortSwigger — Blind OS command injection with time delays](https://portswigger.net/web-security/os-command-injection)
- [CWE-78](https://cwe.mitre.org/data/definitions/78.html)
