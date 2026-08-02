---
name: unit-testing-standards
description: Unit testing standards for every language — tests ship with every change, TDD red-green-refactor, Arrange-Act-Assert with one behavior per test, testing through the public surface, fakes over mocks, the FIRST properties, coverage as a floor, and tests treated as first-class code. Use when writing or reviewing tests, fixing a bug, deciding what to assert, or choosing between a fake and a mock.
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
