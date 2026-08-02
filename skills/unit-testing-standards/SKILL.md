---
name: unit-testing-standards
description: Unit testing standards for every language — tests ship with every change, TDD red-green-refactor, Arrange-Act-Assert with one behavior per test, testing through the public surface, choosing between a real collaborator, a fake, a stub and a mock, contract tests that keep fakes honest, the FIRST properties, coverage as a floor, shared test utilities instead of ad-hoc per-file helpers, and tests treated as first-class code. Use when writing or reviewing tests, fixing a bug, deciding what to assert, or choosing and wiring a test double.
---

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

**Name the doubles precisely.** A *stub* returns canned answers to queries, a
*fake* is a working in-memory implementation, and a *mock* is a double whose
calls you assert on. "Mock" is not a synonym for "test double" — a mocking
library used only to return a value produces a stub, and no test asserts on a
stub. Asserting on a stub welds the test to the call graph without proving
anything about behavior.

**Use the real collaborator by default.** Construct the real object and pass it
in; value objects, entities, pure functions, and in-process domain services are
never doubled. Substitute a double only when the collaborator crosses a process
boundary, is nondeterministic, has side effects, or is too slow for a
millisecond-scale test — and substitute it at an architectural port, never in the
middle of the domain. Nondeterminism is always a port: clock, random, UUID, and
environment are injected and faked, never reached through a global. The
framework's fake-timer utilities are the one sanctioned interception, for code
driven by `setTimeout`/`setInterval` it does not own.

**Prefer a fake; reach for a mock when a fake would be disproportionate.** A fake
lets the test assert on resulting state, which survives refactoring; a mock
asserts on the call, which does not. Mocks earn their place where the outgoing
call *is* the observable behavior — an email was sent, an event was published —
or where a whole fake is more machinery than one error branch is worth. Choosing
a mock over a fake is a judgment the test states in a one-line comment.

**Never mock what you don't own.** Wrap a third-party SDK or HTTP client behind a
port you define and fake the port. A double of a type you don't control encodes
your guess about its behavior, and the guess is never re-checked when it changes.

**Never mock the system under test.** No partial mocks, no spying on the class
under test, no stubbing one of its own methods. If a method has to be stubbed out
to test its siblings, it is a collaborator that has not been extracted yet.

**Inject dependencies; never intercept modules.** Module-level interception
(`jest.mock("./mailer")`, patching an import path) fakes the module graph instead
of a design seam: it hides untestable wiring and breaks on file moves. A
dependency that can't be passed in is the defect to fix.

**Verify the minimum.** A mock assertion states that one command happened with
the right payload. No call-order assertions, no "no more interactions" checks, no
argument matchers loose enough to pass on wrong data. Everything else the
collaborator received is implementation detail.

**Stateful fakes ship with a contract test.** A fake that models state or
invariants — a repository, a cache, a queue — comes with one shared suite run
against both the fake and the real adapter, so drift surfaces as a red test
instead of a production incident. Write-only gateways that merely record calls
are exempt.

```typescript
// order-repository.contract.ts — one suite, both implementations
export function anOrderRepositoryContract(make: () => OrderRepository): void {
  it("returns null for an unknown id", async () => {
    expect(await make().findById("missing")).toBeNull();
  });

  it("rejects a duplicate order id", async () => {
    const repository = make();
    await repository.save(anOrder({ id: "a-1" }));
    await expect(repository.save(anOrder({ id: "a-1" }))).rejects.toThrow();
  });
}

anOrderRepositoryContract(() => new FakeOrderRepository()); // unit CI
anOrderRepositoryContract(() => new PostgresOrderRepository(db)); // integration CI
```

**FIRST properties.** Tests are Fast (milliseconds, no real network, disk, or
clock), Isolated (any order, in parallel, no shared mutable state), Repeatable
(no flakiness tolerated — a flaky test is fixed or deleted the day it flakes),
Self-validating, and Timely (written with, not after, the change).

**Coverage is a floor, not a goal.** Maintain full statement and branch coverage
on new and changed code; any excluded line carries a comment stating why. High
coverage never substitutes for asserting the right things — every test must be
able to fail for a real defect.

**Shared test utilities, never ad-hoc helpers.** Setup functions, fakes, data
builders, and custom assertions live in a dedicated testing module that every
spec imports — never redefined at the top of each spec file. A spec file contains
its Arrange–Act–Assert cases and nothing else; the moment a helper is worth
writing, it is worth putting where the next spec can find it. A setup block
copy-pasted between two spec files is the same review failure as duplicated
production code.

```typescript
// testing/order-service.fixture.ts
export function anOrderService(overrides: Partial<Deps> = {}): OrderService {
  return new OrderService(overrides.repository ?? new FakeOrderRepository());
}

// order-service.spec.ts — reads as expectations, not plumbing
it("rejects an order whose total is negative", () => {
  const service = anOrderService();
  expect(service.place(anOrder({ totalCents: -100 }))).toEqual({
    ok: false,
    error: "invalid-total",
  });
});
```

**Tests are first-class code.** Co-locate specs with the module under test, hold
them to the same review standards, and keep the shared utilities they import
under the same lint, type, and coverage rules as production code. Delete a test
only when the behavior it specifies is deleted.
