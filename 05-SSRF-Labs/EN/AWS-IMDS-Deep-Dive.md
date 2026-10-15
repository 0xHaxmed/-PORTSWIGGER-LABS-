# AWS Cloud SSRF Deep Dive — EC2 Instance Metadata Service (IMDS) & IAM Credential Theft

A plain-language-first, then technical, guide to why SSRF becomes a **critical, credential-stealing** vulnerability in AWS environments specifically — covering the Instance Metadata Service (IMDS), IAM roles, IMDSv1 vs IMDSv2, safe lab scenarios, detection/mitigation, and real-world case studies. This complements the PortSwigger/TryHackMe lab writeups in this folder: those labs teach the *mechanics* of SSRF, this guide explains what happens when a vulnerable server is *also* sitting inside AWS.

> **Scope note:** every command, script, and "lab" in this document is designed to run against a local mock service or a learner-owned AWS sandbox account — never against production or third-party infrastructure.

---

## 1. Why SSRF Becomes More Dangerous Inside AWS

**In simple terms:** normally, SSRF lets an attacker make the *server* fetch a URL it shouldn't. That's already bad — it can leak internal pages. But on AWS, there's one extra-dangerous URL every EC2 server can reach: a special local address that hands out temporary login keys for the server's AWS account. If a web app on that server has an SSRF bug, an attacker can potentially ask the server to fetch that address and hand those keys straight back to them — turning a "the app leaked a page" bug into "the attacker can now act as the server inside AWS."

