# TryHackMe — "Upload Vulnerabilities" Room: Solving Guide

[![Platform](https://img.shields.io/badge/Platform-TryHackMe-red)](https://tryhackme.com/room/uploadvulns)
[![Type](https://img.shields.io/badge/Type-Guided%20Walkthrough-blue)]()

> Room link: [tryhackme.com/room/uploadvulns](https://tryhackme.com/room/uploadvulns) (Premium room)
>
> **This document explains *how* to approach and solve each task's questions — it deliberately does not give you the final flag values.** The flags are unique per-deployment anyway (they're generated fresh when you deploy the room's machine), so a hardcoded answer wouldn't work for you regardless. Use this as a guide for *method*, then run the steps yourself against your own deployed instance to get your own flags.

---

## Setup

Deploy the attached machine, then add its IP to your `/etc/hosts` (Linux/macOS) or `C:\Windows\System32\drivers\etc\hosts` (Windows) under the hostnames the room specifies (`overwrite.uploadvulns.thm`, `shell.uploadvulns.thm`, `java.uploadvulns.thm`, `annex.uploadvulns.thm`, `magic.uploadvulns.thm`, `jewel.uploadvulns.thm`). The room gives you the exact command — just substitute your machine's assigned IP.

## Task 4 — Overwriting Existing Files (`overwrite.uploadvulns.thm`)

**What the task is testing:** whether an upload form lets you overwrite a file that already exists on the server, simply by uploading a file with the same name.

**How to approach it:**
1. Browse to the site and view the page source to find where the displayed image is being loaded from (look for an `<img src="...">` tag) — this tells you both the filename and the directory it lives in.
2. Find or create a replacement image file with the **exact same filename**.
3. Use the upload form to submit your replacement.
4. Reload the page — if the displayed image changed to yours, the overwrite succeeded and the flag should be revealed (either directly on the page, in the new image, or in the server's response).

**Question 1** asks for the filename being overwritten — this comes straight from the `<img>` tag you find in step 1.
**Question 2** asks for the flag you receive after a successful overwrite — it appears once the overwrite is confirmed.

## Task 5 — Remote Code Execution (`shell.uploadvulns.thm`)

**What the task is testing:** basic web shell / reverse shell upload against a site with no filtering at all.

**How to approach it:**
1. Run a directory brute-force scan first — this is explicitly called out as a good habit:
   ```bash
   gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://shell.uploadvulns.thm
   ```
   Note which directory looks like it's for storing uploads (this answers the Gobuster question).
2. Confirm the backend is PHP (try uploading a legitimate image first and see where/how it's served).
3. Write a minimal PHP web shell:
   ```php
   <?php echo system($_GET["cmd"]); ?>
   ```
   Upload it through the form.
4. Navigate to the uploaded file's URL with a `cmd` parameter, e.g. `?cmd=whoami` or `?cmd=ls+/var/www/` to confirm execution and explore.
5. Once you've confirmed command execution, either keep using the web shell's `cmd` parameter to `cat` the flag directly, **or** upload a full PHP reverse shell (e.g. the well-known pentestmonkey script — remember to edit the IP at the top to your TryHackMe `tun0` address) and catch it with a Netcat listener (`nc -lvnp <port>`) for an interactive session.
6. Locate and read the flag file inside `/var/www/`.

**Tip:** Viewing the page source rather than the rendered page often makes your shell's command output far easier to read.

## Task 6 — Filtering (Theory, No Target)

**What the task is testing:** understanding of core concepts before you start bypassing anything.

**How to approach it:** this task is purely conceptual — re-read the explanation of client-side vs. server-side filtering, and the different validation types (extension, MIME, magic number, length, filename). The three questions are directly definitional:
- What language has traditionally dominated server-side web scripting?
- What's the term for an extension-validation approach that only accepts a pre-approved list (the opposite of a blacklist)?
- What MIME type does a `.csv` file typically report? (A quick search for "CSV MIME type" gives you this.)

## Task 7 — Bypassing Client-Side Filtering (`java.uploadvulns.thm`)

**What the task is testing:** that filtering which runs only in the browser provides no real security.

**How to approach it:**
1. Run Gobuster against the target first, as recommended, to discover the upload directory (it changes per-challenge).
2. View the page's source / any loaded `.js` files to find the client-side validation logic. The room explicitly warns: *"not all client-side scripts are inline"* — meaning the validation might live in an external `.js` file rather than directly in the HTML. Check Burp's proxy history for all loaded resources, not just the main page.
3. Once you've found the check (likely comparing `file.type` against an allowed MIME string), pick one of the four bypass approaches explained in the task:
   - Disable JavaScript entirely in your browser.
   - Intercept the **response** carrying the page/script in Burp (`Proxy → Intercept → "Do intercept" → "Response to this request"`) and strip or break the validation function before it loads.
   - Let the page load normally, select a file that passes the client check, then intercept the **upload request itself** in Burp and swap the file's content/extension/MIME after the client-side gate already let it through.
   - Skip the browser UI and `POST` directly to the upload endpoint with `curl`, supplying whatever filename/MIME/content you want.
4. Get a shell uploaded and executing (same approach as Task 5), then read the flag from `/var/www/`.

## Task 8 — Bypassing Server-Side Filtering: File Extensions (`annex.uploadvulns.thm`)

**What the task is testing:** that a naive extension blacklist or substring check can be defeated once you understand *how* it's implemented.

**How to approach it — this is a black-box enumeration exercise:**
1. Establish a baseline: upload a legitimate image. Confirm it's accepted and find out how/where it's served.
2. Try the obvious: upload `shell.php` directly. It will almost certainly be rejected — note the exact rejection message, it sometimes hints at *why*.
3. Systematically test alternate PHP-executable extensions one at a time: `.php3`, `.php4`, `.php5`, `.php7`, `.phtml`, `.phar`, `.pht`. Some might upload successfully but fail to *execute* (meaning the extension passed the filter but Apache isn't configured to run it as PHP) — keep trying until you find one that both uploads **and** executes.
4. If no single extension works, consider that the filter might be checking for a substring (e.g. "does `.jpg` appear anywhere in this filename?") rather than a strict suffix. Try a double extension: `shell.jpg.php`.
5. Remember the task's own warning: this challenge has **randomized filenames**, and directory listing may be off. If your upload succeeds but you can't guess the path, Gobuster with the `-x` flag to append likely extensions to a wordlist of guesses can help locate it — or look closely at the upload response/confirmation message, which may reveal the generated name directly.
6. Once your shell executes, retrieve the flag from `/var/www/`.

## Task 9 — Bypassing Server-Side Filtering: Magic Numbers (`magic.uploadvulns.thm`)

**What the task is testing:** bypassing content-based (not just extension-based) validation by faking a file signature.

**How to approach it:**
1. Confirm that a plain `shell.php` is rejected but a legitimate image is accepted — this suggests content inspection, not just extension filtering.
2. Pick a file signature for a format you'd expect to be allowed (the task specifically asks for **GIF** here — look up the GIF89a header bytes).
3. Open your PHP shell file in a hex editor (`hexeditor` is pre-installed on Kali) and overwrite the very first bytes with the chosen magic number. Alternatively, prepend the equivalent ASCII characters directly at the top of a plain-text PHP file, then verify with the `file` command that it now reports as the spoofed type.
4. Upload the modified file. If magic-number validation is the only content check in place, it should now be accepted.
5. This challenge, like Task 8, uses a less-discoverable storage location (directory indexing is off) — run **Gobuster** to find candidate upload directories first (`/graphics`, `/assets`, etc., are hinted at), then work out your file's exact path from there.
6. Trigger the shell directly via its URL and retrieve the flag from `/var/www/`.

## Task 10 — Example Methodology (Theory, No Target)

No target to hack here — this task lays out a general step-by-step approach for file upload testing (fingerprint the stack → find the upload point → inspect client-side code → do a baseline legitimate upload and work out where it lands → attempt a malicious upload and read the error → use the error to infer which filter type is in play → test extension/magic-number/MIME/length filters one at a time to characterize them precisely). Internalize this flow — you'll need it unmodified for the final challenge.

## Task 11 — Challenge (`jewel.uploadvulns.thm`)

**What the task is testing:** applying everything from Tasks 6–10 together, black-box, with no hints about which filters are active.

**How to approach it, following the Task 10 methodology exactly:**
1. **Fingerprint first.** Load the site in Burp and check response headers (`Server`, `X-Powered-By`) — don't assume it's PHP; the room explicitly warns *"not all webservers have a PHP backend."*
2. **Enumerate directories and endpoints.** Run Gobuster against the root, and separately against any interesting subdirectory you discover, ideally with a tailored or larger wordlist, and consider the `-x` flag.
3. **Inspect the upload mechanism closely.** This challenge's upload may not be a traditional multipart form — check any loaded JavaScript for how the file is actually transmitted (hint: look for it being read and sent as part of a JSON payload rather than a standard file field).
4. **Establish your baseline** with a known-good upload, and figure out exactly how uploaded files are named and where they're placed.
5. **Layer your bypasses.** This challenge is explicitly described as combining multiple filters — work through extension bypasses, magic number spoofing, and MIME type changes as needed, testing one variable at a time.
6. **If you get stuck:** the room links an official hints page and a full video walkthrough — use the hints incrementally to keep the learning value of the challenge.
7. Once you achieve execution, retrieve the flag from `/var/www/`.

---

## General Tips That Apply Across the Whole Room

- **Always view source / check Burp history for *every* loaded resource**, not just the page you're looking at — client-side filters often live in separate `.js` files.
- **Gobuster is your friend for almost every task** — run it early, and don't forget the `-x` flag when you suspect extension-based naming or storage.
- **Change one variable at a time** when probing a black-box filter (extension, then MIME, then content) so you can tell exactly which check you defeated.
- **Check the actual response body**, not just whether the browser renders something — a shell's output is often visible in the raw HTML/response even if the page "looks broken."
- If your reverse shell script needs editing (e.g. pentestmonkey's PHP reverse shell), **double-check the IP and port** match your TryHackMe VPN (`tun0`) address and your active Netcat listener before uploading.

---

*This guide is a method reference for the TryHackMe "Upload Vulnerabilities" room, written for self-study. It intentionally omits flag values since they are unique per deployment — work through each step against your own running instance.*
