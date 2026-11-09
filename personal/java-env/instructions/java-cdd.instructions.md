---
description: 'Cognitive-Driven Development (CDD) for Java: measure intrinsic complexity points (ICP), enforce team-defined limits per code unit and refactor to keep code understandable'
applyTo: '**/*.java'
---

# Cognitive-Driven Development (CDD) for Java

Keep each code unit small enough to be understood without overloading working memory. Measure complexity with Intrinsic Complexity Points (ICP) and limit it per unit. CDD complements `java-design-principles.instructions.md` and the functional guidelines. It does not replace them.

## Relation to Other Guidelines

- This file governs **coupling and size per unit** with a flat count and team-defined limits.
- `java-cognitive-complexity.instructions.md` governs **control-flow understandability per method**, with nesting penalties.
- Apply both. Do not add the two scores together.

## Scope

- Unit of measure: **method** (constructors and lambda bodies count toward the enclosing method) and **class** (file).
- Apply to production code. Do not apply ICP limits to test classes.

## ICP Scoring

Add 1 point for each element below, within the unit.

| Element | Points |
|---|---|
| `if`, `else if`, `else`, ternary operator | 1 each |
| `switch` (statement or expression, all cases combined) | 1 per `switch` |
| Loop (`for`, enhanced `for`, `while`, `do`) | 1 each |
| `try/catch` block | 1 each |
| Distinct project class or interface referenced (coupling) | 1 each, counted once per unit |

- Do not count JDK types, primitives, or the class being measured as coupling.
- Do not count lambdas or method references passed as arguments. Streams and functional style are not penalized.
- A class's ICP is the sum of its methods' ICP plus its own coupling.

### Example

```java
public Money discount(Order order, Customer customer) {
    if (order.items().isEmpty()) {                   // +1
        return Money.ZERO;
    }
    if (customer.isPremium()) {                      // +1
        return order.total().multiply(PREMIUM);
    } else if (order.isLarge()) {                    // +1
        return order.total().multiply(LARGE);
    } else {                                         // +1
        return order.total();
    }
}
// Coupling: Money, Order, Customer = +3
// ICP = 7
```

## Limits

The team defines and revises the limits. Use these starting values until the team sets its own:

| Unit | Limit |
|---|---|
| Method | 10 |
| Class | 40 |

- A unit above its limit requires refactoring.
- A unit above 80% of its limit is under watch. Do not add complexity to it without need.

## Rules When Writing or Changing Code

- Before editing, estimate the ICP of the affected method and class.
- Do not push a unit over its limit. Refactor first, or place the new logic in a new unit.
- When a changed unit exceeds its limit, state its ICP and the refactoring applied.
- Measuring is not evaluating. If a unit exceeds the limit and splitting it would break cohesion or hurt readability, keep it and state the reason in one line.
- Do not split a cohesive unit into meaningless fragments only to lower the number.

## Reducing ICP

| Symptom | Action |
|---|---|
| Many conditionals | Use guard clauses (early return), `sealed` + `switch`, polymorphism or a `Map<Key, Function<...>>` |
| Nested conditionals | Extract a named method or `Predicate` |
| Loop with accumulation | Extract into a pure function or a stream pipeline |
| `try/catch` in business logic | Move to the boundary. Use `Result` for expected errors |
| Many project classes referenced | Pass only the needed data (a `record`), depend on a port, split responsibilities |
| Class over limit | Extract a class by responsibility. Move rules into pure functions in a separate class |

- Extracting private methods lowers the method's ICP but not the class's. When the class is over its limit, extract a class.

### Good

```java
public Money discount(Order order, Customer customer) {
    if (order.items().isEmpty()) return Money.ZERO;
    return order.total().multiply(rateFor(order, customer));
}

private Rate rateFor(Order order, Customer customer) {
    if (customer.isPremium()) return PREMIUM;
    return order.isLarge() ? LARGE : Rate.NONE;
}
```

### Bad

```java
public Money discount(Order order, Customer customer) {
    if (!order.items().isEmpty()) {
        if (customer.isPremium()) {
            if (order.isLarge()) {
                // deeply nested rules
            }
        }
    }
    // ...
}
```

## Extension vs Modification

- Modify existing code directly when domain knowledge is limited and automated tests cover the change.
- Extend through a new implementation (interface, decorator, cache proxy) when the point of evolution is clear and a second real use case exists.
- Do not generalize ahead of need. Small duplication is better than a wrong abstraction.
- Do not add extension points for hypothetical futures.

## Avoid

- Adding branches to a unit already near its limit.
- Abstractions created only to reduce ICP that add indirection with no second use.
- Hiding complexity by moving code between methods of the same over-limit class.
- Treating the limit as a goal. The goal is code that a teammate understands quickly.

## Validation

- Build and tests: `mvn verify` or `./gradlew build`.
- Behavior stays unchanged after each ICP-driven refactoring. Run the tests before and after.
