# Broken Access Control — Comprehensive Overview

## 1. What is Broken Access Control?
Access control (authorization) is the mechanism that determines whether a user is permitted to perform a requested action or access a requested resource. Broken Access Control (BAC) occurs when these restrictions are not properly enforced, allowing attackers to access unauthorized data, modify resources, or perform administrative functions.

In the OWASP Top 10, Broken Access Control is ranked as **A01:2021 — Broken Access Control**, representing the single most prevalent and high-impact category of web application vulnerabilities.

## 2. Core Access Control Models
To understand how access control fails, we must first understand standard security models:
- **Discretionary Access Control (DAC):** Resource owners decide who has access to specific objects (e.g., standard filesystem permissions).
- **Mandatory Access Control (MAC):** A central authority defines strict security levels (e.g., Top Secret, Secret, Confidential); users cannot reassign permissions.
- **Role-Based Access Control (RBAC):** Permissions are assigned to specific roles (e.g., Admin, Manager, Editor, User), and users are assigned to roles.
- **Attribute-Based Access Control (ABAC):** Fine-grained dynamic policies evaluating attributes of the user, resource, action, and environment (e.g., location, time of day).

## 3. The Three Primary Dimensions of BAC

### A. Vertical Privilege Escalation
Occurs when an attacker accesses functionality or resources intended for users with higher permissions (e.g., a standard user accessing `/admin` or executing user-deletion APIs).
- **Common Vectors:** Missing role checks on sensitive routes, unprotected admin URLs, client-side authorization checks, reliance on hidden UI elements.

### B. Horizontal Privilege Escalation (IDOR)
Occurs when an attacker accesses or manipulates data belonging to another user of the *same* privilege level.
- **Classic IDOR (Insecure Direct Object Reference):** Changing `GET /api/user?id=101` to `id=102` to view another customer's invoice or private data.
- **Common Vectors:** Exposing database primary keys directly in URLs without validating that the authenticated session owns the requested record.

### C. Context-Dependent & Workflow Bypasses
Occurs when access control logic relies on assumptions about the user's workflow or external signals:
- **Multi-step workflow bypasses:** Enforcing checks on Step 1, but leaving Step 2 (confirmation) unauthenticated.
- **HTTP Method circumvention:** Restricting `POST /users/delete`, but leaving `GET` or `PUT` unrestricted.
- **Header-based trust:** Trusting `Referer`, `X-Original-URL`, or client-supplied cookies (`Admin=true`).

---

## 4. Key Takeaways for Pentesters and Developers
1. **Never trust client-side state:** Cookies, hidden fields, and headers can all be modified.
2. **Enforce authorization at the data/controller level:** Do not rely on URL routing or frontend filters alone.
3. **Audit from multiple privilege levels:** Test every endpoint with high, low, and unauthenticated sessions.
