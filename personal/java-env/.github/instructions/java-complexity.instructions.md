---
description: 'Java complexity limits: Cognitive Complexity per method, ICP per file (CDD), lines per file, and how to reduce them'
applyTo: '**/*.java'
---

# Java Complexity

Keep each file small enough to read without overloading working memory. Three metrics are evaluated together; a file that passes one and fails another still needs review.

| Metric | Unit | Starting limit |
| --- | --- | --- |
| Cognitive Complexity (Sonar `java:S3776`) | method | 15 |
| ICP, Intrinsic Complexity Points (CDD) | file | 15 in a new project; in a legacy project, the ICP of its 5th most complex class |
| Lines of code | file | 200 |

The team defines the final limits (per layer too); tools measure them in CI. Take the limits from the project configuration when present.

## Cognitive Complexity (per method)

- +1 plus the current nesting level: `if`, ternary, `switch`, `for`, `while`, `do`, `catch`.
- +1 flat: `else if`, `else`, each sequence of identical `&&`/`||`, each method in a recursion cycle, labeled `break`/`continue`.
- Nesting grows inside those structures and inside lambdas.
- Method declarations, `try`, `finally` and early `return` score 0.

## ICP (per file)

- +1 each: `if`, `else if`, `else`, ternary, `switch` (all cases together), loop, `try/catch`.
- +1 per distinct project class or interface referenced (JDK, framework and third-party types do not count).
- Lambdas and method references passed as arguments do not count.

## Reduce complexity

| Symptom | Action |
| --- | --- |
| Deep nesting | Guard clauses (early return), `continue`, extract the inner block |
| `if / else if` chain on one variable | `switch`; for closed variants, `sealed` + `switch` |
| Long boolean expression | Named `boolean` method |
| Loop with conditions | Stream with named predicates |
| `try/catch` in business logic | Move to the boundary; `Result` for expected errors |
| Many project classes referenced | Pass a `record` with the needed data; depend on a port; split responsibilities |
| File over limit | Extract a class by responsibility; private methods do not lower the file's ICP |

## Rules for changes

- Write the algorithm first, then measure, then reorganize.
- Refactor first (or put the new logic in a new method or class) when a change would push a method or file over its limit, or when the method is above 80% of it.
- State before and after scores when a refactoring is driven by a limit.
- Keep a cohesive file or method that is over the limit when splitting would hurt readability, and state the reason in one line.
- Give extracted methods a meaningful name and a single purpose.

## Verify

- Sonar rule `java:S3776` (SonarLint, SonarQube) and the project's build; behavior stays the same after each refactoring, with tests run before and after.
