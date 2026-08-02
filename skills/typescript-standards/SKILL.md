---
name: typescript-standards
description: TypeScript standards — strict compiler options that are never weakened, ECMAScript private fields, no `any` with `unknown` narrowed at boundaries, discriminated unions with exhaustiveness checks, modern language features, immutability by default, type-only imports, first-class AbortSignal cancellation, and Error subclasses with `cause`. Use when writing or reviewing TypeScript (*.ts) code.
---

- Compile with `strict: true` plus `noUncheckedIndexedAccess`,
`exactOptionalPropertyTypes`, and `verbatimModuleSyntax`. Never weaken these
per-file; fix the type error instead of suppressing it. `@ts-expect-error`
requires a comment explaining why, and `@ts-ignore` is forbidden.
- Use ECMAScript private fields (`#name`) for all private state and private
methods — never the TypeScript `private` keyword, which is erased at runtime and
offers no real encapsulation.

```typescript
class Account {
  #balance = 0;

  deposit(amount: number): void {
    this.#balance += amount;
  }

  get balance(): number {
    return this.#balance;
  }
}
```
- `any` is forbidden. Use `unknown` at trust boundaries (HTTP responses, JSON
parsing, catch clauses) and narrow it with type guards or a schema validator
before use.

```typescript
function isUser(value: unknown): value is User {
  return typeof value === "object" && value !== null && "id" in value;
}
```
- Model state and domain variants as discriminated unions, and switch on the
discriminant with an exhaustiveness check so adding a variant fails compilation
everywhere it isn't handled.

```typescript
type LoadState<T> =
  | { status: "idle" }
  | { status: "loading" }
  | { status: "loaded"; data: T }
  | { status: "failed"; error: Error };

function render(state: LoadState<string>): string {
  switch (state.status) {
    case "idle":
      return "…";
    case "loading":
      return "Loading";
    case "loaded":
      return state.data;
    case "failed":
      return state.error.message;
    default:
      return state satisfies never;
  }
}
```
- Prefer the latest stable language features over legacy patterns: `satisfies` for
validated-but-not-widened literals, `using`/`await using` for deterministic
resource cleanup, `structuredClone` over hand-rolled deep copies, `??`/`?.` over
`||` chains, and `Object.groupBy`/`Array.prototype.at` over manual equivalents.
- Declare with `const`, mark properties and arrays `readonly`, and return new
values instead of mutating inputs. Mutation is allowed only inside a function's
own local scope for performance, never on parameters or shared state.
- Import types with `import type { … }` so type-only dependencies are erased from
the emitted graph and circular value imports can't hide behind types.
- Cancellation is first-class for async work that can outlive the caller's
interest. A long-running or externally-waiting async method (I/O, GPU work,
worker round-trips) accepts a trailing `options?: { abortSignal?: AbortSignal }`,
honours it — `options?.abortSignal?.throwIfAborted()` at each await boundary — and
forwards it to inner async calls. Recognize cancellation by `error.name ===
"AbortError"` and treat it as a normal outcome, not a failure. A disposable that
owns in-flight async work holds an internal `AbortController` that
`[Symbol.dispose]()` aborts, and links a caller's signal to it with
`AbortSignal.any([caller, own.signal])`. Do not add a signal to fast,
purely-synchronous, or fire-and-forget async methods — a signal must mean
something.
- Throw only `Error` subclasses, never strings or plain objects. Define one custom
error class per failure category, carry context in fields, and chain the
original failure via the `cause` option.

```typescript
class OrderNotFoundError extends Error {
  constructor(readonly orderId: string, options?: ErrorOptions) {
    super(`order ${orderId} not found`, options);
    this.name = "OrderNotFoundError";
  }
}
```
