# Path Traversal Labs — Index

Professional PoC writeups for PortSwigger Web Security Academy's Path Traversal labs (01–06).

## 🧠 Start Here

Before jumping into the lab writeups, read the **[Pentester Mindset — Path Traversal](./METHODOLOGY.md)** doc. It explains how to *think* about this vulnerability class — what makes a parameter suspicious, the standard bypass escalation order, and what to do once you get a file read — rather than just the specific payloads used in each lab.

## 📋 Lab Index

| # | Lab | Type | Severity | File |
|---|---|---|---|---|
| 01 | File Path Traversal, Simple Case | Path Traversal — No Sanitization | High | [Lab-01-File-Path-Traversal-Simple-Case.md](./Lab-01-File-Path-Traversal-Simple-Case.md) |
| 02 | Traversal Sequences Blocked with Absolute Path Bypass | Path Traversal — Blacklist Bypass (Absolute Path) | High | [Lab-02-Traversal-Sequences-Blocked-with-Absolute-Path-Bypass.md](./Lab-02-Traversal-Sequences-Blocked-with-Absolute-Path-Bypass.md) |
| 03 | Traversal Sequences Stripped Non-Recursively | Path Traversal — Non-Recursive Sanitization Bypass | High | [Lab-03-Traversal-Sequences-Stripped-Non-Recursively.md](./Lab-03-Traversal-Sequences-Stripped-Non-Recursively.md) |
| 04 | Traversal Sequences Stripped with Superfluous URL-Decode | Path Traversal — Double URL-Decoding Bypass | High | [Lab-04-Traversal-Sequences-Stripped-with-Superfluous-URL-Decode.md](./Lab-04-Traversal-Sequences-Stripped-with-Superfluous-URL-Decode.md) |
| 05 | Validation of Start of Path | Path Traversal — Prefix Validation Bypass | High | [Lab-05-Validation-of-Start-of-Path.md](./Lab-05-Validation-of-Start-of-Path.md) |
| 06 | Validation of File Extension with Null Byte Bypass | Path Traversal — Null Byte Injection | High | [Lab-06-Validation-of-File-Extension-with-Null-Byte-Bypass.md](./Lab-06-Validation-of-File-Extension-with-Null-Byte-Bypass.md) |
