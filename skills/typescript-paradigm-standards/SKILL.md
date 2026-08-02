---
name: typescript-paradigm-standards
description: Choosing between object-oriented and functional design in TypeScript — neither is the default, each is selected for the problem it fits; what object orientation is for (identity, encapsulated state, lifecycle, substitutable hierarchies), what functional design is for (transformations, derived state, stable data with growing operations), correct use of inheritance under Liskov substitution and designed-for-extension bases, discriminated unions versus interfaces versus class hierarchies for polymorphism, and serializable data at boundaries. Use when deciding whether something should be a class, a function, or a union, when designing a type hierarchy, when modelling a domain, or when reviewing the shape of a new module.
---

Neither paradigm is the default. TypeScript is genuinely multi-paradigm, and the
right answer is chosen per unit of code from the shape of the problem: object
orientation where there is identity, state, and substitutable behaviour;
functional design where there is transformation of data. A codebase that reaches
for one everywhere pays for it — all-classes code accumulates stateless
ceremony, all-functions code smuggles state into module scope and passes the
same five arguments everywhere.

Decide deliberately, and be able to say in one sentence why this unit is a class,
a function, or a union. "Consistency with the file next to it" is a reason;
"we always do it this way" is not.

## Object-oriented design fits when

- **State must be encapsulated behind invariants.** The unit owns mutable state
that callers must not corrupt, and every public method leaves it valid — a
connection pool, rate limiter, cache, circuit breaker, running state machine, or
domain aggregate enforcing rules across its parts.
- **The thing has identity that outlives its values.** An entity is the same
thing after its fields change; a value is not. Entities want classes with an
identifier and behaviour; values want immutable data.
- **Behaviour and the data it guards belong together.** When callers should never
see the representation — only what can be done with it — a class is the tool
that makes that unrepresentable rather than merely discouraged.
- **There are several substitutable implementations of one contract.** Ports and
adapters, strategies, providers: the caller depends on the abstraction and the
implementation is chosen at the composition root.
- **A resource has a lifecycle.** Anything opened and closed, or holding an
`AbortController`, subscription, timer, or handle, is a class implementing
`[Symbol.dispose]`/`[Symbol.asyncDispose]`.
- **The framework requires it.** Angular components and services, ORM entities,
custom elements, `Error` subclasses. Follow the framework.

## Functional design fits when

- **The unit is a transformation.** Input in, value out, no state retained
between calls — pricing rules, validation, parsing, formatting, sorting, mapping
between DTO and domain.
- **The state is already owned elsewhere.** When a store, signal, database row,
or request context owns the state, the logic over it is `(state, input) =>
newState`, not a second object shadowing the same state.
- **Operations grow faster than shapes.** A fixed set of data variants with a
steadily growing set of things done to them is served better by functions over a
union than by methods on a hierarchy.
- **The result must be derived, cached, or recomputed freely.** Pure functions are
memoizable, parallelizable, and reorderable; methods that read `this` are not.
- **It must cross a boundary.** See *Data at the boundaries*.

## Inheritance

Inheritance is a legitimate and sometimes the best tool. It models an *is-a*
relationship in which the subtype is usable anywhere the base type is. Use it
when that is true, and use composition when it is not — the failure mode is
inheritance chosen for code reuse, not inheritance itself.

- **Subtypes must satisfy Liskov substitution.** A subclass honours the base
contract: no strengthened preconditions, no weakened postconditions, no method
overridden to throw or no-op because it "doesn't apply here". If a subtype must
refuse part of the base surface, the hierarchy is wrong — split the base.
- **A base class is either designed for inheritance or not extended.** Designing
for it means deliberate extension points, documented invariants the subclass must
maintain, and a small, stable `protected` surface. TypeScript has no `final`, so
state the intent in the class doc and don't extend classes that never made the
promise.
- **`protected` members are public API to your subclasses.** Changing one breaks
every subclass, in and out of the repo. Note that ECMAScript `#private` fields
are invisible to subclasses by design — keep `#` for what subclasses must not
touch, and use `protected` only for the extension surface you intend to support.
- **Constructors must not call overridable methods.** The subclass's fields are
not initialized yet, so the override observes `undefined`. Do the work in a
factory or an `init` step instead.
- **Inherit behaviour and contract, never just convenience.** Needing a helper
method is a *has-a*: import the function or hold the collaborator. A hierarchy
that exists so unrelated types can share fields is a fragile base class waiting
to happen.
- **Each level must earn itself** by specializing state or behaviour. Depth is not
capped by rule, but a level that only forwards to its parent should be removed,
and cross-cutting concerns (logging, caching, retries) belong in decorators that
implement the same interface — not in another tier of the tree.
- **`abstract class` when the family shares state or implementation; `interface`
when it shares only a contract.** Prefer the interface when there is nothing real
to inherit, so implementations stay free to choose their own base.

