---
description: 'Cognitive-Driven Development (CDD) for Java: ICP per file, multidimensional evaluation, limits by context and layer, periodic review'
applyTo: '**/*.java'
---

# Cognitive-Driven Development (CDD) for Java

Keep each file small enough to be understood without overloading working memory. Measure complexity with Intrinsic Complexity Points (ICP) and evaluate it with more than one metric.

Project conventions take precedence over these generic guidelines. When they conflict, follow the project and say so.

## Relation to Other Guidelines

- `java-cognitive-complexity.instructions.md` scores control flow per method (Sonar metric). It is one of the three metrics used below.
- Generalization and abstraction rules are in `java-design-principles.instructions.md`.
- Apply all of them. Do not add the scores together.

## Measure: ICP per File

Add 1 point for each element in the file.

| Element | Points |
|---|---|
| `if`, `else if`, `else`, ternary operator | 1 each |
| `switch` (statement or expression, all cases combined) | 1 per `switch` |
| Loop (`for`, enhanced `for`, `while`, `do`) | 1 each |
| `try/catch` block | 1 each |
| Distinct project class or interface referenced (coupling) | 1 each, counted once per file |

- Do not count as coupling: JDK types, primitives, the class itself, or classes from frameworks and third-party libraries.
- Do not count lambdas or method references passed as arguments.
- The file's ICP is the sum of its methods' points plus its coupling.

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
// ICP of this file so far = 7
```

## Evaluate: Three Metrics

Measuring is cold data. Evaluate with all three metrics together. A single metric pushes the code toward unwanted outcomes (for example, short files full of branches).

| Metric | Unit | Starting limit |
|---|---|---|
| ICP | file | See "Limits by Context" |
| Lines of code | file | 200 |
| Cognitive Complexity (Sonar) | method | 15 |

- A file that passes one metric and fails another still needs review.
- The team may replace or add metrics.

## Limits by Context

- **New project**: ICP limit of 10 per file. It forces early attention to structure.
- **Legacy project**: take the 5 most complex classes. The 5th one's ICP is the starting limit (for example, 50). Do not use a limit that every touched file would exceed.
- When a legacy file is touched, measure it. Use the results to see where refactoring is most needed.
- Raise or lower a limit only by team decision.

## Limits by Layer

- Apply stricter limits to the outermost layer (controllers, adapters).
- Use cases and domain entities may use different limits.
- Persistence classes may be measured by query complexity instead of ICP.
- Until the team defines per-layer values, use the file limit.

## Test Files

- Measure test files with a separate metric and limit. The team defines them.
- Until defined, do not enforce a limit on tests. Keep conditionals and loops out of tests.

## Rules When Writing or Changing Code

- Write the algorithm first, then measure, then reorganize. Do not optimize structure before the code works.
- Estimate the ICP of the affected file before and after the change.
- When a change would exceed a limit, tell the user and propose options. Do not contort the solution only to stay under the number.
- Measuring is not evaluating. If a file exceeds the limit and splitting it would break cohesion or readability, keep it and state the reason in one line.
- Do not split a cohesive file into meaningless fragments to lower the number.

## Reducing ICP

| Symptom | Action |
|---|---|
| Many conditionals | Guard clauses (early return), `sealed` + `switch`, polymorphism, or a `Map<Key, Function<...>>` |
| Nested conditionals | Extract a named method or `Predicate` |
| Loop with accumulation | Extract into a pure function or stream pipeline |
| `try/catch` in business logic | Move to the boundary. Use `Result` for expected errors |
| Many project classes referenced | Pass only the needed data (a `record`), depend on a port, split responsibilities |
| File over limit | Extract a class by responsibility. Move rules into pure functions in a separate class |

- Extracting private methods does not lower the file's ICP. When the file is over its limit, extract a class.

## Team Decisions (edit as the team decides)

- `switch` counts 1 per block. Alternative: 1 per `case`.
- Lambdas and method references passed as arguments do not count. Alternative: 1 each.
- Framework and library classes do not count as coupling. Alternative: count them.

## Review

- Review limits and metrics every sprint or every 4 weeks, with team feedback.
- Record each change of limit and the reason.

## Avoid

- Adding branches to a file already near its limit.
- Treating the limit as the goal. The goal is code a teammate understands quickly.
- Hiding complexity by moving code between methods of the same over-limit file.

## Validation

- Build and tests: `mvn verify` or `./gradlew build`.
- Behavior stays unchanged after each ICP-driven refactoring. Run the tests before and after.
