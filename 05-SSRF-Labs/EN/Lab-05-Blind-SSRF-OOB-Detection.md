# Lab 05 — Blind SSRF with Out-of-Band Detection

[![Category](https://img.shields.io/badge/Category-SSRF-red)]()
[![Type](https://img.shields.io/badge/Type-Blind%20SSRF%20(OAST)-orange)]()
[![Difficulty](https://img.shields.io/badge/Difficulty-Practitioner-9cf)]()
[![Severity](https://img.shields.io/badge/Severity-High-critical)]()
[![Source](https://img.shields.io/badge/Source-PortSwigger%20Web%20Security%20Academy-blue)](https://portswigger.net/web-security/ssrf)

> Part of the [SSRF Labs](./README.md) series — PortSwigger Web Security Academy.

---


## 1. Overview

| Field | Detail |
|---|---|
| **Vulnerability Class** | Server-Side Request Forgery (Blind / Out-of-Band) |
| **Weakness (CWE)** | CWE-918 |
| **Difficulty** | Practitioner |
| Source | PortSwigger Web Security Academy |
| **Objective** | Cause the application to issue an HTTP request to the public Burp Collaborator server via the `Referer` header |

## 2. Summary

This site runs server-side analytics software that automatically fetches whatever URL is present in the incoming `Referer` header, in order to analyze referring sites. The result of that fetch is never returned anywhere in the application's response, making this a **blind** SSRF — there is no visible output to confirm exploitation through. Because the request is entirely one-way, the only reliable detection method is an out-of-band (OAST) technique: injecting a unique, attacker-controlled domain into `Referer` and monitoring for an inbound DNS/HTTP interaction from the application's infrastructure.

## 3. Proof of Concept (PoC)

1. Browse to any product page, capture the request in Burp Suite, and send it to Repeater.
2. Replace the `Referer` header's domain with a unique Burp Collaborator-generated subdomain (via **Insert Collaborator payload** in Burp Suite Professional).
3. Send the request.
4. Open the **Collaborator** tab and click **Poll now** — since the analytics fetch happens asynchronously, allow a few seconds and poll again if nothing appears immediately.
5. Confirm DNS (and typically HTTP) interactions logged against the injected subdomain, originating from the lab's infrastructure.

## 4. Evidence

**Request (Repeater):**
```
GET /product?productId=1 HTTP/1.1
Host: <lab-id>.web-security-academy.net
Referer: https://<unique-subdomain>.oastify.com
```

**Collaborator interaction log:**
```
Type: DNS  — A record lookup for <unique-subdomain>.oastify.com
Type: HTTP — GET / request received from the lab's server IP
```

The presence of these interactions confirms the server issued an outbound request based on attacker-controlled input, solving the lab.

## 5. Root Cause Analysis

**Attacker perspective:** The analytics component blindly trusts the client-supplied `Referer` header as a legitimate URL worth visiting, with no validation of its destination and no restriction on what hosts it's allowed to contact. Injecting a Collaborator domain causes the server to resolve and connect to attacker infrastructure — proof that arbitrary outbound requests can be triggered purely through a header value, something rarely scrutinized as closely as query/body parameters.

**Defender perspective:** `Referer`-driven fetches are a commonly overlooked SSRF surface precisely because they aren't part of the application's "normal" request/response flow from the developer's point of view — they're a side effect of an auxiliary analytics feature. The one-way nature (no reflected response) likely contributed to this being deployed without the same scrutiny as user-facing input fields.

**Note on DNS-only interactions:** It's common to see a DNS lookup for the Collaborator domain without a following HTTP request — this usually indicates the DNS resolution succeeded, but outbound HTTP/HTTPS was blocked at the network layer. A DNS-only interaction is still sufficient to confirm the SSRF primitive exists.

## 6. Impact

On its own, blind SSRF cannot be used to directly read back data from internal systems. However, it still provides a powerful primitive: it can be used to perform internal network/port scanning (via differing response-time or interaction patterns), and — as shown in Lab 07 — chained with other vulnerabilities on reachable internal hosts to achieve full remote code execution and data exfiltration via OAST channels.

## 7. Remediation

- Do not perform server-side fetches based on client-controlled headers (`Referer`, `X-Forwarded-*`, etc.) without validating the destination against a strict allowlist.
- Where auxiliary functionality (analytics, link previews, webhooks) genuinely requires fetching external URLs, isolate that fetching component on a separate, heavily firewalled host or service account with no access to internal infrastructure.
- Apply egress filtering so that even a successfully triggered blind SSRF cannot reach internal-only services or make arbitrary outbound connections.

## 8. References

- [PortSwigger — Blind SSRF](https://portswigger.net/web-security/ssrf/blind)
- [PortSwigger — Blind SSRF with out-of-band detection](https://portswigger.net/web-security/ssrf/blind)
- [Burp Collaborator documentation](https://portswigger.net/burp/documentation/collaborator)
- [CWE-918](https://cwe.mitre.org/data/definitions/918.html)