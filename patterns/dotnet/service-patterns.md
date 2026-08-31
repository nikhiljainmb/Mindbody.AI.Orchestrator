# Service patterns (.NET)

Structural conventions for ASP.NET Core services. reviewer-architecture checks conformance;
coder consults before implementing.

## Constructor DI

Dependencies arrive via constructor as interfaces; no service locator, no `new` for anything
with behavior. Guard with `ArgumentNullException.ThrowIfNull`. Register in one composition
root, grouped in extension methods per layer so `Program.cs` stays a table of contents.

```csharp
public class OrderService(IOrderProvider provider, IClock clock) : IOrderService { /* ... */ }

// Program.cs
builder.Services.AddDomainServices().AddInfrastructure(builder.Configuration);

// Infrastructure registration extension
public static IServiceCollection AddInfrastructure(this IServiceCollection s, IConfiguration c) =>
    s.AddSingleton<IOrderProvider, SqlOrderProvider>();  // stateless clients: singleton, never per-request
```

## Options pattern, validated at startup

Configuration binds to a typed options class via `IOptions<T>`. Validate with data
annotations plus `ValidateOnStart` — a bad config fails deployment, not the first request.
Never inject `IConfiguration` into domain code.

```csharp
public sealed class RetryOptions
{
    [Range(0, 10)] public int MaxAttempts { get; init; } = 3;
    [Required] public required Uri BaseAddress { get; init; }
}

builder.Services.AddOptions<RetryOptions>()
    .BindConfiguration("Retry")
    .ValidateDataAnnotations()
    .ValidateOnStart();
```

## Result types over exceptions for expected failures

An outcome the caller must handle (validation failure, not-found, rejected state transition)
is a return value, not an exception. Exceptions are for bugs and infrastructure faults.
Callers switch on the result; nothing is thrown across layers for control flow.

```csharp
public sealed record Result<T>
{
    public bool IsSuccess { get; init; }
    public T? Value { get; init; }
    public string? Error { get; init; }

    public static Result<T> Success(T value) => new() { IsSuccess = true, Value = value };
    public static Result<T> Failure(string error) => new() { Error = error };
}

public Result<Order> Confirm(Order order) =>
    order.Status == OrderStatus.PendingConfirmation
        ? Result<Order>.Success(order.WithStatus(OrderStatus.Confirmed))
        : Result<Order>.Failure("Order is not in a confirmable state");
```

## Layering and dependency direction

`API -> Application -> Domain`. References point inward only; Domain references nothing
above it and no infrastructure package. Infrastructure implements interfaces owned by the
inner layers — Domain defines `IOrderProvider`, an Infrastructure project implements it, and
the composition root wires them. No vendor SDK or persistence type ever appears in a Domain
signature.

```csharp
// Domain — owns the interface, speaks only domain types
public interface IOrderProvider
{
    Task<Order?> GetAsync(OrderId id, CancellationToken ct);
}

// Infrastructure — implements it; SDK/database types stay internal to this project
internal sealed class SqlOrderProvider(IDbConnectionFactory db) : IOrderProvider { /* ... */ }
```

## Thin controllers

Controllers own HTTP concerns only: routing, auth attributes, request-shape validation,
mapping transport models to domain requests, and mapping results to status codes. No
business rules, no branching on domain state — one service call per action.

```csharp
[HttpPost("{id}/confirm")]
public async Task<IActionResult> Confirm(Guid id, CancellationToken ct)
{
    var result = await _orderService.ConfirmAsync(new OrderId(id), ct);
    return result.IsSuccess ? Ok(OrderResponse.From(result.Value!)) : Conflict(result.Error);
}
```

## Cancellation tokens flow end to end

Every async method takes a `CancellationToken` and passes it to every awaited call — from
the controller action (ASP.NET Core binds it to request abort) down through services,
providers, and clients. Never swallow it, never substitute `CancellationToken.None` mid-chain,
and never leave a default value on an inner-layer signature where forgetting to pass it
would compile silently.

```csharp
public async Task<Result<Order>> ConfirmAsync(OrderId id, CancellationToken ct)
{
    var order = await _provider.GetAsync(id, ct);          // token flows down
    if (order is null) return Result<Order>.Failure("Order not found");
    return Confirm(order);
}
```
