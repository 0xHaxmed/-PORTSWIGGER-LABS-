# Information Disclosure — Root Cause Analysis

## 1. Executive Summary
Information disclosure vulnerabilities almost never stem from complex algorithmic failures. Instead, they are the result of **configuration drift, developer oversights, and lack of automated deployment hygiene**.

## 2. Primary Root Causes

### 1. Production Running in Debug / Development Mode
Frameworks like Django, Express, and Spring have built-in diagnostic modes that display detailed stack traces, SQL queries, and environment dumps when an error occurs. Leaving `DEBUG=True` in production transforms any simple runtime exception into a full internal reconnaissance report.

### 2. Failure to Sanitize Error Handling
Code that catches exceptions often re-throws or serializes the raw error directly into HTTP response bodies rather than presenting a generic, user-friendly error page.
```java
// VULNERABLE: Direct stack trace serialization
catch (Exception e) {
    response.getWriter().write(e.toString());
    e.printStackTrace(response.getWriter());
}
```

### 3. Insecure Deployment and Artifact Packaging
Copying entire workspace folders into production web roots includes non-runtime artifacts:
- `.git/` directory
- `.env` files containing secrets
- Editor temporary backups (`ProductTemplate.java.bak`, `index.php~`)

### 4. Default Server Configurations & Diagnostic Methods
Web servers (Apache, Nginx, IIS) with default configurations that leave directory listing enabled, or allow diagnostic methods like `TRACE` without evaluating the security implications of reverse proxy headers.
