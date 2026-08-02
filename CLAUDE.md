## architecture-standards

Architecture standards for all services and applications in this project.

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

## general-coding-standards

General coding standards that apply to every language and framework in this
project.

**Self-documenting code.** Names of files, types, functions, and variables say
what a thing is or does — precisely enough that most comments become
unnecessary. Comments exist only to explain _why_: a non-obvious constraint,
trade-off, or workaround. A comment that restates the code it precedes is
deleted in review.

**Small units, single responsibility.** A function does one thing at one level
of abstraction; a class/module has one reason to change. Prefer extracting a
well-named function over adding a section comment. Deep nesting is flattened
with guard clauses and early returns.

**Fail fast and loudly.** Validate inputs at boundaries and throw immediately
with a message naming what was wrong and what was expected. Never silently
swallow an error, return a magic default on failure, or log-and-continue past a
broken invariant.

**DRY, applied with judgment.** Duplicate _knowledge_ is never acceptable — a
business rule lives in exactly one place. Incidental similarity between two
pieces of code is not duplication; wait for the rule of three before
abstracting, and prefer extending an existing tested abstraction over writing a
parallel one.

**SOLID as the default shape.** Depend on abstractions where a surface has more
than one consumer; extend existing behavior rather than modifying tested logic;
keep interfaces small and role-specific rather than one wide interface per
class.

**No dead or speculative code.** Delete unused code, commented-out blocks, and
"we might need it later" flexibility — version control remembers. Every line in
the repo is live, tested, and reachable.

**Tooling is the authority.** Formatting and lint rules are enforced by the
project's configured tools and run in CI; style is never debated in review.
Warnings are errors — a change that introduces a new warning does not merge.

**Every change is reviewed and tested.** No direct commits to the main branch. A
pull request is small enough to review in one sitting, describes _why_ as well
as _what_, includes the tests that prove the change (see the unit-testing
standards), and merges only green.

**Security hygiene by default.** No secrets in source or logs, all input treated
as untrusted at trust boundaries, dependencies pinned and updated deliberately,
and least privilege for every credential, token, and permission the code
requests.

## unit-testing-standards

Unit testing standards — these apply to every code change, in every language.

**Always implement unit tests.** No production behavior ships without unit tests
proving it. A bug fix starts with a failing test that reproduces the bug. Code
that is "hard to test" is a design problem to fix (extract a port, inject the
dependency), never a reason to skip the test.

**Follow TDD whenever possible.** Red–Green–Refactor: write a failing test that
specifies the behavior, write the minimum code to pass, then refactor with the
tests as a safety net. When strict test-first is impractical (e.g. exploratory
spikes), the spike is thrown away and rebuilt test-first — tests written after
the fact must still be seen to fail when the behavior is reverted.

**Arrange–Act–Assert, one behavior per test.** Every test has three visible
sections and verifies exactly one behavior; multiple assertions are fine only
when they describe one outcome. Test names state the scenario and expectation
(`rejects an order whose total is negative`), not the method name.

```typescript
it("rejects an order whose total is negative", () => {
  // Arrange
  const service = new OrderService(new FakeOrderRepository());

  // Act
  const result = service.place({ items: [], totalCents: -100 });

  // Assert
  expect(result).toEqual({ ok: false, error: "invalid-total" });
});
```

**Test behavior through the public surface.** Assert on observable outcomes —
return values, emitted events, state visible through the API — never on private
internals or call sequences that are mere implementation detail. A refactor that
preserves behavior must not break tests.

**Prefer fakes over mocks.** Substitute real collaborators at the architectural
ports with simple in-memory fakes; reserve mocks/spies for verifying genuine
outgoing commands (an email was sent, an event was published). Never mock types
you don't own — wrap them behind a port and fake the port.

**FIRST properties.** Tests are Fast (milliseconds, no real network/disk/clock —
inject a fake clock and use the framework's time-mocking utilities), Isolated
(any order, in parallel, no shared mutable state), Repeatable (no flakiness
tolerated — a flaky test is fixed or deleted the day it flakes),
Self-validating, and Timely (written with, not after, the change).

**Coverage is a floor, not a goal.** Maintain full statement and branch coverage
on new and changed code; any excluded line carries a comment stating why. High
coverage never substitutes for asserting the right things — every test must be
able to fail for a real defect.

**Tests are first-class code.** Co-locate them with the module under test, hold
them to the same review standards, and refactor shared setup into
builders/fixtures instead of copy-paste. Delete a test only when the behavior it
specifies is deleted.