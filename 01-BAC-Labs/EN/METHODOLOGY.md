# 🧠 Pentester Mindset — Broken Access Control

> Before diving into individual lab writeups, this document explains **how to think** when hunting for Broken Access Control (BAC) issues — not just the specific payloads, but the underlying questions a tester should be asking at every step.

---

## 1. The Core Question

Every access control test boils down to one question, asked repeatedly:

> **"Is the server actually verifying that *this* user is allowed to do *this* action on *this* resource — on every single request?"**

If the answer relies on anything other than a fresh, server-side check tied to the authenticated session, the control is probably broken.

## 2. The Three Flavors of BAC

| Type | What it means | The question you ask |
|---|---|---|
| **Vertical** | A low-privilege user reaches high-privilege functionality (e.g. a normal user hitting an admin endpoint). | "What happens if I just request the admin URL directly?" |
| **Horizontal** | A user accesses another user's data of the *same* privilege level (classic IDOR). | "What happens if I change this ID/reference to someone else's?" |
| **Context-dependent** | Access control that depends on a multi-step process, a specific HTTP method, or an external signal (Referer, headers). | "What happens if I skip a step, change the method, or forge the signal?" |

## 3. What a Tester Actually Looks For

- **Hidden or unlinked functionality**: admin panels, debug routes, internal APIs not shown in the UI. Check `robots.txt`, JS bundles, sitemap files, and any client-side conditional rendering (`if (isAdmin)`).
- **Any parameter that looks like an identifier**: `id`, `user`, `account`, `file`, `ref`. Every such parameter is a candidate for IDOR — try substituting another user's value.
- **Any client-controlled state used for a security decision**: cookies (`Admin=true`), hidden form fields, JSON body fields not shown in the UI (mass assignment candidates).
- **Multi-step workflows**: wizards, confirmation screens, "step 2 of 2" flows. Ask: *"What happens if I call the final step directly, skipping the earlier gate?"*
- **How the same action is reachable by more than one route**: does the app expose the same business logic via `GET` and `POST`? Via a public path *and* a proxy-rewritten internal path (`X-Original-URL`)?
- **Response bodies on denied requests**: a `302`/`403` doesn't guarantee the body is empty — inspect raw responses (not just what the browser shows you) for data that was rendered *before* the redirect was issued.

## 4. A Practical Testing Checklist

1. **Map the application** as a low-privilege (or unauthenticated) user. List every URL, parameter, and action you can see.
2. **Map it again** as a higher-privilege user (if you have one), and diff the two maps — anything only visible to the higher-privileged user is now a target to try reaching directly as the lower-privileged one.
3. For every **identifier-looking parameter**, try: another user's value, a sequential neighbor, a null/empty value, an array/object instead of a scalar.
4. For every **write/update endpoint**, inspect the full request body (not just the UI fields) and try injecting plausible privileged field names (`role`, `isAdmin`, `roleId`, `permissions`).
5. For every **sensitive action**, try: a different HTTP method, removing/forging headers like `Referer` or custom routing headers, and calling it with no prior steps in a multi-step flow.
6. Always test from **two different authenticated sessions** (or one authenticated + one anonymous) so you can clearly tell what *should* be denied vs. what's actually being returned.

## 5. Tools of the Trade

- **Burp Suite** — Repeater for manual parameter tampering, Intruder for enumerating IDs, the raw HTTP history for inspecting actual response bodies (not the rendered browser view).
- **Autorize / Auth Analyzer (Burp extensions)** — automatically replay every request under a second, lower-privileged session and flag ones that still succeed.
- **Browser DevTools** — inspect loaded JS for hidden routes and client-side-only authorization checks.

## 6. The Defender's Mirror

For every attacker technique above, the corresponding defensive question is the same:

> **Does the server independently verify authorization for this exact resource and action, on this exact request, regardless of how the client got here?**

If a fix only addresses the specific bypass you found (e.g. blocking one URL) without answering that question generally, assume there's another bypass nearby.

---

➡️ Once you're comfortable with this mindset, head to the [lab index](./README.md) to see how each of these ideas plays out in a concrete, step-by-step exploit.
