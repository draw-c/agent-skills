---
name: elixir-phoenix-developer
description: Senior Phoenix developer for full-stack web apps with LiveView, Ecto, contexts, and real-time features. Use for controllers, views, components, authentication, and everyday Phoenix development.
---

# Elixir Phoenix Developer Skill

## Core Principles
- Always use contexts (`mix phx.gen.context`) — never put business logic in controllers.
- LiveView: keep assigns minimal, use components, streams for large lists.
- Ecto: changesets for all validation, never raw SQL in app code.
- Follow Elixir style + `mix format`.
- Phoenix 1.7+ patterns (verified routes, HEEx components).

## Workflow
1. New app: `mix phx.new app --live --tailwind`
2. Generators: `mix phx.gen.live`, `mix phx.gen.auth`, `mix phx.gen.context`
3. Forms: Ecto changesets + LiveView real-time validation
4. Real-time: Phoenix PubSub + Presence
5. Assets: Tailwind + esbuild or esbuild + Alpine if needed

## LiveView Best Practices
- Use `phx-debounce` and `phx-throttle` appropriately.
- Prefer server-side rendering; push only necessary updates.
- JS hooks only when truly needed (e.g., complex client interop).

## Review Checklist
- [ ] `mix credo` and `mix format` clean
- [ ] All tests pass (`mix test`)
- [ ] No business logic in controllers
- [ ] LiveView tests cover events and renders
- [ ] Routes use verified routes where possible