---
description: 'Functional style in Java 21: pure functions, immutability, records, sealed types, Optional and Result, streams'
applyTo: '**/*.java'
---

# Java Functional Style

Pattern-matching `switch` needs Java 21; `record` and `sealed` need 17 or later.

## Pure functions
- Write business rules as `static` methods in `final` classes with a private constructor.
- Depend only on parameters; receive `Instant`, `UUID` and random values as arguments.
- Turn instance methods that ignore object state into static functions.

## Immutability
- Use `record` for value objects, DTOs and domain models; validate invariants in the compact constructor.
- In regular classes, use `final` fields and no setters.
- Return a new instance (`withStatus`) instead of mutating.
- Use `List.of`, `Map.of`, `stream.toList()` and `List.copyOf`.

## Sealed types
- Model states and variants with `sealed interface` + `record`, so invalid states cannot be represented.
- Use pattern-matching `switch` without `default` on sealed types, so the compiler checks exhaustiveness.

## Absence and errors
- Use `Optional<T>` as a return type only.
- Return expected business errors as the project's `Result`/`Either`. If none exists, create a small `sealed interface Result<T, E>` with `Ok` and `Err` records; add Vavr only if it is already a dependency.
- Reserve exceptions for unexpected and infrastructure failures.
- Return empty `Optional` or an empty collection for absence.

## Higher-order functions and streams
- Pass behavior as `Function`, `Predicate`, `Supplier` or `Consumer`; compose with `andThen`, `and`, `or`.
- Move lambdas longer than a few lines into named methods and use method references.
- Use `map`, `filter`, `collect`, `reduce` and `groupingBy` instead of loops with mutable accumulators.
- Keep stream stages free of side effects; use `peek` only to debug.
- Measure before using `parallelStream`.
- Replace deep recursion with a stream or a loop inside a pure function.

## Example

```java
public static Result<Order, OrderError> pay(Order order, String txId, Instant now) {
    return switch (order.status()) {
        case OrderStatus.Created c  -> new Result.Ok<>(order.withStatus(new OrderStatus.Paid(now, txId)));
        case OrderStatus.Paid p     -> new Result.Err<>(new OrderError.AlreadyPaid());
        case OrderStatus.Canceled x -> new Result.Err<>(new OrderError.Canceled());
    };
}

Map<String, Money> totalByCustomer = orders.stream()
    .filter(paid)
    .collect(Collectors.groupingBy(
        Order::customerId,
        Collectors.reducing(Money.ZERO, Order::total, Money::add)));
```

## Verify
- Test pure functions as input → output, without mocks, covering every branch of each `switch` over a sealed type.
