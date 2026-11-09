---
description: 'Cognitive Complexity (Sonar) for Java: scoring rules for control-flow understandability, per-method limit and refactoring tactics'
applyTo: '**/*.java'
---

# Cognitive Complexity for Java

Limit the control-flow complexity of each method so it can be read top to bottom without effort. Based on the Cognitive Complexity metric (G. Ann Campbell, SonarSource), the same metric as Sonar rule `java:S3776`.

## Relation to Other Guidelines

- This file governs **control flow per method**.
- `java-cdd.instructions.md` governs **coupling and size per unit**, with team-defined limits.
- Apply both. Do not add the two scores together.
- Methods are not scored for existing. Extracting a well-named method is encouraged (it "shorthands" several statements into one call).

## Scoring

Add points to a method's score as follows.

### Structural: +1, plus the current nesting level

`if`, ternary operator, `switch` (statement or expression, all cases combined), `for`, enhanced `for`, `while`, `do while`, `catch`.

### Hybrid: +1, no nesting penalty

`else if`, `else`. They still raise the nesting level for what is inside them.

### Fundamental: +1

- Each **sequence** of identical binary logical operators (`a && b && c` = +1; `a && b || c` = +2).
- Each method in a recursion cycle, direct or indirect.
- `break` or `continue` with a label.

### Nesting level

The nesting level increases inside: `if`, `else if`, `else`, ternary, `switch`, loops, `catch`, and method-like structures (lambdas, anonymous classes, nested methods). A lambda adds +0 itself, but structures inside it are scored with the raised nesting level.

### Not scored

Method declarations, `try`, `finally`, early `return`, unlabeled `break`/`continue`, and calls such as `Optional.map(...).orElse(...)`.

### Example

```java
static int countValid(List<Order> orders, Set<String> blocked) {
    int count = 0;
    for (Order o : orders) {                                    // +1
        if (o.isActive() && !blocked.contains(o.id())) {        // +2 (nesting 1), +1 for &&
            if (o.total().signum() > 0) {                       // +3 (nesting 2)
                count++;
            }
        }
    }
    return count;
}   // score 7
```

```java
static long countValid(List<Order> orders, Set<String> blocked) {
    return orders.stream()
        .filter(o -> isCountable(o, blocked))                   // lambda: +0
        .count();
}   // score 0

static boolean isCountable(Order o, Set<String> blocked) {
    return o.isActive() && !blocked.contains(o.id()) && o.total().signum() > 0;   // +1
}   // score 1
```

## Limits

| Unit | Limit |
|---|---|
| Method | 15 (default of `java:S3776`). Use the team's value if defined |
| Class | Sum of its methods' scores. Use to compare classes. Apply a limit only if the team defines one |

- A method above the limit requires refactoring.
- A method above 80% of the limit is under watch. Do not add branches to it without need.

## Rules When Writing or Changing Code

- Estimate the score of the method you change before and after.
- Do not push a method over its limit. Refactor first, or put the new logic in a new method.
- When you refactor because of the limit, state the before and after scores.
- Measuring is not evaluating. If a method is over the limit and splitting it would hurt cohesion or readability, keep it and state the reason in one line.

## Reducing the Score

| Source of score | Action |
|---|---|
| Deep nesting | Use guard clauses (early return). Flatten with `continue`. Extract the inner block |
| `if / else if` chain on one variable | Use `switch` (+1 total). For closed variants, use `sealed` + `switch` |
| Long boolean expression | Extract into a named `boolean` method. Avoid mixing `&&` and `||` |
| Loop with conditions inside | Use a stream pipeline with named predicates, or extract the loop body |
| `catch` nested in logic | Move exception handling to the boundary. Use `Result` for expected errors |
| Control flow inside lambdas | Extract into a named method and use a method reference |
| Recursion | Use a loop or stream when it reads equally well |

- Extraction moves complexity, it does not remove it. Extract only when the new method has a meaningful name and a single purpose.
- Do not split a cohesive method into fragments to lower the number.

## Avoid

- Adding a branch to a method already near its limit.
- Nesting a conditional inside a loop inside a conditional when a guard clause or extraction is possible.
- Complex logic inside lambdas.
- Extracting methods named `step1`, `process2` or similar only to lower the score.

## Validation

- Sonar rule: `java:S3776` (SonarLint in the IDE, SonarQube/SonarCloud in CI).
- Build and tests: `mvn verify` or `./gradlew build`.
- Behavior stays unchanged after each refactoring. Run the tests before and after.
