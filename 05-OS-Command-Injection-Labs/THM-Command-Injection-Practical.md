# TryHackMe — OS Command Injection (Room Practical)

## 1. Overview

| Field | Value |
|---|---|
| Vulnerability Class | OS Command Injection (In-band / Verbose) |
| CWE | CWE-78 |
| Source | TryHackMe — "OS Command Injection" room, Task 4 Practical |
| Objective | Identify the application's running user and retrieve the flag at `/home/tryhackme/flag.txt` |

## 2. Summary

The room deploys a live web application with a form that passes user input directly into a system command. There is no further sanitisation, so standard shell metacharacters can be used to chain an additional command onto the application's intended command. Because the application returns command output in its response, this is a verbose/in-band injection, directly reflecting the lessons from PortSwigger Lab 01 in a live-target format.

## 3. Proof of Concept (PoC)

1. Deploy the attached lab machine and open the vulnerable web application in the split-screen browser.
2. Submit a normal, benign value through the form to establish a baseline of expected behaviour and output.
3. Inject a shell metacharacter followed by a reconnaissance command to confirm the vulnerability and identify the running user, e.g.:
   ```
   ; whoami
   ```
4. Observe that `www-data` is reflected in the application's response, confirming command injection as the web server's service account.
5. Retrieve the flag by chaining a `cat` command onto the vulnerable input:
   ```
   ; cat /home/tryhackme/flag.txt
   ```
6. Read the flag value from the reflected output.

## 4. Evidence

**Example injected input:**
```
; cat /home/tryhackme/flag.txt
```

**Reflected output (illustrative):**
```
www-data
THM{COMMAND_INJECTION_COMPLETE}
```

## 5. Root Cause Analysis

**Attacker perspective:** The application concatenates the submitted form value directly into a system command without validating its contents. Using `;` as a sequential command separator (Linux), any text following the first legitimate command is executed as a second, independent command by the shell — in this case `whoami` or `cat /home/tryhackme/flag.txt`.

**Defender perspective:** As with the PortSwigger PHP/Python examples covered in this room (`exec()`, `shell_exec()`, `subprocess.Popen(..., shell=True)`), the root cause is trusting user-supplied input as part of a shell command with no intermediate validation layer. The fact that the application runs as `www-data` (a low-privilege account) limits — but does not eliminate — the blast radius.

**Alternative payloads (per the room's own guidance, multiple approaches should work):**
- `; whoami` / `; cat /home/tryhackme/flag.txt` — sequential separator
- `&& whoami` — run on success
- `| whoami` — pipe
- `%0a whoami` (newline) — when semicolons are filtered

## 6. Impact

Full arbitrary command execution as `www-data`, enabling the attacker to read sensitive files (as demonstrated by the flag retrieval), enumerate the filesystem, and potentially pivot toward privilege escalation if further misconfigurations (e.g. SUID binaries, writable cron jobs, excessive sudo rights) exist on the host.

## 7. Remediation

- Replace direct shell invocation with safe, purpose-built APIs for the required functionality (e.g. database lookups instead of `grep` over a flat file, as discussed in the room's PHP example).
- Apply server-side input validation (e.g. `filter_input()` with `FILTER_VALIDATE_NUMBER`/regex allowlists) and reject anything outside the expected format.
- Never rely solely on client-side validation (HTML `pattern` attributes) — it is trivially bypassed via tools like Burp Suite or `curl`.
- Apply defence in depth: combine validation, allowlisting, and least-privilege execution, since no single control (e.g. character-blacklisting/escaping) is guaranteed to withstand bypass techniques such as hex-encoding.

## 8. References

- TryHackMe — OS Command Injection room
- [CWE-78](https://cwe.mitre.org/data/definitions/78.html)
- [OWASP Top 10:2025 — A05: Injection](https://owasp.org/Top10/)