**Technically:** every AWS EC2 instance can reach `http://169.254.169.254` — the **Instance Metadata Service (IMDS)**. This is a link-local address (defined by RFC 3927, range `169.254.0.0/16`) that only the instance itself can route to; it is intercepted at the hypervisor/network level and never leaves the physical host. If the instance has an **IAM Role** attached (common, since it's how apps authenticate to S3, DynamoDB, etc. without hardcoding keys), IMDS will hand back that role's live, temporary AWS credentials to *any process on the instance that asks* — including, if SSRF is present, a web application that an attacker has tricked into asking on their behalf.

---

## 2. SSRF Fundamentals (Quick Recap)

This section is a short refresher — the full breakdown (in-band vs blind vs OOB, finding SSRF, filter bypasses) lives in [`METHODOLOGY.md`](./METHODOLOGY.md) in this same folder.

- **SSRF** happens when a server builds an outbound HTTP request using attacker-influenced input (a URL, hostname, or path) with insufficient validation.
- Common entry points: URL preview/"unfurl" features, image proxies/resizers, PDF generators (which often use a full browser engine like headless Chromium internally), webhook configuration, and "import from URL" features.
- **In-band SSRF**: the fetched response is reflected back to the attacker.
- **Blind SSRF**: the request happens, but nothing is reflected — confirmation requires out-of-band (OAST) techniques.
- **Proving the bug exists is not the same as proving impact.** Triggering a DNS lookup to a Collaborator domain proves the server makes attacker-controlled requests; it does not by itself prove an attacker can read secrets or pivot further. The IMDS scenario below is exactly the gap between those two things — and AWS metadata is often how a "just a bug" SSRF becomes a full account compromise.

---

## 3. AWS Architecture: EC2, IMDS, and IAM Roles

### 3.1 What IMDS Is, in Plain Terms

Think of IMDS as a notepad the cloud provider tapes to the back of every virtual server, readable only by that server, containing facts about itself (its ID, its network info, its startup script) — and, if the server has been assigned an identity ("IAM Role"), a drawer with a temporary keycard inside that notepad.

### 3.2 Technical Architecture

```
┌────────────────────────────────────────────────────────┐
│                    AWS EC2 Instance                     │
│   ┌─────────────────────┐      ┌─────────────────────┐  │
│   │ Application/Workload │      │  AWS CLI / SDK      │  │
│   └──────────┬───────────┘      └──────────┬──────────┘  │
│              │ HTTP GET/PUT                │ fetches credentials │
│              ▼                             ▼              │
│        [ Kernel network stack / vNIC ]                   │
└───────────────────────┬──────────────────────────────────┘
                         │ link-local traffic → 169.254.169.254:80
                         ▼
┌────────────────────────────────────────────────────────┐
│          AWS Hypervisor (e.g. Nitro)                     │
│      ┌──────────────────────────────────────┐            │
│      │   Instance Metadata Service (IMDS)   │            │
│      └────────────────┬──────────────────────┘            │
└───────────────────────┼──────────────────────────────────┘
                         │ AssumeRole via the instance profile
                         ▼
              ┌─────────────────────────┐
              │  AWS Security Token      │
              │  Service (STS)           │
              └─────────────────────────┘
```

The `169.254.0.0/16` range is explicitly non-routable (RFC 3927) — routers will never forward these packets across a network boundary, which is precisely why AWS can safely use it: it's only ever reachable from inside the instance that owns it.

### 3.3 From IAM Role to Temporary Credentials

1. An admin creates an **IAM Role** with a policy describing exactly what it's allowed to do (e.g. "read objects from this one S3 bucket").
2. The role is wrapped in an **Instance Profile** and attached to an EC2 instance.
3. Any process on that instance can request:
   ```
   GET http://169.254.169.254/latest/meta-data/iam/security-credentials/<ROLE_NAME>
   ```
4. The response is JSON containing a **temporary** `AccessKeyId` (prefixed `ASIA...`, distinguishing it from a permanent user key, which is prefixed `AKIA...`), a `SecretAccessKey`, a session `Token`, and an `Expiration` timestamp.
5. The official AWS SDKs do this automatically in the background and refresh the credentials before they expire — which is exactly why so many apps are built assuming this endpoint is always safe to call.

### 3.4 Metadata vs. Credentials — Two Different Risk Levels

| Category | Example paths | What it reveals | Risk if leaked |
|---|---|---|---|
| **Operational metadata** | `/latest/meta-data/ami-id`, `/latest/meta-data/local-ipv4`, `/latest/meta-data/hostname`, `/latest/user-data` | Infrastructure IDs, internal IPs, and the instance's boot/startup script | Reveals network layout; `user-data` very often contains hardcoded secrets engineers embedded in startup scripts by mistake |
| **Authentication credentials** | `/latest/meta-data/iam/security-credentials/<role>` | Live, temporary AWS access keys for the attached IAM role | Lets an attacker authenticate to AWS *as that role* and perform whatever actions its policy allows |

### 3.5 "I Have the Keys" ≠ "I Own the AWS Account"

This is the single most important nuance to understand, and the one most often lost in simplified writeups:

- **Permission scope**: the stolen credentials can only do what the attached role's IAM policy allows. A role scoped to `s3:GetObject` on one specific bucket cannot create users, change network settings, or read other buckets.
- **Trust relationships**: the role's trust policy restricts who/what can assume it (usually just `ec2.amazonaws.com`) — this doesn't limit the attacker once they hold valid session credentials, but it does mean the attacker is bound by the *same* scope the legitimate application has, not a separate privilege.
- **Permissions boundaries / Service Control Policies (SCPs)**: an AWS Organization can enforce account-wide ceilings that apply even to an over-privileged role, capping worst-case impact regardless of what a single role's policy says.

This is why least-privilege IAM design is the single biggest lever an organization has over how bad an SSRF-to-IMDS chain actually turns out to be.

---

## 4. IMDSv1 vs. IMDSv2

AWS introduced **IMDSv2** in **November 2019**, specifically as a response to the SSRF/open-proxy pattern described above (and, notably, shortly after the Capital One breach — see Section 8).

### 4.1 Comparison Table

| Aspect | IMDSv1 (classic, unrestricted) | IMDSv2 (session-oriented) |
|---|---|---|
| Request model | Simple, stateless GET/HEAD | Session-based: first get a token, then use it |
| HTTP methods | `GET` / `HEAD` only | `PUT` to fetch a token, then `GET`/`HEAD` with that token |
| Required headers | None | `X-aws-ec2-metadata-token-ttl-seconds` on the `PUT`; `X-aws-ec2-metadata-token` on every subsequent request |
| Token lifetime | N/A | 1 second minimum, 21,600 seconds (6 hours) maximum, set by the caller |
| Proxy/forwarded-header protection | None | Any token `PUT` request carrying an `X-Forwarded-For` header is rejected with `400` |
| Hop limit | Effectively unrestricted | `HttpPutResponseHopLimit` — commonly defaulted to `1` on modern instances |
| Enforcement | Was enabled alongside IMDSv2 for backward compatibility for years | AWS has progressively pushed new AMIs/instances toward requiring it (`HttpTokens=required`) |

### 4.2 Why Each IMDSv2 Control Actually Stops Basic SSRF

```
Naive SSRF payload (plain GET)            ──► IMDSv2 ──► 401 Unauthorized
SSRF via a misconfigured reverse proxy
  (request carries X-Forwarded-For)       ──► IMDSv2 ──► 400 Bad Request
Containerized app requesting the token
  (TTL=1 hop limit, bridge network)       ──► packet dies at the bridge → never reaches IMDS
```

- **The `PUT` requirement** quietly defeats the overwhelming majority of real-world SSRF bugs, because most vulnerable "fetch this URL" features only ever issue `GET` (or sometimes `POST`) requests — they have no code path that performs a `PUT`.
- **The required custom header** defeats SSRF bugs where the attacker only controls the *URL*, not the request's headers — which describes the vast majority of URL-preview, PDF-generation, and webhook-style vulnerabilities.
- **Rejecting `X-Forwarded-For`** specifically targets the exact failure mode behind the Capital One breach: a misconfigured reverse proxy/WAF relaying a crafted request toward IMDS.
- **The hop limit** is a quiet but effective control for containerized workloads: Docker/Kubernetes bridge networking consumes one hop just getting the packet out of the container, so with `HttpPutResponseHopLimit=1`, a containerized app can't reach IMDS at all unless it shares the host's network namespace.

### 4.3 IMDSv2 Is a Mitigation, Not a Cure

IMDSv2 only protects *this one endpoint*. It does nothing about:

- **Full HTTP control**: if the SSRF-vulnerable feature lets the attacker choose the method and set arbitrary headers (some webhook/integration builders do exactly this — see the Typebot case study below), the attacker can simply perform the legitimate `PUT`-then-`GET` dance themselves.
- **Raw/transparent TCP proxies**: if the vulnerable code opens a raw socket rather than going through a well-behaved HTTP client, there's no `X-Forwarded-For` header to reject in the first place.
- **Every other internal target**: IMDSv2 says nothing about internal databases, Redis instances, admin panels, or other microservices reachable from the same server — SSRF against those is exactly as exploitable as ever.

---

## 5. The Attack Chain, Stage by Stage

```
1. Find an SSRF entry point
        │
2. Determine what the vulnerable client can actually do
   (methods / headers / protocols / redirect-following)
        │
3. Classify the response: in-band, blind, or out-of-band
        │
4. Test reachability of internal ranges (RFC 1918) and 169.254.169.254
        │
5. Determine whether IMDS is enabled, and which version is enforced
   ├── IMDSv1 reachable  → plain GET is enough
   └── IMDSv2 enforced   → need PUT + custom header capability
        │
6. (Authorized testing only) Assess the attached role's actual permissions and impact
```

| Stage | Preconditions | Evidence needed | What blocks it | Common false assumption |
|---|---|---|---|---|
| 1. Entry point | A parameter/field that influences a server-side request | The app visibly fetches/renders remote content, or behaves differently for different URLs | Strict allowlisting, no URL-shaped input anywhere | "It ends in `.jpg` so it must be safe" — the path after `?` can still be redirected |
| 2. Client capability | Multiple test requests | Behavior differences for different methods/headers/protocols | The library hardcodes `GET` with no header passthrough | Assuming every SSRF can set arbitrary headers or methods — most can't |
| 3. Response visibility | — | Reflected content (in-band) or a Collaborator/listener hit (blind) | Fixed generic responses, async processing | Treating "no visible output" as "not exploitable" |
| 4. Internal reachability | Server has a route to `169.254.169.254` or RFC 1918 ranges | Distinct status codes/timing for reachable vs unreachable hosts | Local `iptables` rules, egress proxy, network ACLs | Assuming cloud = automatically isolated from its own metadata endpoint |
| 5. IMDS version | — | `200` + directory listing (v1) vs `401` (v2 enforced) | `HttpTokens=required` with no header-injection capability in the SSRF primitive | Assuming IMDSv2 alone closes the whole class of risk |
| 6. Impact assessment | **Explicit written authorization**, a lab/sandbox account | `aws sts get-caller-identity`, policy review | Least-privilege roles, SCPs, permission boundaries | Assuming "I got credentials" automatically means "I own the account" |

---

## 6. Safe, Self-Contained Lab Scenarios

All three scenarios below run entirely on infrastructure the learner owns — local Docker containers or a dedicated AWS sandbox account — with no real credentials and no third-party targets involved.

### Scenario A — Basic SSRF Against a Mock Internal Service

**Setup:** a small Node.js app with a vulnerable "fetch preview" endpoint, plus a second container simulating a protected internal admin API, both on an isolated Docker bridge network.

```yaml
# docker-compose.yml
version: '3.8'
services:
  vulnerable-app:
    build: { context: ., dockerfile: Dockerfile.app }
    ports: ["8080:8080"]
    networks: [internal-net]

  internal-admin-service:
    image: python:3.9-slim
    command: >
      python -c "import http.server, socketserver;
      class H(http.server.SimpleHTTPRequestHandler):
          def do_GET(self):
              self.send_response(200); self.send_header('Content-type','application/json'); self.end_headers()
              self.wfile.write(b'{\"status\":\"ADMIN_ACCESS\",\"flag\":\"LAB_SUCCESS_FLAG_1\"}')
      socketserver.TCPServer(('0.0.0.0', 8081), H).serve_forever()"
    networks: [internal-net]

networks:
  internal-net: { driver: bridge }
```

```javascript
// app.js — deliberately vulnerable endpoint
const express = require('express');
const axios = require('axios');
const app = express();

app.get('/fetch-preview', async (req, res) => {
  const targetUrl = req.query.url;
  if (!targetUrl) return res.status(400).send('URL parameter is required');
  try {
    const response = await axios.get(targetUrl, { timeout: 3000 });
    res.send(response.data);
  } catch (err) {
    res.status(500).send(`Error fetching content: ${err.message}`);
  }
});

app.listen(8080, () => console.log('App running on port 8080'));
```

**Test:**
```
GET /fetch-preview?url=http://example.com                     → normal external content
GET /fetch-preview?url=http://internal-admin-service:8081/     → {"status":"ADMIN_ACCESS","flag":"LAB_SUCCESS_FLAG_1"}
```

**Lesson:** with no destination validation, the "fetch preview" feature is a free pass into a service that was never meant to be reachable from outside the Docker network.

### Scenario B — IMDSv1 vs. IMDSv2 Behavior, Simulated Locally

**Setup:** a small Python mock server that faithfully reproduces IMDSv2's token requirement, required headers, and `X-Forwarded-For` rejection — so the token mechanics can be studied with zero AWS cost or risk.

```python
# imds_mock.py
import http.server, socketserver, uuid

ACTIVE_TOKENS = set()

class IMDSHandler(http.server.BaseHTTPRequestHandler):
    def do_PUT(self):
        if self.path == '/latest/api/token':
            ttl = self.headers.get('X-aws-ec2-metadata-token-ttl-seconds')
            if not ttl:
                self.send_response(400); self.end_headers()
                self.wfile.write(b"Missing required TTL header"); return
            if 'X-Forwarded-For' in self.headers:
                self.send_response(400); self.end_headers()
                self.wfile.write(b"Proxy forwarded requests are rejected"); return
            token = str(uuid.uuid4()); ACTIVE_TOKENS.add(token)
            self.send_response(200); self.end_headers(); self.wfile.write(token.encode())
        else:
            self.send_response(404); self.end_headers()

    def do_GET(self):
        token = self.headers.get('X-aws-ec2-metadata-token')
        if not token or token not in ACTIVE_TOKENS:
            self.send_response(401); self.end_headers()
            self.wfile.write(b"Unauthorized: Valid IMDSv2 token required"); return
        if self.path == '/latest/meta-data/iam/security-credentials/':
            self.send_response(200); self.end_headers(); self.wfile.write(b"LabDemoRole")
        elif self.path == '/latest/meta-data/iam/security-credentials/LabDemoRole':
            self.send_response(200); self.send_header('Content-Type', 'application/json'); self.end_headers()
            self.wfile.write(b'{"AccessKeyId":"ASIA-SYNTHETIC-KEY","SecretAccessKey":"SYNTHETIC-SECRET","Token":"SYNTHETIC-TOKEN"}')
        else:
            self.send_response(404); self.end_headers()

if __name__ == '__main__':
    print("Starting IMDSv2 mock on port 9090...")
    socketserver.TCPServer(('127.0.0.1', 9090), IMDSHandler).serve_forever()
```

**Test sequence:**
```bash
# 1. Naive GET — fails, exactly like real IMDSv2
curl -i -X GET http://127.0.0.1:9090/latest/meta-data/iam/security-credentials/
# → 401 Unauthorized

# 2. Token request through a "proxy" — rejected
curl -i -X PUT http://127.0.0.1:9090/latest/api/token \
  -H "X-aws-ec2-metadata-token-ttl-seconds: 60" \
  -H "X-Forwarded-For: 203.0.113.195"
# → 400 Proxy forwarded requests are rejected

# 3. The legitimate two-step flow — succeeds
TOKEN=$(curl -s -X PUT "http://127.0.0.1:9090/latest/api/token" \
  -H "X-aws-ec2-metadata-token-ttl-seconds: 60")
curl -i -s -H "X-aws-ec2-metadata-token: $TOKEN" \
  http://127.0.0.1:9090/latest/meta-data/iam/security-credentials/LabDemoRole
# → synthetic JSON credentials
```

**Lesson:** this makes concrete exactly which capabilities (method control + header injection) an SSRF primitive needs before IMDSv2 stops being a meaningful barrier.

### Scenario C — Least Privilege in a Real AWS Sandbox Account

**Prerequisites:** a dedicated AWS sandbox/test account (never production), AWS CLI configured with admin rights *on that sandbox account only*.

```bash
# 1. A deliberately narrow policy — list access to one bucket only
cat << 'EOF' > /tmp/s3-read-policy.json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Sid": "AllowSpecificBucketList",
    "Effect": "Allow",
    "Action": ["s3:ListBucket"],
    "Resource": "arn:aws:s3:::my-unique-lab-test-bucket-2026"
  }]
}
EOF
aws iam create-policy --policy-name RestrictedS3LabPolicy --policy-document file:///tmp/s3-read-policy.json

# 2. A role trusted only by EC2, carrying that narrow policy
cat << 'EOF' > /tmp/trust-policy.json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Principal": {"Service": "ec2.amazonaws.com"},
    "Action": "sts:AssumeRole"
  }]
}
EOF
aws iam create-role --role-name MinimalEC2Role --assume-role-policy-document file:///tmp/trust-policy.json
aws iam attach-role-policy --role-name MinimalEC2Role --policy-arn arn:aws:iam::<ACCOUNT_ID>:policy/RestrictedS3LabPolicy
aws iam create-instance-profile --instance-profile-name MinimalEC2Profile
aws iam add-role-to-instance-profile --instance-profile-name MinimalEC2Profile --role-name MinimalEC2Role

# 3. Launch with IMDSv2 enforced from the start
aws ec2 run-instances \
  --image-id ami-0c55b159cbfafe1f0 --instance-type t3.micro \
  --iam-instance-profile Name=MinimalEC2Profile \
  --metadata-options "HttpTokens=required,HttpPutResponseHopLimit=1,HttpEndpoint=enabled"
```

**Verification (via SSM Session Manager, not SSH with long-lived keys):**
```bash
aws sts get-caller-identity              # confirms the role's identity
aws s3 ls s3://my-unique-lab-test-bucket-2026   # succeeds — in scope
aws ec2 describe-instances                # fails — AccessDenied, out of scope
aws iam list-users                        # fails — AccessDenied, out of scope
```

**Lesson:** this is the real-world proof that a tightly-scoped role turns "credentials were stolen" into "an attacker can list one bucket" rather than "an attacker owns the AWS account."

**Cleanup:**
```bash
aws ec2 terminate-instances --instance-ids <INSTANCE_ID>
aws iam remove-role-from-instance-profile --instance-profile-name MinimalEC2Profile --role-name MinimalEC2Role
aws iam delete-instance-profile --instance-profile-name MinimalEC2Profile
aws iam detach-role-policy --role-name MinimalEC2Role --policy-arn arn:aws:iam::<ACCOUNT_ID>:policy/RestrictedS3LabPolicy
aws iam delete-role --role-name MinimalEC2Role
aws iam delete-policy --policy-arn arn:aws:iam::<ACCOUNT_ID>:policy/RestrictedS3LabPolicy
```

---

## 7. Detection, Prevention, and Mitigation

### 7.1 Why Blacklisting URLs Never Fully Works

The address `169.254.169.254` alone has many equivalent forms a naive string-match filter won't catch:

| Form | Example |
|---|---|
| Decimal/integer | `http://2852039166/` |
| Hexadecimal | `http://0xa9fea9fe/` |
| Octal | `http://0251.0376.0251.0376/` |
| Mixed | `http://169.254.0xa9fe/` |

Parser inconsistencies compound the problem: a validating component and the actual requesting component can disagree about where a URL like `http://user:pass@169.254.169.254#@attacker.com/` actually points — one sees `attacker.com`, the other connects to the metadata service. (This exact class of bug is explored in depth in `Lab-06-Whitelist-Filter-Bypass.md` in this folder.)

### 7.2 DNS Rebinding

**In simple terms:** the filter checks where a domain name points *once*, approves it, but then the actual connection re-checks the domain a moment later — and the attacker has swapped the answer in between.

```
Validation moment:  app → DNS query → attacker's DNS server → returns 203.0.113.10 (public, approved)
Connection moment:  app → DNS query → attacker's DNS server → returns 169.254.169.254 (now switched)
```

The attacker's DNS server sets a `TTL=0` record so the second lookup is free to return something different. **Fix:** resolve the hostname once, validate that IP, and connect directly to the validated IP (never re-resolve), or enable DNS pinning in the HTTP client.

### 7.3 Redirect Handling

An app may correctly block direct requests to internal addresses but still allow a request to `http://attacker.com/redirect`, which replies:
```
HTTP/1.1 302 Found
Location: http://169.254.169.254/latest/meta-data/
```
If the HTTP client follows redirects automatically without re-validating the new destination, the entire upfront check is bypassed. (This is the exact mechanism behind `Lab-04-Open-Redirect-Filter-Bypass.md` in this folder, applied here specifically against IMDS.)

### 7.4 A Safe Application-Level Pattern

```python
import ipaddress, socket, urllib.parse, requests

def is_safe_ip(ip_str):
    try:
        ip = ipaddress.ip_address(ip_str)
        return ip.is_global and not ip.is_link_local and not ip.is_private and not ip.is_loopback
    except ValueError:
        return False

def safe_fetch(url):
    parsed = urllib.parse.urlparse(url)
    if parsed.scheme not in ('http', 'https'):
        raise ValueError("Invalid protocol scheme")

    hostname = parsed.hostname
    for family, _, _, _, sockaddr in socket.getaddrinfo(hostname, None):
        if not is_safe_ip(sockaddr[0]):
            raise PermissionError(f"Access to restricted IP blocked: {sockaddr[0]}")

    # Never follow redirects blindly — re-validate every hop the same way
    return requests.get(url, allow_redirects=False, timeout=5).text
```

Key properties: validates the **resolved IP**, not the raw hostname string; rejects link-local/private/loopback ranges explicitly (not just one hardcoded address); and disables automatic redirect-following.

### 7.5 Enforcing IMDSv2 Account-Wide

**Service Control Policy (deny launching instances that don't require IMDSv2):**
```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Sid": "EnforceIMDSv2AcrossAccount",
    "Effect": "Deny",
    "Action": "ec2:RunInstances",
    "Resource": "arn:aws:ec2:*:*:instance/*",
    "Condition": { "StringNotEquals": { "ec2:MetadataHttpTokens": "required" } }
  }]
}
```

**Harden an existing instance:**
```bash
aws ec2 modify-instance-metadata-options \
  --instance-id <INSTANCE_ID> --http-tokens required \
  --http-put-response-hop-limit 1 --http-endpoint enabled
```

**Disable IMDS entirely** for instances that never need AWS credentials or metadata:
```bash
aws ec2 modify-instance-metadata-options --instance-id <INSTANCE_ID> --http-endpoint disabled
```

### 7.6 Monitoring

- **CloudWatch metrics**: `MetadataNoToken` (IMDSv1-style requests — a rising count flags legacy code or active exploitation attempts) vs `MetadataWithToken` (expected IMDSv2 usage).
- **CloudTrail**: watch for `sts:AssumeRole` activity and API calls made with `ASIA...` keys from unexpected source IPs or outside normal infrastructure ranges.

---

## 8. Real-World Case Studies

### 8.1 Capital One (2019)

- **Stack:** AWS EC2 behind a misconfigured open-source WAF acting as a reverse proxy.
- **Mechanism:** a WAF misconfiguration allowed a crafted request to be relayed toward `http://169.254.169.254/latest/meta-data/`.
- **Preconditions:** IMDSv1 was in use (no session token required), and the WAF's own IAM role was broadly over-privileged.
- **Verified impact:** temporary credentials for the WAF's role were used (`aws s3 sync`-style access) to read **over 700 S3 buckets**, exposing data on more than 100 million customers, and resulting in an **$80M regulatory fine**.
- **Remediation:** fixed the proxy misconfiguration, tightened IAM role scope. This incident is widely credited as a major driver behind AWS shipping IMDSv2's `X-Forwarded-For` rejection.
- **Source:** U.S. Department of Justice indictment; subsequent academic/industry analyses.
- **Relevance today:** this specific attack path does not work against an instance with IMDSv2 enforced and a properly scoped role.

### 8.2 GitLab Webhooks — CVE-2021-22214

- **Stack:** GitLab CE/EE, commonly self-hosted on AWS EC2.
- **Mechanism:** unauthenticated SSRF via the webhook/CI integration request path.
- **Preconditions:** "allow requests to the local network" enabled, and the hosting EC2 instance not enforcing IMDSv2.
- **Verified impact:** attackers could force the GitLab server to reach internal network addresses and the EC2 metadata endpoint, extracting credentials.
- **Remediation:** GitLab 13.12.2 / 13.11.5 enforced stricter internal-URL validation.
- **Source:** HackerOne report #1110131; NVD CVE-2021-22214.
- **Relevance today:** patched at the application layer; IMDSv2 serves as an independent, defense-in-depth layer against similar future bugs.

### 8.3 Typebot / Chatbot Webhook Builders — GHSA-8gq9-rw7v-3jpr (2024/2025 research)

- **Stack:** self-hosted chatbot-building platforms running on AWS EKS (Kubernetes).
- **Mechanism:** the webhook-builder feature let users choose **any HTTP method** (including `PUT`) and set **arbitrary custom headers** — exactly the two capabilities Section 4.3 flags as defeating IMDSv2.
- **Preconditions:** IMDSv2 enforced, but the container could reach the node's network path to IMDS.
- **Verified impact:** researchers issued the legitimate two-step flow themselves — a `PUT` to `/latest/api/token` with `X-aws-ec2-metadata-token-ttl-seconds: 21600` (using the decimal IP form `2852039166` to additionally dodge naive string filters), then a `GET` with the returned token — fully extracting the EKS worker node's IAM role credentials.
- **Remediation:** block outbound requests to link-local-range hosts regardless of header content, and restrict which headers a webhook feature is allowed to set (specifically disallow anything matching `X-aws-ec2-metadata-token*`).
- **Source:** GitHub Security Advisory GHSA-8gq9-rw7v-3jpr.
- **Relevance today:** a recent, concrete demonstration that **"IMDSv2 is enforced" is not sufficient** when the vulnerable feature itself grants full HTTP method and header control.

---

## 9. Penetration Testing Methodology for This Specific Scenario

### 9.1 Safe Testing Principles

- **Verify, don't exfiltrate:** confirm access with a lightweight, low-sensitivity path first (`/latest/meta-data/instance-id`, `/latest/meta-data/ami-id`) rather than immediately pulling and storing live credentials.
- **No lateral movement without written authorization:** never use recovered credentials to call other AWS services (S3, RDS, etc.) beyond what's explicitly in scope.
- **Redact secrets in reports:** mask `SecretAccessKey`/`Token` values (e.g. `ASIA********************`) in any evidence shared with a client.

### 9.2 Sample Vulnerability Report

> **Title:** High Severity: Server-Side Request Forgery (SSRF) Leading to IMDSv1 Information Disclosure
>
> **Executive Summary:** During an authorized assessment of the employee portal application, an SSRF vulnerability was identified in the resume-import preview feature. The flaw allows an unauthenticated... *(adjust per your actual access level)* ...attacker to force the server to issue requests to internal, network-isolated resources, including the AWS EC2 Instance Metadata Service (IMDSv1), exposing instance infrastructure details to unauthorized disclosure.
>
> **Technical Description:** The `/api/v1/import-resume` endpoint accepts a `remote_url` JSON field. The server fetches this URL via its HTTP client with no validation or filtering of reserved/link-local IP ranges. Because the instance runs with `HttpTokens=optional` (IMDSv1 reachable without a session token), the server's response reflected metadata content back to the client.
>
> **Preconditions:** a standard authenticated user account with access to the resume-import feature; ability to send `POST` requests to the API.
>
> **Reproduction Steps:**
> ```
> POST /api/v1/import-resume HTTP/1.1
> Host: target-lab.internal.domain
> Authorization: Bearer <TEST_TOKEN>
> Content-Type: application/json
>
> {"applicant_name": "Security Audit Test", "remote_url": "http://169.254.169.254/latest/meta-data/instance-id"}
> ```
> **Evidence:**
> ```
> HTTP/1.1 200 OK
> {"status": "success", "imported_content": "i-0abcd1234ef56789a"}
> ```
> As a precaution, the `iam/security-credentials/` path was **not** queried during testing to avoid unnecessarily handling live credentials.
>
> **Impact Analysis:** `CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:N/A:N` (7.5, High). Confirms the server can be forced to reach internal/link-local infrastructure; if the attached IAM role is over-privileged, this primitive could be escalated (with authorization) to full credential theft.
>
> **Remediation:**
> ```bash
> aws ec2 modify-instance-metadata-options --instance-id <ID> --http-tokens required --http-put-response-hop-limit 1
> ```
> Combined with resolved-IP allowlist validation at the application layer (see Section 7.4), rejecting RFC 1918 and RFC 3927 ranges.
>
> **References:** OWASP SSRF Prevention Cheat Sheet; AWS IMDSv2 documentation.

---

## 10. Advanced Topics

### 10.1 Containerized AWS Environments

- **ECS (Elastic Container Service):** tasks get credentials from a **different** link-local address, `169.254.170.2`, via the `AWS_CONTAINER_CREDENTIALS_RELATIVE_URI` environment variable — targeting `169.254.169.254` from inside an ECS task may hit the host's own metadata (if host networking is used) or simply fail; `169.254.170.2` is the relevant target for the **task's** own role.
- **EKS (Kubernetes):** modern practice uses **IRSA (IAM Roles for Service Accounts)** or **EKS Pod Identities**, injecting credentials via web-identity tokens so pods never need to call IMDS directly. However, if a pod can still reach the worker node's own IMDS endpoint, it may be able to steal the **node's** IAM role — a different, often more powerful, blast radius than the pod's own intended role (the Typebot case study in Section 8.3 is exactly this).

### 10.2 Metadata Service Comparison Across Cloud Providers

| Provider | Endpoint | Built-in protection |
|---|---|---|
| AWS (EC2) | `http://169.254.169.254` | IMDSv2: requires a `PUT`-issued session token and custom headers |
| Google Cloud | `http://metadata.google.internal` (or `169.254.169.254`) | Requires the header `Metadata-Flavor: Google` on every request — rejects requests without it by default |
| Microsoft Azure | `http://169.254.169.254/metadata/` | Requires the header `Metadata: true` on every request |

Historically, AWS's original (IMDSv1) design was the most "open by default" of the three — no required header at all — which is a large part of why it became the most heavily targeted in public research before IMDSv2 closed that specific gap.

---

## 11. Glossary

| Term | Meaning |
|---|---|
| **SSRF** | A vulnerability letting an attacker influence the destination of a server-generated request. |
| **IMDS** | Instance Metadata Service — a cloud provider's local endpoint for an instance to query its own configuration and (if applicable) credentials. |
| **Link-local address** | An IP range (`169.254.0.0/16` for IPv4) reserved for same-segment-only communication; never forwarded by routers. |
| **STS** | AWS Security Token Service — issues short-lived, scoped credentials. |
| **Hop limit** | The number of network hops a packet is allowed before being dropped; used here to fence off container traffic from the host's IMDS. |
| **DNS rebinding** | Manipulating DNS TTLs so a hostname resolves differently between a validation check and the actual connection. |
| **OOB (out-of-band)** | Confirming/exploiting a vulnerability via a side channel (e.g. DNS/HTTP to a Collaborator) rather than the application's own response. |

---

## 12. References

- [AWS — Configure the instance metadata service (IMDSv2)](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/configuring-instance-metadata-service.html)
- [AWS Security Blog — Add defense in depth against open firewalls, reverse proxies, and SSRF vulnerabilities with enhanced domain protections for IMDSv2](https://aws.amazon.com/blogs/security/)
- [AWS — Instance metadata categories](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html)
- [AWS — IAM roles for Amazon EC2](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_use_switch-role-ec2.html)
- [OWASP — Server-Side Request Forgery Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Server_Side_Request_Forgery_Prevention_Cheat_Sheet.html)
- [PortSwigger — Server-side request forgery (SSRF)](https://portswigger.net/web-security/ssrf)
- [RFC 3927 — Dynamic Configuration of IPv4 Link-Local Addresses](https://www.rfc-editor.org/rfc/rfc3927)
- U.S. Department of Justice — Capital One data breach indictment
- [NVD — CVE-2021-22214 (GitLab SSRF)](https://nvd.nist.gov/vuln/detail/CVE-2021-22214)
- GitHub Security Advisory — GHSA-8gq9-rw7v-3jpr (Typebot SSRF → EKS credential exposure)
- Datadog Security Labs — Securing the EC2 Instance Metadata Service

---

## Cross-Reference to This Repo

| Concept here | Related file in this folder |
|---|---|
| General SSRF confirmation & recon | [`METHODOLOGY.md`](./METHODOLOGY.md) |
| Blacklist/allowlist bypass mechanics (Section 7.1) | [`Lab-03-Blacklist-Filter-Bypass.md`](./Lab-03-Blacklist-Filter-Bypass.md), [`Lab-06-Whitelist-Filter-Bypass.md`](./Lab-06-Whitelist-Filter-Bypass.md) |
| Redirect-chaining bypass (Section 7.3) | [`Lab-04-Open-Redirect-Filter-Bypass.md`](./Lab-04-Open-Redirect-Filter-Bypass.md) |
| Blind SSRF confirmation via OOB | [`Lab-05-Blind-SSRF-OOB-Detection.md`](./Lab-05-Blind-SSRF-OOB-Detection.md) |
| Blind SSRF chained into RCE on an internal host | [`Lab-07-Blind-SSRF-Shellshock.md`](./Lab-07-Blind-SSRF-Shellshock.md) |
