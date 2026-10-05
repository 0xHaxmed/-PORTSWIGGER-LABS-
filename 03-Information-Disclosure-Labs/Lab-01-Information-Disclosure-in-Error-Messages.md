# Lab 01 — Information Disclosure in Error Messages

[![Category](https://img.shields.io/badge/Category-Information%20Disclosure-blue)]()
[![Type](https://img.shields.io/badge/Type-Information%20Disclosure%20—%20Verbose%20Error%20Messages-orange)]()
[![Severity](https://img.shields.io/badge/Severity-Low-critical)]()
[![Source](https://img.shields.io/badge/Source-PortSwigger%20Web%20Security%20Academy-blue)](https://portswigger.net/web-security/information-disclosure)

> Part of the [Information Disclosure Labs](./README.md) series — PortSwigger Web Security Academy.

---

## 1. Overview

| Field | Detail |
|---|---|
| **Lab Name** | Information Disclosure in Error Messages |
| **Vulnerability Class** | Information Disclosure — Verbose Error Messages |
| **Weakness (CWE)** | CWE-209: Generation of Error Message Containing Sensitive Information |
| **Severity** | Low |
| **Objective** | Trigger a verbose stack trace to identify the exact version of a vulnerable third-party framework in use. |

## 2. Summary

The application does not handle unexpected input types gracefully. Supplying a non-integer value to a parameter that the back end expects to parse as an integer triggers an unhandled exception, and the full stack trace — including the framework name and version — is returned directly in the HTTP response.

## 3. Proof of Concept (PoC)

**Step 1 — Identify the target parameter:**  
Browse to any product page and capture the request in Burp Suite. Note the `productId` parameter:

```http
GET /product?productId=1 HTTP/2
```

**Step 2 — Send to Repeater:**  
Forward the request to Burp Repeater for manipulation.

**Step 3 — Break the expected data type:**  
Replace the numeric value with a non-integer string to force a type-casting failure:

```http
GET /product?productId="invalid" HTTP/2
```

**Step 4 — Capture the disclosure:**  
Send the request. The server returns `500 Internal Server Error` along with a complete Java stack trace, ending in the framework signature.


## 4. Evidence (Request / Response)

```http
GET /product?productId="invalid" HTTP/2
Host: <lab-id>.web-security-academy.net

HTTP/2 500 Internal Server Error
Content-Length: 1682

Internal Server Error: java.lang.NumberFormatException: For input string: "invalid"
    at java.base/java.lang.NumberFormatException.forInputString(NumberFormatException.java:67)
    at java.base/java.lang.Integer.parseInt(Integer.java:647)
    ...
Apache Struts 2 2.3.31
```

## 5. Root Cause Analysis

The parameter-parsing logic (`Integer.parseInt()`) is not wrapped in proper exception handling, and the production environment has verbose/debug error reporting enabled instead of generic error pages. The uncaught exception's full stack trace — including internal class names, file paths, and the exact framework version — is serialized directly into the client-facing response.

### Attacker vs. Defender Perspective

- **Attacker view:** Fuzz parameters with unexpected data types/characters and study verbose error responses for framework names, versions, and internal paths.
- **Defender view:** Return generic error pages in production; never let raw exceptions or stack traces reach the client.

## 6. Impact

On its own, a leaked framework version is low severity. However, it is a critical reconnaissance primitive: an attacker can cross-reference the disclosed version (e.g. Apache Struts 2.3.31) against public CVE databases to find pre-built, high-severity exploits — in this specific case, versions in this range are associated with critical Remote Code Execution vulnerabilities (e.g. CVE-2017-5638), potentially leading to full server compromise with a single follow-up request.

## 7. Remediation

- Disable verbose/debug error reporting in production; return generic, static error pages (e.g. a plain `500 Internal Server Error` with no implementation details) to the client.
- Wrap all user-input parsing and type-conversion logic in explicit try/catch blocks, and validate input types/formats before they reach business logic.
- Log full stack traces only to a secured, internal logging system — never to the HTTP response body.
- Keep all third-party frameworks and libraries patched and track their disclosed CVEs as part of routine dependency management.

## 8. References

- [PortSwigger — Information Disclosure](https://portswigger.net/web-security/information-disclosure)
- [OWASP Top 10 2021 — A01/A05: Broken Access Control / Security Misconfiguration](https://owasp.org/Top10/)
- [CWE-209: Generation of Error Message Containing Sensitive Information](https://cwe.mitre.org/data/definitions/209.html)

---

*Writeup produced for educational and authorized lab purposes only (PortSwigger Web Security Academy environment).*
