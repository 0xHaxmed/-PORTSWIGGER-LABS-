# 🧠 Pentester Mindset — File Upload Vulnerabilities

> Before diving into individual lab writeups, this document explains **how to think** when hunting for file upload vulnerabilities — what layers of defense exist, how to probe each one, and the order in which to try bypasses.

---

## 1. The Core Question

> **"Once this file lands on the server, can I make it *execute* — and can I make the server tell *me* what happened when it did?"**

A file upload vulnerability isn't just "I can upload a weird file." It's the full chain: **upload → land somewhere reachable → get interpreted as code (or otherwise abused) → I observe the result.** Break any link, and the vulnerability is still there but you can't prove or use it — so test the whole chain, not just the upload step.

## 2. Map the Validation Layers First

Real applications stack multiple independent checks. Before trying payloads, figure out *which* layers exist by testing each in isolation:

| Layer | What it checks | How to detect it |
|---|---|---|
| **Client-side (JS)** | Extension/MIME before the request is even sent | View page source / JS files; trivially bypassed by disabling JS or editing the request directly |
| **Extension filter** | The filename's extension (blacklist or allow-list) | Upload a file with an obviously invalid extension (e.g. `.invalidext`) — if it's *accepted*, it's a blacklist; if rejected, it's an allow-list |
| **MIME type filter** | The `Content-Type` in the multipart body | Upload a legitimate file, then replay it with the `Content-Type` changed to something that should be blocked |
| **Magic number / content filter** | The actual byte signature at the start of the file | Upload a legitimate image, then corrupt its first bytes and re-upload — if it's rejected, the server is checking magic numbers |
| **File length filter** | Upload size limits | Upload progressively larger files until one is rejected |
| **Filename handling** | Whether your filename survives intact, and whether it can traverse directories | Upload something and then try to find it; try embedding `../` (and URL-encoded variants) in the filename |

Testing each layer independently (rather than guessing a combined bypass blind) tells you exactly which defenses you're up against and which bypass techniques are even relevant.

## 3. The Escalation Order for Extension Filters

1. **Try an extension the filter doesn't know about.** PHP alone can be executed via `.php`, `.php3`, `.php4`, `.php5`, `.php7`, `.phtml`, `.phar`, and more, depending on server config — a blacklist that only covers `.php` is trivially bypassed.
2. **Try smuggling the extension inside the filename.** If validation checks "is `.jpg` anywhere in this string" rather than "does the filename end with `.jpg`," a name like `shell.jpg.php` can slip through.
3. **Try obfuscating the end of the filename.** Null bytes (`exploit.php%00.jpg`), trailing dots, trailing whitespace, or alternate data streams (Windows) have all historically defeated extension-suffix checks on certain stacks.
4. **Try path traversal in the filename.** Even when execution is disabled in the target directory, a (URL-encoded) `../` sequence in the filename may let you write the file into a parent directory with different (laxer) execution rules.
5. **Try uploading a server config file.** `.htaccess` (Apache) or `web.config` (IIS) can redefine which extensions the server treats as executable — letting you register your *own* custom, non-blacklisted extension as a script handler.

## 4. The Escalation Order for Content-Based Filters

1. **Check if only the extension/MIME is validated, not actual content.** If so, a `.jpg`-extensioned PHP file with no real image structure may still pass.
2. **Fake the magic number.** Prepend the correct byte signature for an allowed format (e.g. `FF D8 FF DB` for JPEG, `47 49 46 38 37 61` for GIF) to the front of your payload file.
3. **Build a genuine polyglot.** If the server actually parses and validates the image structure (not just the first few bytes), you need a file that is *simultaneously* a structurally valid image *and* contains your payload somewhere the parser ignores — classically, embedded in EXIF metadata (`exiftool -Comment="<?php ... ?>" image.jpg -o shell.php`).
4. **Consider a race condition.** If validation happens *after* the file is already written to a public location, a well-timed request during that window can execute the file before it's deleted — regardless of how strict the validation itself is.

## 5. Finding Where Your File Landed

A successful upload is useless if you can't locate it:

- Check the response for a direct link or confirmation message naming the stored path.
- Try the most predictable location first: `/uploads/<filename>`, `/files/avatars/<filename>`, `/assets/<filename>`.
- If the filename was randomized, brute-force or enumerate with a tool like **Gobuster** (`-x php,phtml,phar` to also guess extensions on top of names).
- Check for directory listing being enabled on likely upload folders.

## 6. Burp Suite Workflow

- **Repeater** for manual tampering of extensions, MIME types, and filenames.
- **Intruder** to fuzz through a list of extensions or payload variants quickly.
- **Turbo Intruder** (or Repeater's "Send group in parallel / single-packet attack") for race-condition exploitation, where precise timing between the upload and the follow-up read matters.
- Always check the **raw response**, not just what the browser renders — a `200 OK` with your payload reflected back confirms success even if the browser shows a broken image icon.

## 7. The Defender's Mirror

For every bypass above, the underlying principle is the same:

> **Never let an uploaded file be both reachable by a URL and executable as code.** Validate content genuinely (not just metadata), re-encode media server-side, store uploads outside the web root or in a non-executing location, and never expose a file publicly before validation has fully completed.

---

➡️ Once you're comfortable with this mindset, head to the [lab index](./README.md) to see how each of these ideas plays out in a concrete, step-by-step exploit — or read the [full topic deep dive](./TOPIC-OVERVIEW.md) for the complete theory and payload cheat sheet.
