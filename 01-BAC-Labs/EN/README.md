# Broken Access Control Labs (EN)

[![Category](https://img.shields.io/badge/Category-Broken%20Access%20Control-red)]()
[![Labs Covered](https://img.shields.io/badge/PortSwigger%20Labs-13%20Labs-blue)]()
[![THM](https://img.shields.io/badge/TryHackMe-Room%20Walkthrough-orange)]()

Welcome to the English documentation for **Broken Access Control (OWASP A01)**. This directory contains detailed, production-grade Proof-of-Concept writeups for all 13 PortSwigger Academy labs, along with complete architectural breakdowns and study guides.

---

## 📚 Core Documents

| Document | Description |
|---|---|
| [📖 OVERVIEW.md](./OVERVIEW.md) | Comprehensive theoretical guide on access control models (DAC, MAC, RBAC, ABAC) and flaw categories. |
| [🧠 METHODOLOGY.md](./METHODOLOGY.md) | The Pentester Mindset: Systematic mapping, testing workflows, and tools (Autorize). |
| [🔍 ROOT-CAUSE.md](./ROOT-CAUSE.md) | Technical breakdown of why access controls fail in real applications. |
| [🛡️ REMEDIATION.md](./REMEDIATION.md) | Developer guide for secure architecture, middleware enforcement, and mitigation code patterns. |
| [🎯 THM-Broken-Access-Control.md](./THM-Broken-Access-Control.md) | Complete conceptual walkthrough for the TryHackMe Broken Access Control room. |
| [🎯 THM-IDOR.md](./THM-IDOR.md) | Complete conceptual walkthrough for the TryHackMe Insecure Direct Object Reference (IDOR) room. |

---

## 📋 Lab Writeup Index

| # | Lab Title | Vulnerability Type | Severity | Link |
|---|---|---|---|---|
| 01 | Unprotected Admin Functionality | Vertical Access Control | High | [Lab-01](./Lab-01-Unprotected-Admin-Functionality.md) |
| 02 | Unprotected Admin Functionality with Unpredictable URL | Vertical Access Control | High | [Lab-02](./Lab-02-Unprotected-Admin-Functionality-with-Unpredictable-URL.md) |
| 03 | User Role Controlled by Request Parameter | Vertical Access Control | Critical | [Lab-03](./Lab-03-User-Role-Controlled-by-Request-Parameter.md) |
| 04 | User Role Can Be Modified in User Profile | Vertical Access Control (Mass Assignment) | Critical | [Lab-04](./Lab-04-User-Role-Can-Be-Modified-in-User-Profile.md) |
| 05 | User ID Controlled by Request Parameter | Horizontal Access Control (IDOR) | High | [Lab-05](./Lab-05-User-ID-Controlled-by-Request-Parameter.md) |
| 06 | User ID Controlled by Request Parameter with Unpredictable IDs | Horizontal Access Control (IDOR + GUID Leak) | High | [Lab-06](./Lab-06-User-ID-Controlled-by-Request-Parameter-with-Unpredictable-User-IDs.md) |
| 07 | User ID Controlled by Request Parameter with Data Leakage in Redirect | Horizontal Access Control (302 Body Leak) | High | [Lab-07](./Lab-07-User-ID-Controlled-by-Request-Parameter-with-Data-Leakage-in-Redirect.md) |
| 08 | User ID Controlled by Request Parameter with Password Disclosure | Horizontal-to-Vertical Privilege Escalation | Critical | [Lab-08](./Lab-08-User-ID-Controlled-by-Request-Parameter-with-Password-Disclosure.md) |
| 09 | Insecure Direct Object References (Predictable Static Files) | Horizontal Access Control (Static File IDOR) | High | [Lab-09](./Lab-09-Insecure-Direct-Object-References-(Predictable-Static-Files).md) |
| 10 | URL-based Access Control Can Be Circumvented | Vertical Access Control (Proxy Header Bypass) | Critical | [Lab-10](./Lab-10-URL-based-Access-Control-Can-Be-Circumvented.md) |
| 11 | Method-based Access Control Can Be Circumvented | Vertical Access Control (HTTP Method Bypass) | High | [Lab-11](./Lab-11-Method-based-Access-Control-Can-Be-Circumvented.md) |
| 12 | Multi-step Process with No Access Control on One Step | Vertical Access Control (Workflow Bypass) | Critical | [Lab-12](./Lab-12-Multi-step-Process-with-No-Access-Control-on-One-Step.md) |
| 13 | Referer-based Access Control | Vertical Access Control (Header Trust) | High | [Lab-13](./Lab-13-Referer-based-Access-Control.md) |
