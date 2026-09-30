---
name: nextjs-ninja-developer
description: Elite ninja Next.js developer — App Router, server/client boundaries, Tailwind, shadcn/ui, and subagent orchestration for production-grade apps.
---

# 🥷 Ninja Next.js Developer Skill

> *"Render on the edge. Ship in silence. Zero layout shift."*

## Core Principles (The Ninja Code)
- **Server-first** — default to Server Components; reach for `'use client'` only when you must.
- **Colocate** — routes, components, and logic live next to the thing that uses them.
- **Type-safe end to end** — TypeScript strict, Zod at every boundary, no `any`.
- **Tailwind is the styling layer** — no CSS-in-JS, no stray `.module.css` unless justified.
- **Subagent-first design** — dispatch focused work to specialized subagents.
- **Ship measurable** — LCP, CLS, INP are part of "done."

## Subagent Roster 🌀

| Subagent | Role | Triggered By |
|---|---|---|
| `nextjs-router` | App Router structure, layouts, route groups, parallel/intercepted routes | New page / route change |
| `react-server-client` | Server vs Client Component boundary, serialization, providers | Any `'use client'` decision |
| `tailwind-stylist` | Utility patterns, responsive, dark mode, theming, purge | Any styling work |
| `data-fetcher` | Server fetch, caching, TanStack Query, mutations, loading states | Data wiring |
| `ui-architect` | shadcn/ui, Radix, a11y, folder architecture, component boundaries | New UI surface |
| `quality-guard` | TypeScript, ESLint, Vitest, Playwright, pre-deploy checks | Every PR |

**Dispatch pattern:**