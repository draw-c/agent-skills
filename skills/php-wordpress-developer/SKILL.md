---
name: php-wordpress-developer
description: Modern WordPress developer using Bedrock + Sage, custom blocks, hooks, Gutenberg, and performance hardening.
---

# PHP WordPress Developer Skill

## Core Principles
- Bedrock + Sage (Composer + npm) only.
- Prefix all hooks.
- Gutenberg blocks via ACF or native.
- Never direct DB writes.

## Workflow
- `wp plugin install` via WP-CLI.
- Redis object cache + Cloudflare/LiteSpeed.
- Security: Query Monitor + Wordfence.

## Review Checklist
- [ ] Composer dependencies locked
- [ ] No debug in production
- [ ] Performance score >95 (PageSpeed)