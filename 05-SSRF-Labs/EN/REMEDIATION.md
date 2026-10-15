# Server-Side Request Forgery (SSRF) — Prevention & Remediation Guide

## 1. Architectural Defenses

1. **Network Layer Segmentation (Egress Firewalls):**
   Block web applications from initiating outbound connections to private IP ranges (`10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`, `127.0.0.0/8`).
2. **Cloud IMDS Hardening:**
   Enforce **AWS IMDSv2** (requiring session tokens with `X-aws-ec2-metadata-token` and setting hop limit to 1). This completely neutralizes classic SSRF credential theft.
3. **Disable Following Redirects:**
   Configure server-side HTTP clients to never automatically follow HTTP 3xx redirects.

## 2. Application-Level Secure Validation Pattern
A secure URL fetching routine must:
1. Parse the URL and validate the scheme (`http` or `https` only).
2. Resolve the domain name to its IP address via DNS.
3. Verify that the resolved IP address is NOT in a loopback, private, multicast, or link-local range.
4. Pin the connection to that specific resolved IP address to prevent DNS rebinding.

### Python Example:
```python
import ipaddress, socket, requests
from urllib.parse import urlparse

def safe_fetch(url):
    parsed = urlparse(url)
    if parsed.scheme not in ('http', 'https'):
        raise ValueError("Invalid scheme")

    # Resolve IP
    ip = socket.gethostbyname(parsed.hostname)
    ip_obj = ipaddress.ip_address(ip)

    # Reject private, loopback, link-local, and reserved ranges
    if ip_obj.is_private or ip_obj.is_loopback or ip_obj.is_link_local:
        raise ValueError("SSRF protection: destination IP is restricted!")

    # Fetch with redirects disabled
    return requests.get(url, allow_redirects=False, timeout=5)
```
