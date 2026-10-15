# File Upload Vulnerabilities — Root Cause Analysis

## 1. Executive Summary
File upload vulnerabilities (CWE-434) occur when an application allows users to upload files to the filesystem without sufficiently validating their name, type, contents, or target directory, and then exposes those files to a runtime environment capable of executing them.

## 2. The Core Root Causes

### 1. Storage Inside an Executable Web Root
If uploaded files are written directly into a folder served by the web server (e.g. `/var/www/html/avatars/`), and the web server is configured to pass `.php`, `.asp`, or `.jsp` files to an interpreter, any uploaded script will execute immediately when requested via HTTP.

### 2. Trust in Client-Supplied MIME Types
Validation routines that check only the `Content-Type` header (e.g., ensuring it says `image/jpeg`) are trivial to bypass because the header is controlled entirely by the HTTP client (Burp Suite).

### 3. Incomplete Extension Blacklists
Blacklists that attempt to block `.php` frequently forget alternative extensions executed by PHP handlers:
- `.php3`, `.php4`, `.php5`, `.phtml`, `.phar`
- IIS handlers: `.asp`, `.aspx`, `.cer`, `.asa`
- Server config overrides: uploading `.htaccess` on Apache or `web.config` on IIS to reassign script execution to harmless extensions (`.l33t` or `.png`).

### 4. Time-of-Check Time-of-Use (TOCTOU) Race Conditions
Applications often save the incoming uploaded file to disk first, then run a virus scanner or image validation check, and delete the file if it fails. During the small time window between save and delete, an attacker can trigger the file via concurrent HTTP requests.
