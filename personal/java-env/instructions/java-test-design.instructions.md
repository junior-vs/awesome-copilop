---
description: 'Test engineering for Java: integrated-first strategy with a time budget, coverage target, MC/DC, boundary and property-based testing, structure and determinism'
applyTo: '**/*Test.java, **/*IT.java, **/*Steps.java, **/*StepDefs.java'
---

# Test Design for Java

Automated tests have two purposes: raise the reliability of the delivered software and raise the team's confidence to change code. Assertion syntax is in `java-junit5-assertions.instructions.md`. Mocks are in `java-mockito-mocks.instructions.md`.

Project conventions take precedence over these generic guidelines. When they conflict, follow the project and say so.

## Strategy: Most Integrated Within the Time Budget

- Write the most integrated test that runs within the suite's time limit. The closer to production conditions, the higher the confidence.
- Pure functions, value objects and domain rules need no infrastructure, so they are tested directly (input → output).
- Test adapters (repository, HTTP client, messaging) against the real technology (Testcontainers) when the cost is acceptable.
- When a resource is too slow or costly to start, replace it with a fake first, then a mock (see `java-mockito-mocks.instructions.md`).
- Monitor suite execution time. The team defines the limit. When a test slows the build, reduce its integration depth, not its assertions.
- Do not start a Spring context for logic that does not need it.

| Target | Typical test | Suffix |
|---|---|---|
| Pure functions, value objects, domain rules | Direct call, no doubles | `*Test` |
| Application services and use cases | Real collaborators, fakes or mocks for slow resources | `*Test` |
| Adapters and full flows | Real infrastructure | `*IT` |

## Coverage

- Target 90% or more coverage.
- Coverage counts only when tests explore the decision paths. Touching a line is not enough.
- Do not judge test quality by coverage alone. A single metric invites tests without assertions or with few assertions.
- Infrastructure code loaded by the framework and not called directly may stay uncovered.
- A test must fail when the behavior changes. A refactoring that changes behavior and breaks no test shows a gap.

## Techniques

- **MC/DC (modified condition/decision coverage)**: for a decision with several conditions, write tests where changing a single condition, with the others fixed, changes the decision. Cover each condition this way.
- **Boundary testing**: for each comparison (`>`, `<`, `>=`), test the value at the limit, just below and just above.
- **Property-based testing**: after the systematic tests, generate values within ranges and types (for example jqwik) to explore further.
- JaCoCo reports line and branch coverage. MC/DC is verified by test design, with one truth-table row per condition under test.

## Structure

- Follow Arrange / Act / Assert (Given / When / Then), separated by blank lines.
- Test one behavior per test. Several assertions are fine when they describe the same behavior (`assertAll`).
- Keep the code that matters for the scenario visible in the test. Hide only irrelevant setup.

```java
@Test
void shouldRejectPaymentWhenOrderIsCanceled() {
    var order = canceledOrder();

    var result = OrderRules.pay(order, "tx-1", NOW);

    var err = assertInstanceOf(Result.Err.class, result);
    assertInstanceOf(OrderError.Canceled.class, err.error());
}
```

## Naming

- Name tests by behavior and condition: `shouldRejectPaymentWhenOrderIsCanceled`.
- Use `@DisplayName` only when the project already does.
- Name the test class `<ClassUnderTest>Test` (unit, component) or `<ClassUnderTest>IT` (integration).
- Group scenarios of one method with `@Nested` classes when a class grows beyond a handful of tests.

## Qualities of a Good Test (FIRST)

- **Fast**, **Isolated**, **Repeatable**, **Self-validating**, **Timely**.
- For a bug fix, write the failing regression test first.

## Test Behavior, Not Implementation

- Assert on observable outcomes: return values, state changes, messages sent to collaborators.
- Do not assert on private methods, call order or internal structure.
- A refactoring that keeps behavior must not break tests.

## Determinism

- Use `Clock.fixed(...)` or pass `Instant` as a parameter. Do not call `Instant.now()` in tests or in code under test.
- Do not use `Thread.sleep`. Use Awaitility for asynchronous conditions.
- Use `@TempDir` for files.
- Do not share mutable state between tests.
- Do not use random data without a fixed seed.

## Test Data

- Create data with test builders or object mothers (`anOrder()`, `canceledOrder()`).
- Set only the fields relevant to the scenario.
- Use the `with` methods of immutable `record` types to vary one field.

## Parameterized Tests

- Use `@ParameterizedTest` with `@CsvSource`, `@MethodSource` or `@EnumSource` for the same behavior with different inputs, including boundary values.
- Do not merge different behaviors into one parameterized test.

## Test Code Complexity

- Test files follow the CDD rules in `java-cdd.instructions.md` with their own limit.
- Keep conditionals and loops out of test methods.

## Avoid

- Tests that depend on execution order.
- `@Disabled` without a linked issue.
- Overly broad tests that fail for several unrelated reasons.
- Asserting on `toString()` or log output unless it is the contract.
- Catching exceptions manually. Use `assertThrows`.
- Raising coverage with tests that have no assertions.

## Validation

- Unit tests: `mvn test` or `./gradlew test`. Integration tests: `mvn verify` or `./gradlew check`.
- Coverage: `jacoco` report, target 90% or more.
- A new or changed behavior has at least one test that fails without the change.
