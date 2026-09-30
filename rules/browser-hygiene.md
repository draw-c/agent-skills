# Browser Process Lifecycle & Resource Hygiene

## Critical Rule: Prevent IDE Freezes & Orphaned Playwright Drivers

1. **Avoid Unnecessary Browser Subagents**:
   - Do NOT launch `browser_subagent` or headless Playwright for routine tasks, code changes, or type checks.
   - Use fast static analysis (`npx tsc --noEmit`), linters, unit tests, or direct component inspection.
   - Only launch browser automation when the user explicitly requests visual browser testing or video/screenshot capture.

2. **Mandatory Immediate Teardown**:
   - Every single time `browser_subagent` or Playwright is invoked, the agent **MUST** run an immediate cleanup check in the same turn before replying:
     ```bash
     pkill -9 -f "ms-playwright" 2>/dev/null || true
     pkill -9 -f "cli.js run-driver" 2>/dev/null || true
     ```
   - Verify that no lingering Node driver or headless Chromium processes remain running with `ps aux | grep -i ms-playwright`.
   - Never assume the subagent framework cleans up its own child processes on macOS.

3. **Prevent Chrome DevTools Protocol (CDP) Deadlocks**:
   - Never spawn multiple concurrent processes or MCP tools attaching to the same Chrome/Electron debug port (`--autoConnect`).
   - Competing CDP clients freeze the Antigravity IDE Electron renderer and create dead `chrome-error://chromewebdata/` webviews.

4. **Accurate Route Navigation**:
   - Check the routing structure before navigating or telling the user to navigate. In SPAs, navigate to root (`/`) and switch views via UI state rather than generating 404 sub-routes that hang webviews.
