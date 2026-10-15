# OS Command Injection — Prevention & Remediation Guide

## 1. Primary Defense: Avoid Spawning Shells Entirely
The most effective way to eliminate command injection is to avoid calling shell commands directly from application code. Use built-in programming language APIs instead:
- Instead of calling `rm -rf`, use `os.remove()` or `Files.delete()`.
- Instead of calling `ping`, use an ICMP library.

## 2. Parameterized Execution (No Shell Flag)
If invoking an external process is strictly unavoidable, use process execution APIs that pass arguments as an array/vector directly to the executable **without spawning a system shell**:

### Secure Implementation in Python:
```python
import subprocess

# SECURE: shell=False (default). Arguments passed as array.
# Special characters like ';' or '|' are treated as literal strings, not shell syntax.
result = subprocess.run(["/usr/local/bin/check_stock", store_id, product_id],
                        capture_output=True,
                        text=True,
                        shell=False)
```

### Secure Implementation in Node.js:
```javascript
const { execFile } = require('child_process');

// SECURE: execFile does not spawn a shell
execFile('/usr/local/bin/check_stock', [storeId, productId], (error, stdout, stderr) => {
    // Process output
});
```

## 3. Strict Input Validation (Allow-lists)
If a parameter represents a numeric ID or fixed code:
- Validate that it matches an exact regular expression: `^[0-9]+$`
- Reject any request containing shell characters (`&`, `;`, `|`, `` ` ``, `$`, `>`, `<`, `
`).
