# Server-Side Request Forgery (SSRF) — Comprehensive Overview

## 1. What is SSRF?
Server-Side Request Forgery (SSRF) is a vulnerability where an attacker coerces a server-side application to issue unintended HTTP (or other protocol) requests to an arbitrary destination. 

In a typical SSRF attack, the attacker can cause the server to:
- Connect to internal-only infrastructure (e.g. `192.168.x.x`, `10.x.x.x`).
- Access loopback services on the local machine (`127.0.0.1`, `localhost`).
- Query Cloud Instance Metadata Services (AWS EC2 IMDS at `169.254.169.254`, GCP, Azure) to steal temporary cloud IAM credentials.
- Pivot to execute exploits (like Shellshock) on unpatched internal microservices.

In the OWASP Top 10, SSRF is ranked as **A10:2021 — Server-Side Request Forgery** and mapped to **CWE-918**.

## 2. Common SSRF Defense Bypasses

| Defense Mechanism | Flaw | Attacker Bypass Technique |
|---|---|---|
| **Blacklisting 127.0.0.1 / localhost** | Relying on literal string matching. | Alternative IP notations: `127.1`, `2130706433` (decimal), `0x7f000001` (hex), `017700000001` (octal), or DNS rebinding / local domain mappings (`spoofed.burpcollaborator.net`). |
| **Whitelisting Expected Domains** | Weak regex / string-contains matching. | Subdomain manipulation, URL parser discrepancy: `user:pass@expected.com#@attacker.com`, or leveraging Open Redirects on the whitelisted domain. |
| **Input Validation on URL Path** | Validating before decoding. | Double URL-encoding `%25%36%31...` or URL path traversal `..%2f`. |

## 3. Blind SSRF vs. In-band SSRF
- **In-band SSRF:** The response from the target server is reflected back into the attacker's HTTP response (e.g., viewing admin panel HTML).
- **Blind SSRF:** The application issues the outbound request (e.g., webhook notifications, analytics logging via `Referer`), but does not return the response body. Confirmed using Out-of-Band (OAST) tools like Burp Collaborator.
