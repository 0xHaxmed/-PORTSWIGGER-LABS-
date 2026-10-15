# TryHackMe — Broken Access Control Study Guide

[![Platform](https://img.shields.io/badge/Platform-TryHackMe-red)](https://tryhackme.com/room/brokenaccesscontrol)
[![Category](https://img.shields.io/badge/Category-Broken%20Access%20Control-blue)]()

This study guide walks through the **Broken Access Control** room on TryHackMe, explaining concepts, analysis methods, and practical exploitation steps without spoiling dynamic flags.

---

## Task 1 & 2: Concepts & Access Control Models
Access control dictates who can access which resources:
- **Discretionary Access Control (DAC):** Resource owners set permissions.
- **Mandatory Access Control (MAC):** Central authority enforces strict classification labels.
- **Role-Based Access Control (RBAC):** Privileges are tied to roles (Admin, Manager, User).
- **Attribute-Based Access Control (ABAC):** Multi-attribute decisions (time, IP, user status).

---

## Task 4: Application Reconnaissance
When logging into the target application:
1. Observe the response from `functions.php`.
2. Notice the JSON keys: `status`, `message`, `first_name`, `last_name`, `is_admin`, and `redirect_link`.
3. The server communicates the user's role and redirect target directly to the client browser in JSON format.

---

## Task 5: Exploitation (Vertical Privilege Escalation)
1. Intercept the login response in Burp Suite Proxy.
2. In the JSON response, modify the `redirect_link` and role parameters (change `false` to `true`).
3. Forward the modified response. The browser will redirect to `admin.php`.
4. From the admin interface, promote your regular user or change privileges directly.

---

## Task 6: Mitigation Strategies
- Implement RBAC enforced strictly server-side on every request.
- Use parameterized queries (Prepared Statements) to prevent authentication bypasses.
- Use secure HTTP-only, SameSite cookies with strict session expiration.
