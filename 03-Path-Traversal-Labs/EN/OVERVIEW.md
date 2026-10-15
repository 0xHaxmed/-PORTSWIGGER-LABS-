# Path Traversal & File Inclusion — Comprehensive Overview

## 1. What is Path Traversal (Directory Traversal)?
Path traversal (also called Directory Traversal or Dot-Dot-Slash attack) is a vulnerability that enables an attacker to read arbitrary files on the server that runs an application. This might include:
- Application code and data.
- Credentials for back-end systems (databases, API keys).
- Sensitive OS system files (e.g., `/etc/passwd` on Linux, `C:\Windows\win.ini` on Windows).

In the OWASP Top 10, Path Traversal is categorized under **A01:2021 — Broken Access Control** and mapped to **CWE-22 / CWE-23**.

## 2. Path Traversal vs. File Inclusion (LFI / RFI)

| Vulnerability | Mechanics | Maximum Impact |
|---|---|---|
| **Path Traversal (File Read)** | Accessing static files outside the intended root using `../`. | Arbitrary file read (source code, configs, system files). |
| **Local File Inclusion (LFI)** | Passing a local path to an execution directive like `include()` or `require()`. | Remote Code Execution (via log poisoning, session poisoning, php://wrappers). |
| **Remote File Inclusion (RFI)** | Instructing the server to fetch and execute a remote script hosted on attacker infrastructure. | Immediate Remote Code Execution (RCE). |

## 3. Common Filter Bypasses
1. **Absolute Paths:** When relative `../` is stripped, `/etc/passwd` directly bypasses weak path-joining functions.
2. **Non-Recursive Stripping:** `....//....//etc/passwd` leaves `../../etc/passwd` after a single-pass `replace("../", "")`.
3. **Superfluous (Double) URL Encoding:** `%252e%252e%252f` slips past edge filters and decodes twice at the application layer.
4. **Prefix Validation:** `/var/www/images/../../../etc/passwd` satisfies `starts_with("/var/www/images")`.
5. **Null Byte Injection:** `../../../etc/passwd%00.jpg` terminates C-level file APIs, satisfying file extension validation.
