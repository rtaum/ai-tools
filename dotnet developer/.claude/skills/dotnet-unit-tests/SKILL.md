---
name: dotnet-unit-tests
description: "Write or review .NET unit tests for behavior-level functionality. USE FOR: any C#/.NET request to add, fix, improve, or review unit tests; add regression coverage; test business rules; cover multiple scenarios; isolate collaborators; or improve test readability. Trigger even when the user only says tests, specs, or coverage for .NET code. DO NOT USE FOR: integration, end-to-end, UI, load, or snapshot tests unless the user explicitly wants unit-level coverage."
compatibility: "Requires a .NET test project or permission to add one."
---

# .NET Unit Tests

## Goal

Write small behavior tests for .NET functionality. Test what the unit does, not how one class is shaped.

A unit can be one method, service operation, use case, endpoint handler, validator, policy, or workflow boundary. If that boundary is unclear, ask before writing tests.

## When to use

Use this skill when working in a .NET codebase and the user asks to:

- add, fix, improve, or review unit tests;
- add regression coverage for a bug;
- test business rules or application behavior;
- cover several input or scenario variants;
- isolate collaborators with mocks or stubs;
- improve naming, structure, or readability of existing unit tests.

Do not use it for integration, end-to-end, UI, load, or snapshot tests unless the user explicitly wants unit-level coverage.

## First inspect

1. Find existing test projects, packages, naming, folders, and commands.
2. Reuse existing test helpers/builders before creating new ones.
3. Identify unit boundary and externally meaningful behavior.
4. Pick narrowest test command that runs new tests.

Prefer existing repo conventions over this skill when they conflict.

## Test stack

Use:

- xUnit for tests: `[Fact]`, `[Theory]`, `[InlineData]`, `[MemberData]`.
- Shouldly for assertions: `result.ShouldBe(...)`, `action.ShouldThrow<...>()`.
- NSubstitute only when dependency behavior matters: `Substitute.For<T>()`, `Received()`, `DidNotReceive()`.

Do not add new test libraries if repo already has equivalent accepted stack. Ask before changing stack.

## Unit boundary rule

Name test classes around behavior, not class internals.

Good:

```csharp
public sealed class CheckoutPricingTests
{
    [Fact]
    public void CalculateTotalWhenCustomerIsMemberAppliesDiscount()
    {
        // ...
    }
}
```

Avoid class-by-class mirror tests unless class is the unit boundary.

Bad:

```csharp
public sealed class DiscountCalculatorTests
```

if real behavior is checkout pricing and calculator is just implementation detail.

## Parameterize repeated behavior

When multiple tests assert same behavior with different values, use `[Theory]`.

```csharp
[Theory]
[InlineData(0, 0)]
[InlineData(1, 10)]
[InlineData(3, 30)]
public void CalculateTotalWhenQuantityChangesReturnsExpectedPrice(int quantity, decimal expected)
{
    var result = Price(quantity);

    result.ShouldBe(expected);
}
```

Keep separate `[Fact]` tests when behavior differs, setup differs materially, or expected outcome needs a different explanation.

## Naming

Use GivenWhenThen-style names without underscores:

- Given: method name, command, action, or starting condition.
- When: scenario or important collaborator behavior.
- Then: expected externally visible result.

Examples:

- `GetAccountBalanceWhenDatabaseReturnsDataProvidesBalance`
- `AcceptInvitationWhenTokenIsExpiredReturnsExpiredResult`
- `CancelOrderWhenOrderWasShippedRejectsCancellation`

Do not include `Async` in test names even when production method name ends with `Async`.

## Arrange tests

Use Arrange/Act/Assert spacing. Comments usually unnecessary.

```csharp
[Fact]
public async Task AcceptInvitationWhenTokenIsExpiredReturnsExpiredResult()
{
    var clock = Substitute.For<IClock>();
    clock.UtcNow.Returns(new DateTimeOffset(2026, 1, 2, 0, 0, 0, TimeSpan.Zero));
    var sut = new InvitationAcceptanceBuilder()
        .WithClock(clock)
        .Build();

    var result = await sut.AcceptAsync("expired-token", TestContext.Current.CancellationToken);

    result.ShouldBe(InvitationResult.Expired);
}
```

## Builders

Create reusable builder only when scenarios need different settings or mocked dependencies. Keep it in test code. Avoid builder for one trivial constructor.

Builder should:

- mock all dependencies by default;
- expose only knobs relevant to scenarios;
- keep safe defaults that make ordinary tests short;
- allow replacing a dependency when specific behavior matters;
- avoid hiding important assertions or behavior setup.

```csharp
private sealed class InvitationAcceptanceBuilder
{
    private readonly IInvitationRepository _repository = Substitute.For<IInvitationRepository>();
    private IClock _clock = Substitute.For<IClock>();

    public InvitationAcceptanceBuilder()
    {
        _clock.UtcNow.Returns(new DateTimeOffset(2026, 1, 1, 0, 0, 0, TimeSpan.Zero));
    }

    public InvitationAcceptanceBuilder WithInvitation(Invitation invitation)
    {
        _repository.GetByTokenAsync(invitation.Token, Arg.Any<CancellationToken>())
            .Returns(invitation);
        return this;
    }

    public InvitationAcceptanceBuilder WithClock(IClock clock)
    {
        _clock = clock;
        return this;
    }

    public InvitationAcceptanceService Build() => new(_repository, _clock);
}
```

Use compiling code. If example pattern conflicts with C# rules, fix pattern instead of copying blindly.

## Mocks

Mock external collaborators and slow or nondeterministic dependencies: database, HTTP, clock, queue, filesystem, random, auth context.

Do not mock value objects, pure functions, DTOs, or internal implementation details.

Avoid spies and method-call verification by default. Do not use `Verify`, `Received`, or equivalent interaction assertions unless the call itself is essential to correctness and cannot be observed through result or state.

Prefer constructing collaborators so the test fails naturally if required external behavior does not happen. A green test should prove the unit used the collaborator correctly without separately checking that a method was called.

Use interaction verification only for true side-effect behavior with no better observable result, such as publishing a required event or sending a required notification. Even then, verify the domain-significant payload, not every helper call.

Bad: asserting every helper call to lock implementation.

## Async and cancellation

- Use `async Task` tests for async code.
- Pass `CancellationToken` when API accepts it.
- Do not block with `.Result` or `.Wait()`.
- Avoid timing sleeps. Mock clock/time instead.

## Assertions

Use Shouldly messages rarely, only when they clarify domain intent.

Prefer precise assertions:

```csharp
result.Status.ShouldBe(PaymentStatus.Declined);
result.Errors.ShouldContain("card_expired");
```

Avoid broad snapshots or `ShouldNotBeNull` as main proof.

## Test data

Use minimal domain-valid objects. Put helper methods near tests until shared need is real.

Keep test names readable and behavior-focused:

- `GetCustomerWhenCustomerDoesNotExistReturnsNotFound`
- `ConfirmOrderWhenOrderIsValidPublishesOrderSubmitted`
- `CreatePaymentWhenAmountIsOutsideSupportedRangeRejectsPayment`

## Workflow

1. Add focused test that proves behavior or bug.
2. Run focused test and confirm failure when changing production behavior.
3. Implement/fix minimum production code if requested.
4. Run focused test again.
5. Run relevant wider test command before final report.

## Ask before writing tests when

- unit boundary is ambiguous;
- desired behavior conflicts with existing tests or code;
- adding xUnit/Shouldly/NSubstitute would change repo stack;
- test requires integration infrastructure rather than mocked unit collaborators;
- expected behavior is not inferable from code, docs, or user request.
