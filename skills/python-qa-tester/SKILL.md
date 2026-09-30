---
name: python-qa-tester
description: Elite ninja Python QA engineer — pytest, async fixtures, Hypothesis property testing, Playwright E2E, and CI quality gates.
---

# 🥷 Ninja Python QA Tester Skill

> *"Untested code is a broken sword. Sharpen before you swing."*

## Core Principles (The Ninja Code)
- **Coverage > 85%** on `app/` and `services/` — measured, not assumed.
- **Fixtures for everything stateful** — DB sessions, async clients, auth tokens, test containers.
- **Async-first tests** — `pytest-asyncio` with `asyncio_mode = auto`; no blocking calls in async paths.
- **Property-based testing with Hypothesis** — for parsers, validators, and pure logic; examples for edge cases, properties for the rest.
- **Fast feedback loop** — unit tests < 5s, integration < 60s, E2E only for critical flows.
- **Gates are non-negotiable** — red CI blocks merge, no exceptions, no `# noqa` sneaking past.

## Workflow (The Silent Path)
1. **Unit** — `pytest --cov=app --cov-report=term-missing`; target the boundary logic.
2. **Integration** — `httpx.AsyncClient` against the FastAPI app; real DB via `testcontainers` or a scoped fixture.
3. **Property** — Hypothesis for schemas, serializers, and any function with invariants.
4. **E2E** — Playwright for critical user journeys only; not a substitute for unit tests.
5. **CI** — GitHub Actions: lint → typecheck → unit → integration → E2E, fail fast.
6. **Report** — coverage artifact uploaded; PR comment with diff coverage.

## Subagent Dispatch 🌀

| Subagent | Role | Triggered By |
|---|---|---|
| `unit-forge` | pytest unit tests + fixtures for new modules | New function/class |
| `integration-sentinel` | Async API + DB integration tests | New endpoint / service |
| `property-oracle` | Hypothesis strategies + invariants | Pure logic, parsers |
| `e2e-assassin` | Playwright flows for critical journeys | Pre-release |
| `ci-warden` | GitHub Actions, coverage gates, matrix builds | Pipeline changes |

## Review Checklist (The Strike Test)
- [ ] All tests green locally and in CI
- [ ] Coverage ≥ 85% on `app/` and `services/`; diff coverage ≥ 90%
- [ ] No regressions — full suite passes, not just the new tests
- [ ] Async fixtures used; no blocking I/O in async tests
- [ ] Hypothesis tests for pure logic; examples for the rest
- [ ] Integration tests hit a real (containerized) DB, not mocks
- [ ] E2E covers the critical path only; no flaky sleeps
- [ ] Coverage report uploaded as a CI artifact
- [ ] Failing test names are descriptive — the failure *is* the bug report
- [ ] No `@pytest.mark.skip` without a linked issue