```typescript
// Template method — legitimate: closed family, base owns the invariant.
abstract class Migration {
  async run(db: Database): Promise<void> {
    await db.transaction(async (tx) => {
      await this.apply(tx);          // the only varying step
      await tx.recordApplied(this.version);
    });
  }
  protected abstract apply(tx: Transaction): Promise<void>;
  abstract readonly version: string;
}

// Same shape, no shared state or invariant — a parameter is simpler.
async function importRows(raw: string, parse: (raw: string) => Row[]): Promise<void> {
  await save(parse(raw));
}
```

- **Mixins** (`class X extends Serializable(Base)`) compose orthogonal
capabilities onto a class. They are worth the type gymnastics only for genuinely
independent capabilities applied to several unrelated classes; for one consumer,
write the method.

## Polymorphism: union, interface, or hierarchy

Pick by which axis actually changes. Getting it backwards produces either a
`switch` edited for every new provider, or an interface edited for every new
query.

- **Variants fixed, operations grow → discriminated union.** Adding an operation
is one new function, and an exhaustive `switch` makes the compiler find every
case (see `typescript-standards`).
- **Operations fixed, implementations grow → interface, one class each.** Adding
an implementation touches no existing code. A contract with exactly one method is
a function type — `type Clock = () => Date` beats a `Clock` interface with
`now()`.
- **Operations fixed, implementations grow, and they share real state or an
enforced algorithm → abstract base class.** This is the case inheritance exists
for; the base must still meet the rules above.

## Forbidden shapes

These are wrong in both paradigms.

- **A class with only static members.** That is a module with extra syntax.
Export the functions.
- **A class with no state and no contract** — nothing injected, nothing stored,
implementing no interface. A class holding injected collaborators or standing in
for an interface is legitimate; one that exists so callers can write `new` is
not.
- **A `Manager`, `Helper`, or `Util` class that is a bag of unrelated methods.**
Split it by capability into modules, or by state into real objects.
- **`this` used as an implicit parameter** — data stashed in fields so a later
method can read it, rather than passed as an argument.
- **A getter that does work, or a property access with side effects.** Property
reads are cheap and pure; anything else is a method.

## Data at the boundaries

- **Anything crossing a process, thread, storage, or framework boundary is plain
serializable data** — HTTP payloads, events, store and signal state, worker
messages, cache entries. Class instances lose their prototype through `JSON`,
`structuredClone`, and `postMessage`, so their methods silently disappear.
- **Convert at the edge.** Validate `unknown` into a domain type in one place, and
rehydrate into a class where the class earns it by the rules above.

## Side effects

- **Keep decisions separable from effects**, whichever paradigm holds them. I/O,
clocks, randomness, and logging belong in a thin shell around logic that takes
values and returns values — that is what lets tests use fakes instead of mocks
(see `unit-testing-standards`).
- **Take dependencies as parameters or constructor arguments, never as imported
singletons.** An imported mutable module-level instance is a hidden global in
either style.
- **A function that mutates its argument says so in its name and returns `void`.**
Otherwise return a new value and leave inputs `readonly`.

## Write TypeScript, not a translated language

- **Don't port Haskell.** No point-free pipelines of curried helpers, no
monad/typeclass libraries. Exceptions are the default failure channel — throw
`Error` subclasses. A `Result`/`Either` type is justified where failure is an
expected, enumerable outcome the immediate caller must handle, at that boundary,
not threaded through the whole call graph.
- **Don't port Java.** No factory-of-factory indirection, no interface per class
by reflex, no getter/setter pairs wrapping public data, no singleton pattern
where a module or the DI container already does it.

## Review checkpoints

- Every class answers "what state, identity, or contract does this own?" in one
sentence; every free function is one a caller can use without setup.
- Every `extends` answers "is a subclass usable everywhere the base is?" — and the
base was designed for extension.
- New polymorphism is checked against the change axis: union for stable variants,
interface or base class for stable operations.
- No static-only class, no class instance in serialized or reactive state, and no
hierarchy standing in for a shared helper.
