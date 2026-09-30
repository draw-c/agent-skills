---
name: elixir-cicd-deployer
description: Elixir CI/CD and deployment specialist using releases, Fly.io, GitHub Actions, Docker, zero-downtime deploys, and production hardening.
---

# Elixir CI/CD Deployer Skill

## Core Principles
- Use OTP releases (`mix release`) for production.
- Zero-downtime and rolling restarts.
- Environment config via `runtime.exs`.
- Database migrations run safely on deploy.
- Observability with Telemetry + PromEx or similar.

## Deployment Workflow (2026 Preferred)
- Primary: Fly.io (excellent Elixir clustering, global regions, `fly launch` + Machines).
- Alternative: Docker + any orchestrator, Render, or self-hosted with systemd.
- GitHub Actions for CI: test, dialyzer, credo, then deploy.
- `mix phx.digest` for assets.
- Release command: `mix release --overwrite`

## Fly.io Specifics
- `fly launch` for Phoenix auto-detection.
- Clustering works out-of-the-box via WireGuard.
- Separate staging/production apps or branches.
- Automated migrations in deploy pipeline.

## Review Checklist
- [ ] `mix release` tested locally
- [ ] CI pipeline runs full test suite + analysis
- [ ] Rollback strategy verified
- [ ] Env vars and secrets handled correctly
- [ ] Migrations safe (no destructive changes without downtime plan)
- [ ] Monitoring/Telemetry configured

Follow these and deployments will be reliable, scalable, and repeatable.