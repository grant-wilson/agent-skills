---
description: dotnet-conventions conventions for this project. Applies to projects that contain global.json or *.sln or *.csproj.
---

- Target the current LTS (or newer STS by explicit team decision), pinned in
`global.json`. Scaffold and maintain with the `dotnet` CLI — `dotnet new`,
`dotnet add package`, `dotnet format` — and centralize shared build settings
(`LangVersion`, analyzers, `TreatWarningsAsErrors`) in `Directory.Build.props`,
package versions in `Directory.Packages.props`.
- New HTTP endpoints use minimal APIs organized into `MapGroup` route groups (one
mapping class per feature), returning `TypedResults` so responses are
compile-checked and OpenAPI metadata is inferred.

```csharp
public static class OrderEndpoints
{
    public static RouteGroupBuilder MapOrders(this IEndpointRouteBuilder routes)
    {
        var group = routes.MapGroup("/orders").WithTags("Orders");
        group.MapGet("/{id:guid}", GetOrder);
        return group;
    }

    private static async Task<Results<Ok<OrderDto>, NotFound>> GetOrder(
        Guid id, IOrderService orders, CancellationToken ct) =>
        await orders.FindAsync(id, ct) is { } order
            ? TypedResults.Ok(order.ToDto())
            : TypedResults.NotFound();
}
```
- Use the built-in container with constructor injection only — no service
locators, no static singletons. Register the narrowest correct lifetime (default
to `Scoped` for per-request state, `Singleton` only for stateless/thread-safe
services) and depend on interfaces, not implementations.
- Bind configuration into validated options classes; never read `IConfiguration`
by string key outside the composition root.

```csharp
builder.Services.AddOptions<SmtpOptions>()
    .BindConfiguration("Smtp")
    .ValidateDataAnnotations()
    .ValidateOnStart();
```
- Log through `ILogger<T>` with message templates — never interpolated strings —
so events stay queryable. Prefer source-generated `LoggerMessage` for hot paths.
Instrument with OpenTelemetry (traces, metrics, logs) and expose health checks
via `MapHealthChecks`.

```csharp
logger.LogInformation("Order {OrderId} placed by {CustomerId}", order.Id, customer.Id);
```
- Queries are async, use `AsNoTracking()` for read-only paths, and project to DTOs
with `Select` instead of loading entities to map in memory. Schema changes flow
through migrations committed with the code; lazy-loading proxies are disabled —
load related data explicitly with `Include` or a projection.
- Validate requests at the edge and return RFC 7807 problem details
(`AddProblemDetails()`) for all errors — no leaking exceptions or ad-hoc error
shapes. Version public APIs explicitly from day one.
- Unit tests use xUnit with a fixture per system-under-test; integration tests
boot the real pipeline via `WebApplicationFactory<Program>` and hit endpoints
over HTTP, substituting only true externals (databases via Testcontainers,
third-party APIs via fakes).
