# Server-Side Request Forgery (SSRF) — Root Cause Analysis

## 1. Executive Summary
SSRF arises when an application receives a full or partial URL from a client and uses a server-side HTTP client (e.g., `curl`, `requests`, `HttpClient`, `fetch`) to retrieve the resource **without strictly validating the destination IP address against private/loopback network ranges at the socket resolution level**.

## 2. The Core Root Causes

### 1. Inherent Trust in Server's Network Location (Network Perimeter Fallacy)
Internal microservices and admin interfaces often omit authentication under the false assumption that "if a request reaches us, it came from inside our private network." SSRF weaponizes the vulnerable web server as an authenticated proxy inside the perimeter.

### 2. URL Parser Differentials
Different components parse URLs differently:
- The validating component parses `http://localhost#@stock.weliketoshop.net/` and believes the hostname is `stock.weliketoshop.net`.
- The HTTP client treats `#` as a fragment delimiter, cuts off everything following it, and makes a connection directly to `localhost`.

### 3. Blindly Following HTTP Redirects
An application may enforce strict domain whitelisting on the initial URL. However, if the server-side HTTP client automatically follows HTTP 301/302 redirects, an attacker can supply an allowed endpoint that hosts an Open Redirect, pivoting the connection to internal resources.

### 4. DNS Rebinding
The server validates that `malicious.com` resolves to a public IP `198.51.100.1` (check passed). When the HTTP client fetches the URL milliseconds later, the attacker's DNS server answers with `127.0.0.1` (Time-of-Check Time-of-Use race condition).
