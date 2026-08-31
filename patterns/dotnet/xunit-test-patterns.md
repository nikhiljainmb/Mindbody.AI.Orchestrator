# xUnit test patterns (.NET)

Shape and style for tests. Sufficiency (coverage targets, what counts as verified) is judged
by `.claude/skills/verification-protocol/` — this file does not restate it.

## Arrange-Act-Assert

Three blocks in order, separated by blank lines. One behavior per test. No assertions in
arrange, no setup after act.

```csharp
[Fact]
public void Confirm_OrderNotConfirmable_ReturnsFailure()
{
    var order = OrderBuilder.WithStatus(OrderStatus.Draft).Build();

    var result = _service.Confirm(order);

    Assert.False(result.IsSuccess);
    Assert.Equal(OrderError.NotConfirmable, result.Error);
}
```

## Naming: Method_Scenario_ExpectedOutcome

The name alone must say what broke when the test fails. No `Test1`, no `Works`, no
restating the method name without a scenario.

```csharp
public void Confirm_AlreadyConfirmed_ReturnsIdempotentSuccess()   // good
public void Confirm_NullOrder_Throws()                            // good
public void TestConfirm()                                         // bad: no scenario, no outcome
```

## Theories for case tables

Same logic across input values → one `[Theory]`, not copy-pasted `[Fact]`s. `[InlineData]`
for compile-time constants; `[MemberData]` when cases need objects.

```csharp
[Theory]
[InlineData(0, false)]
[InlineData(1, true)]
[InlineData(-1, false)]
public void IsValidQuantity_BoundaryValues_ReturnsExpected(int qty, bool expected) =>
    Assert.Equal(expected, Order.IsValidQuantity(qty));

[Theory]
[MemberData(nameof(RejectedOrders))]
public void Confirm_RejectedOrder_ReturnsFailure(Order order) { /* ... */ }

public static TheoryData<Order> RejectedOrders => new()
{
    OrderBuilder.WithStatus(OrderStatus.Cancelled).Build(),
    OrderBuilder.WithStatus(OrderStatus.Expired).Build(),
};
```

## Shared setup: constructor and fixtures

xUnit creates a new test-class instance per test — the constructor is per-test setup, and
`IDisposable.Dispose` is per-test teardown. Reserve `IClassFixture<T>` for state that is
expensive to build and safe to share read-only across the class. Never share mutable state
between tests.

```csharp
public class OrderServiceTests : IClassFixture<TestDatabaseFixture>
{
    private readonly OrderService _service;

    public OrderServiceTests(TestDatabaseFixture db) =>
        _service = new OrderService(db.Repository, new FakeClock());
}
```

## Mocking: mock at the seam, assert behavior

Mock only the interfaces the class under test depends on — its seams — never types internal
to the unit. Assert observable behavior (return values, state changes, calls that ARE the
contract), not implementation details (call order, incidental lookups). A test that breaks
on refactoring with unchanged behavior is over-mocked.

```csharp
var notifier = new Mock<IOrderNotifier>();
var service = new OrderService(FakeRepository.WithOrder(order), notifier.Object);

var result = await service.ConfirmAsync(order.Id, CancellationToken.None);

Assert.True(result.IsSuccess);
notifier.Verify(n => n.OrderConfirmedAsync(order.Id, It.IsAny<CancellationToken>()), Times.Once);
// bad: notifier.Verify(n => n.GetTemplate(...))  — incidental call, not the contract
```

## Negative and edge cases: required per branch

Every branch in new/changed code gets a test on both sides — success, failure, boundary.
Null/empty inputs, state-machine transitions that must be rejected, and error mapping are
tests, not afterthoughts. reviewer-testing audits this per the verification-protocol Tier 2
audit: with coverage tooling absent, untested branches are listed by method name.

```csharp
[Fact]
public void Confirm_NullOrder_ThrowsArgumentNull() =>
    Assert.Throws<ArgumentNullException>(() => _service.Confirm(null!));
```

## Async tests

`async Task`, never `async void`. Await the act. Assert exceptions with
`await Assert.ThrowsAsync<T>(...)`. Pass `CancellationToken.None` explicitly in arranges,
and where the method observes the token, test the cancelled path too.

```csharp
[Fact]
public async Task ConfirmAsync_Cancelled_ThrowsOperationCanceled()
{
    using var cts = new CancellationTokenSource();
    cts.Cancel();

    await Assert.ThrowsAnyAsync<OperationCanceledException>(
        () => _service.ConfirmAsync(OrderId.New(), cts.Token));
}
```
