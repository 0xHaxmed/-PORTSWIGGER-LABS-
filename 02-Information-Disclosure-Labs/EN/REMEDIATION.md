# Information Disclosure — Prevention & Remediation Guide

## 1. Golden Rules for Preventing Information Leakage

1. **Disable Debug Mode in Production:** Ensure `DEBUG=False` and production error handlers are strictly configured.
2. **Implement Generic Error Pages:** Return generic HTTP status codes (400, 404, 500) with sanitized, uniform messages (e.g., "An unexpected error occurred. Please contact support.").
3. **Automate Secret Scanning:** Use Git pre-commit hooks (TruffleHog, Gitleaks) to prevent committing secrets to version control.
4. **Sanitize Build Artifacts:** Configure CI/CD pipelines to exclude `.git/`, `.env`, backup extensions (`.bak`, `.swp`, `.old`), and test files from production Docker images.

## 2. Server Hardening Configurations

### Apache: Disable TRACE and Directory Browsing
```apache
# In httpd.conf
TraceEnable Off
Options -Indexes
```

### Nginx: Block Sensitive Directories & Extensions
```nginx
location ~ /\.(git|env|svn) {
    deny all;
    return 404;
}

location ~* \.(bak|old|orig|save|swp)$ {
    deny all;
    return 404;
}
```
