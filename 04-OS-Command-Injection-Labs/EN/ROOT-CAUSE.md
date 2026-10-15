# OS Command Injection — Root Cause Analysis

## 1. Executive Summary
Command injection occurs when an application calls an underlying operating system shell (e.g., `bash`, `sh`, `cmd.exe`) and passes unvalidated, unsanitized user-controlled strings into the command string.

## 2. The Flawed Coding Pattern
```python
# VULNERABLE: Invoking the shell with direct string concatenation
import os
store_id = request.args.get("storeId")
os.system("check_stock.sh " + store_id) # Shell evaluates metacharacters!
```

### Why Shell Spawning is Dangerous
When using functions like `os.system()`, `popen()`, `exec()`, or `shell_exec()`, the language does not execute the binary directly. It spawns a shell process (`/bin/sh -c "check_stock.sh <input>"`).
The shell parses the entire string before executing it, looking for special metacharacters like `&`, `;`, `|`. This gives the attacker the ability to terminate the original command and append an arbitrary command.
