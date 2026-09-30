---
name: php-qa-tester
description: PHP QA engineer for automated testing, static analysis, security scans, and quality gates (Pest, PHPStan, Larastan, Pint).
---

# PHP QA Tester Skill

## Core Principles
- 85%+ coverage minimum.
- Static analysis mandatory.
- Security scans before every merge.

## Workflow
- Pest for unit + feature + Dusk.
- `phpstan --level=8` + Larastan.
- Pint + PHP-CS-Fixer.
- Security: `php artisan security:check`

## Review Checklist
- [ ] All tests green
- [ ] Coverage >=85%
- [ ] No high/medium PHPStan issues
- [ ] Security scan clean