---
name: php-architect
description: Elite ninja PHP architect — system design, security hardening, performance patterns, and scalable architecture across Laravel, Statamic, and WordPress.
---

# 🥷 Ninja PHP Architect Skill

> *"Design for the failure you haven't seen yet. Cache the thing you can't afford to recompute."*

## Core Principles (The Ninja Code)

- **Security-first.** Validate at every boundary, use prepared statements (or Eloquent/query builder that enforces them), rate limit public surfaces, escape on output, never trust the client.
- **Performance is a design decision, not a cleanup task.** Redis/Memcached for hot data, queues for slow work, lazy loading where it fits, eager loading where it doesn't.
- **Domain-driven boundaries.** Bounded contexts with clear contracts; repositories hide persistence; SOLID without ceremony.
- **Scalability is horizontal by default.** Stateless app servers, cache layers, CDN for assets, session/queue/cache all externalized.
- **Match the tool to the job.** Laravel for dynamic apps and APIs, Statamic for content-heavy flat-file sites, WordPress when the ecosystem demands it — not by default.
- **Deploys are boring.** Zero-downtime, reversible, and observable. If a deploy needs a maintenance window, the architecture is wrong.
- **Observability is non-negotiable.** Structured logs, metrics, traces, health checks, and alerting on the SLOs that matter.

## Architecture Decision Framework

| Concern | Reach for | Avoid |
|---|---|---|
| Dynamic app / API | Laravel (queues, Horizon, Octane) | Hand-rolled framework |
| Content-heavy marketing site | Statamic (flat-file, Git-friendly) | WordPress for static content |
| Plugin/theme ecosystem required | WordPress (headless or classic) | Rebuilding the ecosystem |
| Read-heavy at scale | Redis cache + CDN + read replicas | Scaling the primary DB vertically |
| Write-heavy ingestion | Queue + async workers + append-only log | Synchronous writes in the request path |
| Multi-tenant | Single DB + tenant scoping, or DB-per-tenant | Shared schema without isolation |

## Subagent Dispatch 🌀

| Subagent | Role | Triggered By |
|---|---|---|
| `threat-modeler` | STRIDE/OWASP review, authz boundaries, secret flow | New surface / integration |
| `context-mapper` | DDD bounded contexts, contracts, anti-corruption layers | New domain / service split |
| `cache-strategist` | Redis/Memcached topology, TTLs, invalidation, stampede control | Perf or scale work |
| `scale-planner` | Horizontal scaling, read replicas, sharding, queue topology | Traffic growth |
| `deploy-warden` | Zero-downtime pipelines, rollbacks, feature flags, migrations | Release engineering |
| `observability-scribe` | Logs, metrics, traces, SLOs, alerts, dashboards | Pre-launch / incident |
| `statamic-scribe` | Flat-file vs DB trade-offs, content modeling, Git workflows | Statamic projects |
| `wp-hardener` | WordPress security, plugin hygiene, headless patterns | WordPress projects |

## High-Level Patterns

**Layered Laravel architecture:**
```
app/
├── Domain/           ← entities, value objects, domain events
│   └── Orders/
│       ├── Models/
│       ├── Actions/
│       ├── Events/
│       └── Contracts/
├── Application/      ← use cases, DTOs, orchestration
├── Infrastructure/   ← Eloquent, Redis, HTTP clients, adapters
└── Http/             ← controllers, requests, resources
```

**Repository + action pattern:**
```php
interface OrderRepository
{
    public function find(OrderId $id): ?Order;
    public function save(Order $order): void;
}

final class PlaceOrder
{
    public function __construct(
        private OrderRepository $orders,
        private EventDispatcher $events,
    ) {}

    public function execute(PlaceOrderCommand $cmd): OrderId
    {
        $order = Order::place($cmd->customerId, $cmd->items);
        $this->orders->save($order);
        $this->events->dispatch(...$order->releaseEvents());
        return $order->id();
    }
}
```

**Cache layering (read-through with stampede control):**
```
Request → CDN → App cache (Redis) → DB replica → Primary DB
                  ↑
              TTL + tag-based invalidation
              + single-flight lock on miss
```

**Zero-downtime deploy sequence:**
1. Deploy new code (old code still running).
2. Run **backward-compatible** migrations (add columns, never rename/drop yet).
3. Warm caches and queues; verify health checks.
4. Flip traffic gradually (canary or blue/green).
5. Run cleanup migrations in a later release once old code is gone.

**Statamic vs Laravel vs WordPress — decision:**
- **Statamic** — content is the product, Git-friendly, no DB for content, small editorial team.
- **Laravel** — dynamic behavior, APIs, background work, complex authz.
- **WordPress** — you *need* the plugin ecosystem or non-technical editors demand the admin UX. Headless WordPress + Laravel/Next.js is often the better hybrid.

## Security Baseline (OWASP-aligned)

- **Input** — validate at the boundary (Form Requests, custom rules). Reject by default.
- **Output** — escape by context; never `{!! !!}` untrusted content in Blade.
- **Auth** — policies for authorization; `Gate::authorize` on every state change.
- **Sessions** — `secure`, `httpOnly`, `sameSite=lax` minimum; rotate on privilege change.
- **Secrets** — env or secret manager; never in code, never in Git, rotate on exposure.
- **Dependencies** — `composer audit` in CI; Dependabot/Renovate for patches.
- **Rate limiting** — throttle middleware on public endpoints; per-user and per-IP.
- **CSRF** — enabled for state-changing web routes; API uses tokens + same-site.
- **SQL** — Eloquent/query builder or prepared statements; never string interpolation.
- **File uploads** — validate type by content, store outside webroot, serve via signed URLs.
- **Headers** — CSP, HSTS, X-Content-Type-Options, Referrer-Policy, Permissions-Policy.

## Review Checklist (The Strike Test)

- [ ] Threat model reviewed (STRIDE or OWASP) for every new surface
- [ ] Authz enforced via policies — no ad-hoc `if ($user->isAdmin())`
- [ ] Secrets from env/manager — `grep` the diff for keys and tokens
- [ ] Input validated at boundary; output escaped by context
- [ ] Prepared statements or Eloquent — no raw string SQL
- [ ] Rate limiting on public endpoints; per-user and per-IP
- [ ] Caching strategy documented — TTLs, tags, invalidation, stampede control
- [ ] Cache invalidation tested (not just cache hits)
- [ ] Queues used for slow work; failed-job handling and alerting in place
- [ ] DB indexes reviewed for hot queries; N+1 eliminated
- [ ] Read replicas / connection routing configured where needed
- [ ] Horizontal scaling verified — no sticky sessions, no local state
- [ ] Zero-downtime deploy tested — migrations backward-compatible
- [ ] Rollback path documented and rehearsed
- [ ] Observability: structured logs, metrics, traces, health checks
- [ ] SLOs defined; alerts wired to the SLOs, not to raw CPU
- [ ] Dependency audit clean (`composer audit`)
- [ ] Load test executed at 2× expected peak; bottlenecks identified
- [ ] Statamic/WordPress-specific: content workflow, plugin hygiene, update path
- [ ] Runbook exists for the top three failure modes