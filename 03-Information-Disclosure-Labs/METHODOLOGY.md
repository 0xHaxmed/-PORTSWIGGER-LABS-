# 🧠 Pentester Mindset — Information Disclosure

> Before diving into individual lab writeups, this document explains **how to think** when hunting for information disclosure issues — where to look, what counts as "interesting," and how to judge whether a leak actually matters.

---

## 1. The Core Question

> **"Does this response contain anything that wasn't meant for me — and if so, what can I actually *do* with it?"**

Information disclosure is rarely the end goal on its own; it's almost always reconnaissance for a bigger attack. The skill is recognizing a useful leak the moment you see it, even while you're testing for something completely different.

## 2. Two Categories Worth Separating

| Category | Examples | Why it matters |
|---|---|---|
| **Direct sensitive data (High Impact)** | Credit card numbers, passwords, API keys, session tokens | Immediately exploitable, no extra steps needed. |
| **Technical / infrastructure data (Recon)** | Framework + version, internal file paths, stack traces, server banners | Rarely dangerous *alone* — its value is as the missing piece for a bigger, chained exploit (e.g. a known CVE for that exact framework version). |

Don't dismiss technical leaks as "low severity" without first checking whether the disclosed version/technology has known, documented exploits.

## 3. Avoid Tunnel Vision

Leaks show up in all kinds of unrelated places. The core discipline is: **never focus so narrowly on one vulnerability class that you stop noticing anomalies elsewhere.** You'll find most information disclosure issues while testing for something else entirely — a slightly different error message, an unexpected field in a JSON response, a comment in page source you almost skipped past.

## 4. Where Leaks Actually Hide

| Source | What to look for |
|---|---|
| `robots.txt` / `sitemap.xml` | Directories the developer tried to hide from search engines — a free map of "interesting" paths. |
| Directory listings | Auto-generated folder indexes exposing files never meant to be browsed directly. |
| HTML / JS comments | Developer notes, forgotten debug links, hints about internal logic. |
| Verbose error messages | Stack traces, DB table/column names, exact framework + version numbers. |
| Debug / diagnostic pages | `phpinfo()`, `/debug`, `/status` — tools meant for local dev, forgotten in production. |
| Backup files | `.bak`, `.old`, `~` — extensions the server doesn't know how to execute, so it serves the raw source instead. |
| Version control metadata | An exposed `.git` directory contains the *entire* history, including secrets removed in later commits. |
| Insecure configuration | Diagnostic HTTP methods like `TRACE`, which can echo internal headers added by a reverse proxy. |
| User-facing pages | Account/profile pages whose underlying data-fetching logic doesn't verify the requested user matches the session owner. |

## 5. How to Actively Elicit a Leak

Don't just browse normally — **provoke the application** into revealing more than it should:

- **Fuzz parameter types**: send a string where an integer is expected, an array where a string is expected, empty values, and special characters (`' " \ %00`). Watch how the error response changes.
- **Compare responses systematically**: status code, response length, timing — even subtle differences can confirm something is happening behind the scenes.
- **Use Burp's engagement tools**: `Find comments` to pull every HTML comment across the whole site map in one pass; `Discover content` to brute-force hidden paths not linked from the visible UI.
- **Grep responses for keywords**: `secret`, `admin`, `api_key`, `token`, `exception`, `root:`, `SELECT`, `password`.

## 6. Judging Severity Honestly

Not every technical detail you find deserves a high-severity writeup. Ask:

> **"Can I demonstrate a concrete, harmful next step using this specific piece of information?"**

A disclosed, fully-patched framework version is low-impact. The same version, unpatched, with a public RCE exploit, is critical. Context — not the mere presence of disclosure — determines severity.

## 7. The Defender's Mirror

> **Would I be comfortable with this exact response being public on the internet, forever?**

If not: use generic error pages, strip debug tooling from production, remove comments/backups before deploying, and never let `.git` or similar metadata reach the public web root.

---

➡️ Once you're comfortable with this mindset, head to the [lab index](./README.md) to see how each of these ideas plays out in a concrete, step-by-step exploit.
