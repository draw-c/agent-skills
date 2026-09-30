# Agent Skills & Rules Library

A curated, modular library of AI agent skills, rules, and subagent orchestration patterns designed for pairing with AI coding assistants (Antigravity, Claude Code, Cursor, Copilot, and custom agentic frameworks).

---

## Overview

Each skill defines specialized domain expertise, non-negotiable principles, subagent dispatch matrices, and actionable review checklists. Agents automatically load relevant skills on-demand based on prompt context and task requirements.

---

## Catalog of Skills

### 🌐 Next.js & Modern Frontend
- **`nextjs-ninja-developer`**: App Router architecture, server-first defaults, subagent dispatching, and zero-layout-shift performance.
- **`nextjs-router`**: Route trees, layouts, parallel (`@slot`) and intercepting (`(.)`) routes, typed route params.
- **`nextjs-architect`**: Production-grade Next.js App Router audits, bundle optimization, and rendering boundaries.
- **`react-server-client`**: Strict Server vs Client Component boundaries, leaf-node `'use client'`, and props serialization.
- **`data-fetcher`**: Data wiring, Next.js cache strategies (`force-cache`, `revalidate`), TanStack Query boundaries, and Zod validation.
- **`ui-architect`**: Feature-driven folders, accessible headless primitives (Radix), and compound components.
- **`tailwind-stylist`**: Utility ordering, responsive design, class-based dark mode, and `cva` variant composition.
- **`tailwind-designer`**: Design systems, taste, restraint, accessibility, semantic color tokens, and OKLCH palettes.
- **`quality-guard`**: Automated release gates, strict TypeScript, ESLint, test coverage, and bundle analysis.

### 🐍 Python & Backend Engineering
- **`python-fastapi-developer`**: High-performance async APIs, Pydantic v2 validation, dependency injection, and subagent orchestration.
- **`python-data-engineer`**: Async SQLAlchemy 2.0, Alembic migrations, Polars/Pandas ETL, and strict API-to-frontend contracts.
- **`python-qa-tester`**: `pytest-asyncio`, containerized integration tests, Hypothesis property testing, and coverage gates.
- **`python-testing-engineer`**: Test automation patterns, fixtures, and CI pipelines.
- **`python-cicd-deployer`**: GitHub Actions workflows, Docker containers, security scans, and automated releases.
- **`python-scraping-specialist`**: Anti-bot evasion, TLS fingerprinting, proxy rotation, and resilient extraction.
- **`duckdb-architect-elite`**: High-performance analytical queries, Parquet/S3 integration, and vectorized in-process data modeling.

### 🎭 Browser Testing & Process Hygiene
- **`ninja-playwright`**: OS/kernel-level process hygiene, macOS/Linux shell traps, Windows Job Objects (`KILL_ON_JOB_CLOSE`), session isolation, and prevention of orphaned Chromium processes.
- **`ninja-playwright-testing`**: Test craft: semantic locators (`getByRole`), auto-waiting web-first assertions, `storageState` auth reuse, network mocking, and visual regression.

### 🐘 PHP Ecosystem
- **`php-architect`**: System design, OWASP security baselines, caching topologies, read replicas, and zero-downtime deploys.
- **`php-laravel-developer`**: Resource controllers, Eloquent discipline, Form Requests, queued jobs with Horizon, Pest, and Larastan.
- **`php-statamic-developer`**: Git-first flat-file CMS, Antlers templating, blueprints, collections, forms, and static caching.
- **`php-wordpress-developer`**: Modern WordPress with Bedrock + Sage, Gutenberg blocks, and performance hardening.
- **`php-qa-tester`**: Pest testing, PHPStan Level 8, Larastan, and automated quality gates.
- **`php-cicd-deployer`**: Zero-downtime deployment pipelines, Forge/Vapor, and release management.

### ⚡ Elixir & Phoenix
- **`elixir-architect`**: OTP design, supervision trees, fault-tolerance, and distributed system scalability.
- **`elixir-phoenix-developer`**: LiveView, Ecto contexts, real-time channels, and full-stack Phoenix applications.
- **`elixir-qa-tester`**: ExUnit test suites, Wallaby integration testing, Dialyxir, and Credo static analysis.
- **`elixir-cicd-deployer`**: OTP releases, Fly.io deployments, Docker, and zero-downtime rollouts.

### 📊 Data Apps & Quant
- **`streamlit`**: Comprehensive Streamlit application structure, caching, session state, and reactive widgets.
- **`streamlit-shadcn`**: Enterprise-grade, high-density Streamlit dashboards using shadcn-inspired design primitives.
- **`mojo-quant`**: High-performance quantitative finance algorithms and hardware-accelerated computation.

### 🛠️ Infrastructure & Dev Servers
- **`dev-servers`**: Unified background server management for dual-stack development environments.

---

## How to Use in Your Projects

### Option 1: Git Submodule (Recommended)
Add this repository as the `.agents` directory directly in any project:
```bash
git submodule add https://github.com/draw-c/agent-skills.git .agents
```

To update to the latest skills anytime:
```bash
git submodule update --remote .agents
```

### Option 2: Global Configuration (Antigravity)
Make all skills available across all workspaces on your machine by cloning directly into your global configuration:
```bash
# Clone directly into Antigravity global skills
git clone https://github.com/draw-c/agent-skills.git ~/.gemini/config/global-agent-skills
ln -s ~/.gemini/config/global-agent-skills/skills ~/.gemini/config/skills
```

---

## Security & Privacy Notice

This repository strictly contains **prompts, architectural guides, and workflow rules**. 
- **Never commit MCP credentials or API keys**: Keep `mcp_config.json` and `.env` files in your local project configurations or secure secret managers.
- `mcp_config.json` is explicitly git-ignored.

---

## License

MIT License. Feel free to use, adapt, and share across your projects.
