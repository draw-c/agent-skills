---
name: elixir-qa-tester
description: Elixir QA engineer for testing, static analysis, property testing, coverage, and quality gates (ExUnit, Wallaby, Dialyxir, Credo, Hypothesis-style testing).
---

# Elixir QA Tester Skill

## Core Principles
- Tests are first-class: unit, integration, doctests, property-based.
- Static analysis mandatory before merge.
- Aim for high coverage with meaningful tests (not just 100% for its own sake).
- Use Mox for mocking external dependencies.

## Workflow
- ExUnit for everything core.
- Wallaby or Hound for browser/LiveView E2E.
- Property testing with StreamData or Hypothesis-style.
- `mix test --cover`
- Static tools: Credo, Dialyxir (Dialyzer), Sobelow (security).

## Common Commands
- `mix test`
- `mix credo`
- `mix dialyzer`
- `mix test --only integration`

## Review Checklist
- [ ] All tests green (`mix test`)
- [ ] Coverage reviewed and meaningful
- [ ] Credo and Dialyxir clean
- [ ] LiveView and endpoint tests included
- [ ] No flaky tests
- [ ] Security scan (Sobelow) passed