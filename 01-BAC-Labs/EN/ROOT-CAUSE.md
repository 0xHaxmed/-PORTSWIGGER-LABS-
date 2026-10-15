# Broken Access Control — Root Cause Analysis

## 1. Executive Summary
Broken Access Control (BAC) vulnerabilities do not stem from cryptographic flaws or syntax injection bugs. Instead, they represent **architectural and logical failures** in how applications authenticate, authorize, and track session context throughout the request lifecycle.

## 2. Primary Root Causes

### 1. Missing Server-Side Authorization Checks (Confusing Authentication with Authorization)
Many applications verify that a user is logged in (Authentication), but fail to check if that user has permission to perform the specific requested operation on that specific resource (Authorization).
```python
# VULNERABLE: Checks authentication only
@app.route('/account/delete', methods=['POST'])
@login_required
def delete_account():
    target_user = request.form.get('user_id')
    db.delete_user(target_user) # Anyone logged in can delete anyone!
```

### 2. Client-Controlled Authorization State (Mass Assignment & Tampering)
Storing privilege flags in client-readable and client-writable locations (such as cookies, local storage, hidden inputs, or unvalidated JSON request bodies) allows users to elevate their privileges by simple parameter modification.

### 3. Edge vs. Origin Divergence (Confused Deputy via Headers)
Reverse proxies, load balancers, and web application firewalls (WAFs) often evaluate access control rules based on the request URL. If the backend origin application re-interprets the URL based on internal headers (such as `X-Original-URL` or `X-Rewrite-URL`), the edge security layer is completely blind to the true backend target.

### 4. Assumption of Sequential Execution (Multi-Step Workflows)
Developers often build wizards or multi-stage forms where Step 1 performs permission validation, and subsequent steps assume the user was already vetted. Because HTTP is stateless, any step can be called directly out of order.

### 5. Flawed Route-Specific vs. Handler-Specific Guards
Decorators and filters applied strictly to specific HTTP verbs (e.g., `@RequireAdmin on POST`) allow attackers to trigger the underlying controller logic using other verbs (like `GET`, `PUT`, or verb tunneling).
