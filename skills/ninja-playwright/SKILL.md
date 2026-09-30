---
name: ninja-playwright
description: Elite ninja Playwright specialist for Python and JS — cross-platform cleanup (macOS/Linux/Windows), session isolation, and no orphaned browsers.
---

# 🥷 Ninja Playwright Skill

> *"Close the browser. Always. No exceptions."*

## Core Principles (The Ninja Code)

- **`browser.close()` in a `finally` block — every script.** This handles clean exits and exceptions. It does **not** handle killed parents, which is your actual problem.
- **A killed parent runs no cleanup code.** When Gemini's session is killed from the IDE, the parent process dies before `__exit__` or `finally` can run. No amount of in-process code can fix this.
- **The fix must live outside the process.** macOS/Linux use shell traps; Windows uses kernel-level Job Objects. This is non-negotiable.
- **Headless by default.** Headed mode steals focus on macOS and worsens orphaned-process impact.
- **One session per agent.** Use unique session names (`-s=gemini-session`, `--isolated`, unique `userDataDir`) so parallel agents never collide.
- **Pre-flight and post-flight sweeps.** Clear leftovers before starting; verify cleanup after finishing.

## The Root Cause (Why This Happens)

When a parent process (Gemini's node worker) is killed externally, **no cleanup code runs at all**. Playwright's own `__exit__` handles clean exits and exceptions correctly — the leak only happens on a **killed parent**.

On macOS, this manifests as `replayd` pegged at ~150% CPU, battery drain, and IDE unresponsiveness. The orphaned Chromium workers hold screen-capture resources, which is why your IDE suffers.

**The fix must live in the kernel or the shell — not in your Python/JS code.**

## Cross-Platform Cleanup

### macOS

Use a shell trap that fires on `EXIT`, `INT`, and `TERM`:

```bash
#!/usr/bin/env bash
set -euo pipefail

cleanup() {
    echo "Ninja cleanup: killing orphaned Playwright processes..."
    pkill -f "headless_shell" 2>/dev/null || true
    pkill -f "chromium" 2>/dev/null || true
    pkill -f "chrome" 2>/dev/null || true
}
trap cleanup EXIT INT TERM

# Pre-flight: clear any leftovers from prior runs
pkill -f "headless_shell" 2>/dev/null || true

python my_test.py
```

**Limitation:** The trap does **not** fire on `SIGKILL`. If the parent is force-killed, orphans survive.

**Immediate fix for stuck processes:**
```bash
pkill -f "playwright" 2>/dev/null
pkill -f "headless_shell" 2>/dev/null
pkill -f "chromium" 2>/dev/null
pkill -f "chrome" 2>/dev/null
```

### Linux

Linux supports **process groups**, which are more reliable than name-based `pkill`:

```bash
#!/usr/bin/env bash
set -euo pipefail

# Re-launch as process group leader if not already
pgid=$(ps -o pgid= -p "$$")
[ $$ -eq $pgid ] || exec setsid --wait "${BASH_SOURCE[0]}" "$@"

trap "kill -- -$pgid 2>/dev/null || true" EXIT INT TERM

# Pre-flight sweep
pkill -f "headless_shell" 2>/dev/null || true

python my_test.py
```

The `kill -- -$pgid` sends the signal to **every process in the group** — browser, content processes, node workers. This doesn't depend on process-name matching.

**WSL note:** Orphaned `headless_shell` processes inside WSL can balloon `vmmemWSL` to several GB, slowing the entire Windows host. Use the same `pkill` pattern, then `wsl --shutdown` to reclaim memory.

**Immediate fix:**
```bash
pkill -f "headless_shell" 2>/dev/null
pkill -f "chrome --headless" 2>/dev/null
pkill -f "playwright" 2>/dev/null
# Force kill after 3 seconds
sleep 3
pgrep -f "headless_shell|chrome|playwright" | xargs -r kill -9
```

### Windows

Windows has no `pkill` and no process groups. The correct fix is a **Job Object** with `JOB_OBJECT_LIMIT_KILL_ON_JOB_CLOSE`. The kernel tears down every process in the job when the parent's handle closes — **even on a crash or external kill**.

**Python (using `win32job`):**

```python
import subprocess
import win32api
import win32con
import win32job

# Create a job that kills all members when the last handle closes
hJob = win32job.CreateJobObject(None, "")
info = win32job.QueryInformationJobObject(
    hJob, win32job.JobObjectExtendedLimitInformation
)
info['BasicLimitInformation']['LimitFlags'] = (
    win32job.JOB_OBJECT_LIMIT_KILL_ON_JOB_CLOSE
)
win32job.SetInformationJobObject(
    hJob, win32job.JobObjectExtendedLimitInformation, info
)

child = subprocess.Popen(["python", "my_test.py"])
# Assign the child to the job — all descendants join automatically
perms = win32con.PROCESS_TERMINATE | win32con.PROCESS_SET_QUOTA
hProcess = win32api.OpenProcess(perms, False, child.pid)
win32job.AssignProcessToJobObject(hJob, hProcess)
```

**PowerShell one-liner for stuck orphans:**
```powershell
Get-Process | Where-Object { $_.ProcessName -match 'headless|chromium|chrome' } | Stop-Process -Force
```

**Limitation:** Windows is the **only** platform with a kernel-enforced guarantee that survives a killed parent. macOS and Linux rely on the shell trap firing, which fails on `SIGKILL`.

### Cross-Platform Summary Table

| Platform | Mechanism | Fires on SIGKILL? | Code needed |
|---|---|---|---|
| macOS | Shell trap + pre-flight `pkill` | No | Shell wrapper |
| Linux | Process group trap (`kill -- -PGID`) | No | Shell wrapper |
| Windows | Job Object `KILL_ON_JOB_CLOSE` | **Yes** | Python `win32job` or wrapper |

## Session Management (Prevent Collisions)

When Gemini runs alongside other agents, use **named sessions**:

```bash
# Gemini's session
playwright-cli -s=gemini-session open https://example.com
playwright-cli -s=gemini-session close    # closes ONLY this session

# If it gets stuck:
playwright-cli close-all                  # close every browser session
playwright-cli kill-all                   # force-kill all playwright-cli daemons
```

The `close` command closes a named session's browser; `close-all` handles every session.

For MCP-based agents, use `--isolated` and unique `userDataDir` per agent:

```json
{
  "mcpServers": {
    "playwright-gemini": {
      "command": "npx",
      "args": ["@anthropic-ai/mcp-playwright@latest", "--isolated", "--headless", "--userDataDir", "/tmp/pw-gemini"]
    }
  }
}
```

## Python + JS Patterns

**Python (async, safest for FastAPI context):**

```python
import asyncio
from playwright.async_api import async_playwright

async def run():
    async with async_playwright() as p:
        browser = await p.chromium.launch(headless=True)
        try:
            page = await browser.new_page()
            await page.goto("https://example.com")
            # ... ninja steps ...
        finally:
            await browser.close()       # never skip this
            await p.stop()              # explicit playwright teardown
```

**Python (pytest fixture pattern):**

```python
@pytest.fixture
def browser_page():
    with sync_playwright() as p:
        browser = p.chromium.launch()
        page = browser.new_page()
        yield page
        page.close()
        browser.close()                 # yields cleanup even on test failure
```

**JavaScript (Node, with hard timeout):**

```javascript
const { chromium } = require('playwright');

(async () => {
    const browser = await chromium.launch({ headless: true });
    const timeout = setTimeout(async () => {
        console.error("Ninja timeout — forcing close");
        await browser.close();
        process.exit(1);
    }, 30000);  // 30s hard kill

    try {
        const page = await browser.newPage();
        await page.goto("https://example.com");
        // ... ninja steps ...
    } finally {
        clearTimeout(timeout);
        await browser.close();
    }
})();
```

**Python shell wrapper with trap (recommended):**

```bash
#!/usr/bin/env bash
set -euo pipefail

cleanup() {
    pkill -f "headless_shell" 2>/dev/null || true
    pkill -f "chromium" 2>/dev/null || true
}
trap cleanup EXIT INT TERM

pkill -f "headless_shell" 2>/dev/null || true
python -m pytest tests/ -v
```

## Review Checklist (The Strike Test)

- [ ] `browser.close()` in a `finally` block — every script
- [ ] `p.stop()` (Python) called explicitly after `browser.close()`
- [ ] Shell trap or process wrapper kills orphans on `SIGTERM`/`SIGINT`
- [ ] Session name is unique per agent (`-s=gemini-session`, `--isolated`)
- [ ] Headless mode by default; headed only when anti-bot forces it
- [ ] Pre-flight `pkill -f headless_shell` if prior runs may have leaked
- [ ] Post-flight verification: `ps | grep playwright` returns nothing
- [ ] `replayd` CPU returns to 0% after session ends (macOS)
- [ ] No `--user-data-dir` collisions between parallel agents
- [ ] Windows: Job Object with `KILL_ON_JOB_CLOSE` if running in CI or agent mode

## Immediate Fix for Your Mac

Run this **right now** to clear the current orphans:

```bash
# Kill everything Playwright-related
pkill -f "playwright" 2>/dev/null
pkill -f "headless_shell" 2>/dev/null
pkill -f "chromium" 2>/dev/null
pkill -f "chrome" 2>/dev/null

# Verify they're gone
ps -axo pid,ppid,pcpu,comm,args | grep -i 'playwright\|headless\|chromium' | grep -v grep

# Check replayd CPU (should drop to 0%)
ps -axo pid,pcpu,comm | grep replayd
```

If `replayd` is still high after killing the Playwright processes, the node worker holding them is still alive — kill that parent too.

## Why This Happens with Gemini Specifically

Gemini (like Codex) spawns a **node worker** as the parent, and Playwright/Chromium as children. When the Gemini session ends or is killed from the IDE, the parent dies but the **children don't automatically die with it on macOS** — there's no job-object equivalent to Windows' `KILL_ON_JOB_CLOSE`. The fix must be **preventive** (trap + session cleanup) or **post-hoc** (`pkill`). There is no in-process fix for a killed parent.

For your IDE issues: the orphaned processes hold file handles and screen-capture resources, which is why the IDE becomes unresponsive. Once you `pkill` the orphans, the IDE should recover immediately.