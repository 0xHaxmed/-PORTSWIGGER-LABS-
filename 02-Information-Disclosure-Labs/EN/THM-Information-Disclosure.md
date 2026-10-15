# TryHackMe — Information Disclosure & Web Reconnaissance Guide

[![Platform](https://img.shields.io/badge/Platform-TryHackMe-red)](https://tryhackme.com/)
[![Category](https://img.shields.io/badge/Category-Information%20Disclosure-blue)]()

This study guide explains the essential methodologies for identifying and exploiting information disclosure vulnerabilities across web targets.

---

## 1. Passive & Active Information Gathering
- **Robots.txt & Sitemaps:** Always inspect `/robots.txt` and `/sitemap.xml` for unlinked administrative endpoints and backup directories.
- **HTTP Headers:** Inspect response headers (`Server`, `X-Powered-By`, `X-AspNet-Version`) to map technology stacks.
- **Source Code Comments:** Search for HTML and JavaScript comments (`<!-- TODO: debug endpoint -->`) indicating internal developer routes.

---

## 2. Directory & Backup Fuzzing
Use tools like `ffuf` or `gobuster` with targeted extension wordlists:
```bash
ffuf -u http://target.com/FUZZ -w wordlist.txt -e .bak,.old,.txt,.zip,.git
```

---

## 3. Git Repository Extraction
If `/.git/` is exposed, use automated dumpers such as `git-dumper` or `GitTools` to reconstruct the repository locally, then inspect `git log` and `git diff` for hardcoded credentials.
