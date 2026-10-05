# Lab 05 — Information Disclosure in Version Control History

[![Category](https://img.shields.io/badge/Category-Information%20Disclosure-blue)]()
[![Type](https://img.shields.io/badge/Type-Information%20Disclosure%20—%20Exposed%20.git%20Repository-orange)]()
[![Severity](https://img.shields.io/badge/Severity-High-critical)]()
[![Source](https://img.shields.io/badge/Source-PortSwigger%20Web%20Security%20Academy-blue)](https://portswigger.net/web-security/information-disclosure)

> Part of the [Information Disclosure Labs](./README.md) series — PortSwigger Web Security Academy.

---

## 1. Overview

| Field | Detail |
|---|---|
| **Lab Name** | Information Disclosure in Version Control History |
| **Vulnerability Class** | Information Disclosure — Exposed .git Repository |
| **Weakness (CWE)** | CWE-527: Exposure of Version-Control Repository to an Unauthorized Control Sphere |
| **Severity** | High |
| **Objective** | Download the exposed .git directory, mine the commit history for a removed administrator password, then log in and delete user carlos. |

## 2. Summary

The application's `.git` version-control directory was deployed to the public web root. Downloading it reconstructs the full commit history locally. Although a later commit removed a hard-coded admin password from the config file (replacing it with an environment-variable reference), Git permanently retains every prior version of every file — the deleted password is still fully readable in that commit's diff.

## 3. Proof of Concept (PoC)

**Step 1 — Confirm exposure:**  
Request `/.git/HEAD` — a `200 OK` with content like `ref: refs/heads/master` confirms the repository is publicly accessible.

**Step 2 — Mirror the repository:**  
From a terminal:

```bash
wget -r --no-parent https://<lab-id>.web-security-academy.net/.git/
```

**Step 3 — Inspect the commit log:**  
From inside the downloaded directory:

```bash
git log --oneline
```

Look for a suspicious message such as `Remove admin password from config`.

**Step 4 — Diff the suspicious commit:**  
```bash
git show <commit-hash>
```

The diff shows the deleted line in red, containing the plaintext password that was replaced by an environment-variable reference.

**Step 5 — Log in and exploit:**  
Use the recovered credentials (`administrator` / leaked password) to log in, then delete `carlos` from the admin panel.


## 4. Evidence (Request / Response)

```http
$ git log --oneline
a54974a (HEAD -> master) Remove admin password from config
4db2c9a Add skeleton admin panel

$ git show a54974a
diff --git a/admin.conf b/admin.conf
index 6062a05..21d23f1 100644
--- a/admin.conf
+++ b/admin.conf
@@ -1 +1 @@
-ADMIN_PASSWORD=k7jsy38dtcb6fjisxwp7
+ADMIN_PASSWORD=env('ADMIN_PASSWORD')
```

## 5. Root Cause Analysis

The `.git` directory — which by design contains the complete, permanent history of every change ever made to the codebase — was deployed alongside the application's public files instead of being excluded from the production build/deploy artifact. Removing a secret in a later commit does not erase it from history; the object remains retrievable by anyone with read access to the repository.

### Attacker vs. Defender Perspective

- **Attacker view:** Pull the exposed .git directory and mine the full commit history (not just the current file state) for secrets removed in later commits.
- **Defender view:** Exclude .git from deployments, block it at the web-server level, and rotate+purge any secret that was ever committed.

## 6. Impact

Full compromise of the administrator account, and more broadly, complete exposure of the application's development history — which commonly contains other secrets, internal comments, or logic that was never meant to reach production, dramatically expanding the attack surface.

## 7. Remediation

- Never deploy the `.git` directory (or any VCS metadata) to a publicly accessible web root; explicitly exclude it in deployment scripts/CI pipelines and block access to dotfiles at the web-server level (e.g. `location ~ /\.git { deny all; }` in Nginx).
- Treat any secret ever committed to version control as permanently compromised: removing it in a later commit is not sufficient — rotate the credential immediately and purge it from history using tools like `git filter-repo` or BFG Repo-Cleaner.
- Never commit hard-coded secrets in the first place; use environment variables backed by a secrets manager from the start of a project.
- Integrate automated secret-scanning (e.g. TruffleHog, Gitleaks) into CI/CD pipelines to catch committed secrets before they ever reach a remote repository.

## 8. References

- [PortSwigger — Information Disclosure](https://portswigger.net/web-security/information-disclosure)
- [OWASP Top 10 2021 — A01/A05: Broken Access Control / Security Misconfiguration](https://owasp.org/Top10/)
- [CWE-527: Exposure of Version-Control Repository to an Unauthorized Control Sphere](https://cwe.mitre.org/data/definitions/527.html)

---

*Writeup produced for educational and authorized lab purposes only (PortSwigger Web Security Academy environment).*
