---
description: 'Functional programming techniques in Java 17+: pure functions, immutability, sealed types, Optional/Result, higher-order functions and streams'
applyTo: '**/*.java'
---

# Functional Programming in Java

Language-level techniques for writing functional-style Java. For architecture and OO integration see `java-functional-oo-integration.instructions.md`.

Project conventions take precedence over these generic guidelines. When they conflict, follow the project and say so.

## Project Context

- Language: Java 21 (minimum 17 for `record` and `sealed`).
- Java version changes: use the `java-upgrade` skill, one LTS version at a time.

## Pure Functions

- Implement business rules as `static` methods in `final` classes with a private constructor.
- Depend only on parameters. Do not read or change external state.
- Receive `Instant`, `UUID` and random values as parameters instead of calling `Instant.now()`, `UUID.randomUUID()` or `Random` inside the function.
- Extract any instance method that does not use object state into a static function.

### Good

```java
public static Result<Order, OrderError> pay(Order order, String txId, Instant now) {
    return switch (order.status()) {
        case OrderStatus.Created c  -> new Result.Ok<>(order.withStatus(new OrderStatus.Paid(now, txId)));
        case OrderStatus.Paid p     -> new Result.Err<>(new OrderError.AlreadyPaid());
        case OrderStatus.Canceled x -> new Result.Err<>(new OrderError.Canceled());
    };
}
```

### Bad

```java
public Order pay(Order order, String txId) {
    order.setStatus("PAID");            // mutates the argument
    order.setPaidAt(Instant.now());     // hidden clock dependency
    repository.save(order);             // I/O mixed with rules
    return order;
}
```

## Immutability

- Use `record` for value objects, DTOs and domain entities.
- In regular classes, declare fields `final` and do not create setters.
- Return a new instance instead of mutating (`withStatus`, `withAmount`).
- Use `List.of`, `Map.of`, `Set.of`, `stream.toList()` and `List.copyOf`.
- Validate invariants in the compact constructor of the `record`.

## Sealed Types and Pattern Matching

- Model states and variants with `sealed interface` + `record`.
- Use pattern-matching `switch`. Do not add `default` to a switch over a `sealed` type, so the compiler keeps checking exhaustiveness.
- Make invalid states unrepresentable.
- Replace Visitor with `sealed` + `switch`.

## Absence and Errors

- Use `Optional<T>` for absence, only as a return type. Do not use it in fields, parameters or collections.
- Use the project's existing `Result`/`Either` type for expected business errors. If none exists, create a small `sealed interface Result<T, E>` with `Ok` and `Err` records. Add Vavr only if it is already a dependency.
- Reserve exceptions for unexpected and infrastructure failures. Do not use them for control flow.
- Do not return `null` or use it as an error signal.

## Higher-Order Functions and Composition

- Pass behavior as `Function`, `BiFunction`, `Predicate`, `Supplier`, `Consumer` or `UnaryOperator`.
- Compose with `andThen`, `compose`, `and`, `or` and `negate`.
- Lambdas capture only *effectively final* variables.
- Extract lambdas longer than a few lines into named methods and use method references.
- For currying, use nested lambdas (`a -> b -> a + b`).

## Streams

- Use `map`, `filter`, `reduce` and `collect` instead of loops with mutable accumulators.
- Do not produce side effects inside `map` or `filter`. Do not modify external collections.
- Use `peek` only for debugging.
- Prefer `Collectors.groupingBy` over maps built in loops.
- Do not use `parallelStream` without measuring first.
- Avoid deep recursion (Java has no tail-call optimization). Use streams or a loop encapsulated in a pure function.

### Good

```java
Map<String, Money> totalByCustomer = orders.stream()
    .filter(paid)
    .collect(Collectors.groupingBy(
        Order::customerId,
        Collectors.reducing(Money.ZERO, Order::total, Money::add)));
```

### Bad

```java
Map<String, Money> totals = new HashMap<>();
orders.forEach(o -> {                   // side effect inside lambda
    if (paid.test(o)) totals.merge(o.customerId(), o.total(), Money::add);
});
```

## Avoid

- `Optional` in fields or parameters.
- Side effects inside streams.
- Mutating method arguments.
- Hidden dependencies on time, randomness or global state.

## Validation

- Build and tests: `mvn verify` or `./gradlew build`.
- Test pure functions without mocks: input → output.
- Follow `java-junit5-assertions.instructions.md` for assertions.
- Cover every branch of each `switch` over `sealed` types.
