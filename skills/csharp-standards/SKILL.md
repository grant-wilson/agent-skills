---
description: csharp-standards conventions for this project. Applies when working with files matching *.cs.
paths: ["*.cs"]
---

- Target the latest stable C# language version and prefer its idioms: file-scoped
namespaces, primary constructors, collection expressions (`[..]`), raw string
literals for embedded text, and target-typed `new`.

```csharp
namespace Billing;

public sealed class InvoiceService(IInvoiceRepository repository, TimeProvider clock)
{
    private readonly List<string> _warnings = [];
}
```
- `<Nullable>enable</Nullable>` is on solution-wide and warnings are errors. Model
"may be absent" with `?` and handle it at the boundary; the null-forgiving
operator `!` is forbidden outside tests.
- Model data with `record` (or `readonly record struct` for small values) using
`init` setters; mutate by non-destructive `with` expressions. Reserve classes
for entities with identity and behavior.

```csharp
public sealed record Money(decimal Amount, string Currency);

var discounted = price with { Amount = price.Amount * 0.9m };
```
- I/O is `async` end to end: no `.Result`, `.Wait()`, or `Task.Run` to fake
synchrony. Every async public API accepts and forwards a `CancellationToken`.
Suffix async methods with `Async` and use `await foreach` /
`IAsyncEnumerable<T>` for streams.
- Prefer switch expressions and property/list patterns over `if`/`else` chains and
type checks; let the compiler's exhaustiveness checking work for you.

```csharp
public static decimal ShippingFor(Order order) => order switch
{
    { Total: > 100m } => 0m,
    { Destination.Country: "US" } => 5m,
    _ => 15m,
};
```
- Throw the most specific exception type, use the throw-helper APIs
(`ArgumentNullException.ThrowIfNull`,
`ArgumentOutOfRangeException.ThrowIfNegative`) for guards, and never swallow
exceptions — a `catch` block either adds context and rethrows (`throw;`, or
wraps with the inner exception) or genuinely handles the failure.
- Use LINQ for declarative transforms, but keep each query readable in one glance;
extract complex pipelines into named methods. Avoid multiple enumeration of
`IEnumerable<T>` — materialize with `ToList()`/`ToArray()` once when reuse is
needed.
- Default every type to the narrowest visibility (`internal`) and mark classes
`sealed` unless designed for inheritance. Expose interfaces, not concrete types,
from public APIs.
