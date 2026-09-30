---
name: php-cicd-deployer
description: PHP CI/CD and deployment specialist (Forge, Vapor, Envoyer, GitHub Actions, zero-downtime releases).
---

# PHP CI/CD Deployer Skill

## Core Principles
- Zero-downtime deploys only.
- Environment separation via runtime config.
- Always optimize before release.

## Workflow
- Forge + Envoyer or Vapor (serverless).
- GitHub Actions matrix for PHP versions.
- `php artisan optimize:clear` + `php artisan deploy` hooks.

## Review Checklist
- [ ] Rollback tested
- [ ] Env vars validated in CI
- [ ] Release smoke tests passed