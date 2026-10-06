# Broken Access Control Labs — Index

Professional PoC writeups for PortSwigger Web Security Academy's Broken Access Control labs (01–13).

## 🧠 Start Here

Before jumping into the lab writeups, read the **[Pentester Mindset — Broken Access Control](./METHODOLOGY.md)** doc. It explains how to *think* about this vulnerability class — what questions to ask, what to look for, and the general testing checklist — rather than just the specific payloads used in each lab.

## 📋 Lab Index

| # | Lab | Type | Severity | File |
|---|---|---|---|---|
| 01 | Unprotected Admin Functionality | Vertical Broken Access Control | High | [Lab-01-Unprotected-Admin-Functionality.md](./Lab-01-Unprotected-Admin-Functionality.md) |
| 02 | Unprotected Admin Functionality with Unpredictable URL | Vertical Broken Access Control | High | [Lab-02-Unprotected-Admin-Functionality-with-Unpredictable-URL.md](./Lab-02-Unprotected-Admin-Functionality-with-Unpredictable-URL.md) |
| 03 | User Role Controlled by Request Parameter | Vertical Broken Access Control | Critical | [Lab-03-User-Role-Controlled-by-Request-Parameter.md](./Lab-03-User-Role-Controlled-by-Request-Parameter.md) |
| 04 | User Role Can Be Modified in User Profile | Vertical Broken Access Control | Critical | [Lab-04-User-Role-Can-Be-Modified-in-User-Profile.md](./Lab-04-User-Role-Can-Be-Modified-in-User-Profile.md) |
| 05 | User ID Controlled by Request Parameter | Horizontal Broken Access Control (IDOR) | High | [Lab-05-User-ID-Controlled-by-Request-Parameter.md](./Lab-05-User-ID-Controlled-by-Request-Parameter.md) |
| 06 | User ID Controlled by Request Parameter with Unpredictable User IDs | Horizontal Broken Access Control (IDOR with Enumerable GUID) | High | [Lab-06-User-ID-Controlled-by-Request-Parameter-with-Unpredictable-User-IDs.md](./Lab-06-User-ID-Controlled-by-Request-Parameter-with-Unpredictable-User-IDs.md) |
| 07 | User ID Controlled by Request Parameter with Data Leakage in Redirect | Horizontal Broken Access Control (Sensitive Data Leak via 302 Response Body) | High | [Lab-07-User-ID-Controlled-by-Request-Parameter-with-Data-Leakage-in-Redirect.md](./Lab-07-User-ID-Controlled-by-Request-Parameter-with-Data-Leakage-in-Redirect.md) |
| 08 | User ID Controlled by Request Parameter with Password Disclosure | Horizontal-to-Vertical Privilege Escalation (Password Disclosure) | Critical | [Lab-08-User-ID-Controlled-by-Request-Parameter-with-Password-Disclosure.md](./Lab-08-User-ID-Controlled-by-Request-Parameter-with-Password-Disclosure.md) |
| 09 | Insecure Direct Object References (Predictable Static Files) | Horizontal Broken Access Control (IDOR via Predictable Filenames) | High | [Lab-09-Insecure-Direct-Object-References-(Predictable-Static-Files).md](./Lab-09-Insecure-Direct-Object-References-(Predictable-Static-Files).md) |
| 10 | URL-based Access Control Can Be Circumvented | Vertical Broken Access Control (Reverse Proxy / Edge-Layer Bypass) | Critical | [Lab-10-URL-based-Access-Control-Can-Be-Circumvented.md](./Lab-10-URL-based-Access-Control-Can-Be-Circumvented.md) |
| 11 | Method-based Access Control Can Be Circumvented | Vertical Broken Access Control (HTTP Method Bypass) | High | [Lab-11-Method-based-Access-Control-Can-Be-Circumvented.md](./Lab-11-Method-based-Access-Control-Can-Be-Circumvented.md) |
| 12 | Multi-step Process with No Access Control on One Step | Vertical Broken Access Control (Multi-Step Workflow Bypass) | Critical | [Lab-12-Multi-step-Process-with-No-Access-Control-on-One-Step.md](./Lab-12-Multi-step-Process-with-No-Access-Control-on-One-Step.md) |
| 13 | Referer-based Access Control | Vertical Broken Access Control (Referer Header Spoofing) | High | [Lab-13-Referer-based-Access-Control.md](./Lab-13-Referer-based-Access-Control.md) |
