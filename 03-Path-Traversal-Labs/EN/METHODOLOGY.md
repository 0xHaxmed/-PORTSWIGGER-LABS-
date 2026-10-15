# 🧠 Pentester Mindset — Path Traversal

> Before diving into individual lab writeups, this document explains **how to think** when hunting for path traversal issues — what makes a parameter suspicious, how filters typically fail, and the order in which to try your bypasses.

---

## 1. The Core Question

> **"Is this parameter ever used to build a filesystem path — and if so, does the server actually confirm the final, resolved path stays inside the folder it's supposed to?"**

Almost every path traversal vulnerability exists because a developer checked the *raw input string* instead of the *final, resolved path* the filesystem will actually use.

## 2. What Makes a Parameter a Target

Any parameter that smells like it references a file is worth testing:

- `filename`, `file`, `path`, `doc`, `template`, `page`, `image`, `download`, `report`
- Parameters used to load: images, PDFs, templates, log files, language/locale files, includes

A quick mental test: *"If this value gets concatenated onto a base folder path, what happens if I add `../` to it?"*

## 3. The Standard Escalation Order

Don't jump straight to exotic bypasses. Work through defenses in increasing sophistication — this mirrors how real applications are typically built (and mirrors the lab series):

1. **Plain traversal**: `../../../etc/passwd` — tests for *no* defense at all.
2. **Absolute path**: `/etc/passwd` — tests whether the path-joining function discards the base directory when given an absolute path.
3. **Nested/doubled sequences**: `....//....//....//etc/passwd` — tests whether the filter strips `../` only once (non-recursively), letting a nested sequence collapse back into a real one.
4. **Encoding tricks**: `%2e%2e%2f`, then `%252e%252e%252f` (double-encoded), then non-standard encodings like `..%c0%af` or `..%ef%bc%8f` — tests whether filtering happens before or after decoding, and how many decode passes occur.
5. **Prefix-satisfying payloads**: `/var/www/images/../../../etc/passwd` — tests whether validation only checks that the input *starts with* the expected folder, without resolving the full path.
6. **Null byte truncation**: `../../../etc/passwd%00.png` — tests whether a required file extension is enforced on the full string while the underlying C-based file API truncates at the first null byte (mostly historical, but still worth trying against legacy/C-based stacks).

## 4. Always Check Both Directions

- **Unix-style**: `../`
- **Windows-style**: `..\` (and note that `C:\Windows\win.ini` is the classic Windows equivalent of `/etc/passwd` as a universally-readable confirmation target)

Some filters only account for one slash direction — always try both.

## 5. What to Do Once You Get a Read

A successful `/etc/passwd` read is a proof of concept, not the end of the engagement. Treat it as reconnaissance:

- **List interactive users** (shells ending in `/bin/bash` rather than `/usr/sbin/nologin`) — these are real accounts worth targeting.
- **Note home directories** — each one is a new target for files like `~/.ssh/id_rsa`, `~/.bash_history`, or application config files.
- **Note running services** implied by system accounts (`mysql`, `postgres`, `mongodb`) — these hint at other components worth probing.
- **Pivot to config/secret files**: `.env`, `wp-config.php`, application YAML/JSON configs, log files — anywhere credentials or internal URLs might be hard-coded.

## 6. Burp Suite Workflow

1. Intercept a request that loads a file-like resource and send it to **Repeater**.
2. Walk through the escalation order above, one payload at a time, reading the response each time.
3. For broader parameter fuzzing, use **Intruder** with Burp's predefined **"Fuzzing — path traversal"** payload list, which already includes many of the encoded variants above.
4. Always check the **raw response body** — an image endpoint returning `200 OK` with `Content-Type: image/jpeg` but a text-based `/etc/passwd` body is a dead giveaway, even if the browser doesn't render it visibly.

## 7. The Defender's Mirror

For every bypass above, the same single fix generally applies:

> **After building the full path (base + user input), resolve it to its canonical, absolute form and explicitly verify it still starts with the intended base directory — reject the request if it doesn't.**

Blacklisting `../`, checking string prefixes, or checking file extensions are all surface-level checks that operate on the *unresolved* string — which is exactly what every bypass above exploits. Canonicalization (`getCanonicalPath()`, `realpath()`, `os.path.realpath()`) is the one check that closes all of them at once.

---

➡️ Once you're comfortable with this mindset, head to the [lab index](./README.md) to see how each of these ideas plays out in a concrete, step-by-step exploit.
