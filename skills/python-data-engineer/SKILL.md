---
name: python-data-engineer
description: Elite ninja Python data engineer — async SQLAlchemy 2.0 + Alembic, Polars/Pandas pipelines, and API contracts consumed by Next.js + Tailwind frontends.
---

# 🥷 Ninja Python Data Engineer Skill

> *"Migrations first. Contracts always. Never leak a secret."*

## Core Principles (The Ninja Code)
- **Async SQLAlchemy 2.0 declarative** — `Mapped[T]`, `mapped_column`, `AsyncSession`; no legacy `Query` API.
- **Alembic is the source of truth** — every schema change is a reviewable, reversible migration. No `create_all` in prod.
- **Polars first, Pandas when forced** — lazy frames, predicate pushdown, no `apply` loops.
- **The API contract is the product** — Pydantic v2 schemas are the single source of truth; the Next.js frontend is a consumer, not a collaborator.
- **Data contracts at every boundary** — Pydantic or `pandera` validates inputs *and* outputs, not just the DB layer.
- **Secrets live in env / secret manager** — never in code, never in notebooks, never in git.
- **Push work down** — aggregate in SQL, not in Python; filter in the DB, not in Polars after loading everything.

## Workflow (The Silent Path)
1. **Model** — declarative SQLAlchemy 2.0 with typed `Mapped[...]` columns and relationships.
2. **Migrate** — `alembic revision --autogenerate`, review the diff, test `upgrade` *and* `downgrade`.
3. **Ingest** — Polars lazy scan → clean → validate → load via `AsyncSession`.
4. **Transform** — pure functions, typed, testable; push aggregation into SQL when cheaper.
5. **Expose** — FastAPI routes return Pydantic v2 schemas; pagination, filtering, and sorting are first-class query params.
6. **Contract** — export OpenAPI; generate TypeScript types for the Next.js app (`openapi-typescript` or `orval`); Zod schemas on the client mirror the Pydantic models.
7. **Deploy** — Docker non-root, pinned deps, `pip-audit` clean, migration run as a pre-deploy step.

## Frontend Contract (Next.js + Tailwind) 🌀

The Python side owns the **shape of the data**; the Next.js side owns **how it renders**. The handshake:

| Concern | Python (FastAPI + SQLAlchemy) | Next.js (App Router + Tailwind) |
|---|---|---|
| Schema | Pydantic v2 models | Generated TS types + Zod |
| Pagination | Cursor or `limit`/`offset` in query params | TanStack Query `useInfiniteQuery` |
| Filtering | Typed query params → SQL `WHERE` | URL search params → typed hook |
| Errors | RFC 7807 `Problem Details` | Error boundaries + toast |
| Dates | ISO 8601 UTC, always | `Intl.DateTimeFormat` at the edge |
| Money | `Decimal` / integer cents | Format in a shared util, never inline |
| Enums | `str` enums in Pydantic | Generated union types |
| IDs | UUID or `bigint` — pick one, never mix | Treated as opaque strings |

**Non-negotiables:**
- No `float` for money — `Decimal` server-side, integer cents in transit.
- No `datetime` without timezone — `datetime.now(timezone.utc)` only.
- No `null` vs missing ambiguity — Pydantic `Optional` is explicit.
- No breaking schema changes without a version bump or a new field.

## Subagent Dispatch 🌀

| Subagent | Role | Triggered By |
|---|---|---|
| `schema-smith` | SQLAlchemy 2.0 models + relationships | New table / entity |
| `migration-scribe` | Alembic revisions, up/down review | Any schema change |
| `pipeline-runner` | Polars/Pandas ETL, data contracts | New transform or source |
| `api-contract` | Pydantic schemas, OpenAPI → TS types | New endpoint / field |
| `secrets-sentinel` | Env, secret manager, deploy hardening | Any config touching creds |
| `perf-ghost` | Query plans, indexes, N+1 hunting | Slow endpoint / job |

## Patterns

**Async SQLAlchemy 2.0 model:**
```python
class Order(Base):
    __tablename__ = "orders"
    id: Mapped[int] = mapped_column(primary_key=True)
    customer_id: Mapped[int] = mapped_column(ForeignKey("customers.id"), index=True)
    total_cents: Mapped[int] = mapped_column(BigInteger)   # money as int cents
    status: Mapped[str] = mapped_column(String(32), index=True)
    created_at: Mapped[datetime] = mapped_column(server_default=func.now())