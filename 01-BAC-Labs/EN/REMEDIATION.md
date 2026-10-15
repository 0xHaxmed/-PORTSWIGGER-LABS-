# Broken Access Control — Prevention & Remediation Guide

## 1. Core Principles of Secure Access Control

1. **Deny by Default:** Unless a resource or action is explicitly marked as public, access must be blocked.
2. **Enforce on Every Request:** Do not cache authorization decisions across requests or assume sequential steps.
3. **Decouple Access Control from the UI:** Hiding buttons or routes in HTML/JavaScript is a UX decision, never a security barrier.

## 2. Technical Remediation Patterns

### A. Centralized Authorization Middleware
Instead of scattering ad-hoc `if (role == 'admin')` checks across controllers, implement a centralized policy engine or middleware layer.
```python
# SECURE: Policy-based check
@app.route('/documents/<doc_id>', methods=['GET'])
@login_required
def get_document(doc_id):
    doc = db.get_document(doc_id)
    if not doc:
        abort(404)
    # Validate that current session owns or has explicit permission on doc
    if doc.owner_id != current_user.id and not current_user.has_permission('view_all_documents'):
        abort(403)
    return render_template('doc.html', doc=doc)
```

### B. Indirect Object References (Mapping or UUIDs)
To mitigate IDOR, avoid exposing sequential database integers. Use random, unguessable UUIDv4 values, or session-bound temporary ID mapping tables.

### C. Reject Client-Supplied Role Parameters (Disable Mass Assignment)
Explicitly declare data transfer objects (DTOs) with strict allow-lists of fields that users are permitted to update.
```csharp
// SECURE: Strict DTO pattern prevents tampering with role/permissions
public async Task<IActionResult> UpdateProfile([FromBody] UserProfileUpdateDto dto) {
    user.Email = dto.Email;
    user.FullName = dto.FullName;
    // roleId is never mapped from incoming payload
    await _db.SaveChangesAsync();
}
```

### D. Secure Reverse Proxy Configurations
Disable support for legacy routing override headers (`X-Original-URL`, `X-Rewrite-URL`, `X-Forwarded-Prefix`) unless strictly required and sanitized before the origin application receives the request.

---

## 3. Automated Testing Checklist
- [ ] Implement unit and integration tests that verify unauthenticated requests receive `401 Unauthorized`.
- [ ] Implement tests that verify low-privileged users receive `403 Forbidden` on admin endpoints.
- [ ] Integrate Burp extensions such as **Autorize** into manual penetration testing workflows.
