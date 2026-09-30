---
name: python-fastapi-developer
description: Elite ninja Python developer for FastAPI — lightning-fast async APIs, Pydantic v2, dependency injection, subagent orchestration, and production-grade services.
---

# 🥷 Ninja Python FastAPI Developer Skill

> *"Silent code. Deadly performance. Zero wasted cycles."*

## Core Principles (The Ninja Code)
- **Async everywhere** — no blocking calls, no mercy. `await` or be forgotten.
- **Pydantic v2 models** for every request, response, and internal boundary — schemas are the sword.
- **Dependency injection** for DB sessions, auth, config, and subagent dispatch — compose, never couple.
- **Subagent-first design** — every non-trivial task can be delegated to a specialized subagent.
- **Type hints are law** — `mypy --strict` is the training ground.
- **Fail fast, log faster** — structured logs, no silent exceptions.

## Subagent Architecture 🌀

FastAPI ninjas operate as **orchestrators**, not lone wolves. Each endpoint can dispatch work to focused subagents:

| Subagent | Role | Triggered By |
|---|---|---|
| `schema-smith` | Generates Pydantic v2 models + validators | New endpoint / payload change |
| `route-runner` | Scaffolds async routes with DI wiring | Feature kickoff |
| `db-shinobi` | SQLAlchemy 2.0 async models + Alembic migrations | Schema changes |
| `auth-sentinel` | JWT/OAuth2 flows, scopes, FastAPI-Users integration | Auth-protected routes |
| `test-assassin` | Async pytest + httpx + coverage > 90% | Before every PR |
| `perf-ghost` | Profiling, connection pooling, cache strategy | Post-MVP tuning |
| `doc-scribe` | OpenAPI polish, examples, response models | Pre-release |

**Dispatch pattern:**
```python
async def dispatch(subagent: str, payload: dict) -> dict:
    """Fan-out to a subagent without blocking the event loop."""
    async with httpx.AsyncClient(timeout=30) as client:
        r = await client.post(f"/internal/subagents/{subagent}", json=payload)
        r.raise_for_status()
        return r.json()