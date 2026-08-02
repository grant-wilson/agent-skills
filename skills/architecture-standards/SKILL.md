---
name: architecture-standards
description: Architecture standards for services and applications — the inward dependency rule, ports and adapters, a single composition root, feature-first organization, bounded contexts, twelve-factor operability, observability, and architecture decision records. Use when designing a new service or feature, choosing where code belongs, adding a dependency or storage technology, wiring dependency injection, or reviewing structural changes.
---

**Dependency rule.** Source code dependencies point inward: domain logic depends
on nothing, application logic depends on the domain, and infrastructure (HTTP,
database, message bus, UI) depends on both. The domain never imports a framework
type. Enforce the direction with project/module boundaries, not convention
alone.

**Ports and adapters.** The application core defines interfaces (ports) for
everything it needs from the outside world — persistence, clocks, external APIs
— and infrastructure supplies the implementations (adapters). Swapping a
database or HTTP client must not touch domain code, and tests exercise the core
through the same ports with in-memory fakes.

**Composition root.** All wiring happens in one place at the process entry point
(DI container registration, config binding, adapter selection). No `new`-ing of
infrastructure inside business logic, no service locators, no static singletons.

**Feature-first organization.** Group code by business capability (`orders/`,
`billing/`), not by technical layer (`controllers/`, `helpers/`). A feature
folder contains its endpoints/components, application logic, and tests together;
shared code is extracted only after a third consumer appears (rule of three).

**Explicit boundaries between contexts.** When two areas of the system disagree
about what a term means, they are separate bounded contexts: separate models,
translated at the boundary via DTOs or events. Never share a database table or
domain model across contexts to save typing.

**Twelve-factor operability.** Configuration comes from the environment
(validated at startup, fail fast on missing values), services are stateless
between requests, logs go to stdout as structured events, and every external
call has a timeout, retry policy with backoff, and a defined failure mode.

**Design for observability.** Every service exposes health checks, emits
structured logs with correlation IDs propagated across service boundaries, and
records traces and key business metrics from day one — observability is a
feature requirement, not an afterthought.

**Decisions are recorded.** Any architecturally significant choice (new
dependency, storage technology, cross-context contract, deviation from these
standards) is captured as a short Architecture Decision Record in the repo —
context, decision, consequences — so the "why" survives the people who made it.
