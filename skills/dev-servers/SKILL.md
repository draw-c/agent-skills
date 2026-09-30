---
name: dev-servers
description: Manage, stop, start, and restart the full-stack development environment (FastAPI backend on port 8000 and Next.js frontend on port 3001). Use whenever the user asks to start, stop, restart, or check the status of local servers or IDE services.
---

# Dev Servers Management Runbook

This project runs two primary development services:
1. **FastAPI Backend**: Runs on `http://localhost:8000` with DuckDB and dynamic scheduler.
2. **Next.js Frontend**: Runs on `http://localhost:3001` as an SPA.

## Quick Actions

### 1. Restart Dev Servers (Backend + Frontend)
```bash
./scripts/restart_dev.sh --daemon
# or
make restart
```

### 2. Stop Dev Servers
```bash
./scripts/stop_dev.sh
# or
make stop
```

### 3. Start Dev Servers
```bash
./scripts/start_dev.sh --daemon
# or
make start
```

### 4. VS Code / Antigravity IDE Tasks
Users can execute these directly within Antigravity IDE:
- Open Command Palette (<kbd>Cmd</kbd>+<kbd>Shift</kbd>+<kbd>P</kbd>) -> **Tasks: Run Task** -> select **Restart Dev Servers (Backend + Frontend)**
- Or run the default build task shortcut: <kbd>Cmd</kbd>+<kbd>Shift</kbd>+<kbd>B</kbd>
