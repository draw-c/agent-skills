---
name: elixir-architect
description: Elixir architect for system design, OTP patterns, supervision trees, scalability, fault-tolerance, and high-level architecture decisions in Phoenix and pure Elixir apps.
---

# Elixir Architect Skill

## Core Principles
- Everything is a process: use GenServer, Supervisor, Agent, Task appropriately.
- Supervision trees first — design for failure and automatic recovery.
- Contexts for domain boundaries (DDD-inspired).
- Keep state simple (structs/maps); avoid deep nesting.
- Leverage BEAM strengths: concurrency, distribution, hot code upgrades.

## OTP Patterns
- GenServer: for stateful processes with `handle_call`, `handle_cast`, `handle_info`.
- Supervisor: one_for_one, one_for_all, rest_for_one strategies.
- Always start GenServers under supervision (`start_link`).
- Use `DynamicSupervisor` for dynamic children.

## Scalability & Design
- Phoenix contexts for bounded contexts.
- Ecto.Multi for complex transactions.
- Distributed Erlang / libcluster for multi-node setups.
- Telemetry for observability from day one.

## Review Checklist
- [ ] Proper supervision hierarchy documented
- [ ] No tight coupling between domains
- [ ] Fault-tolerance strategies reviewed
- [ ] Scalability under load considered
- [ ] OTP behaviours used correctly