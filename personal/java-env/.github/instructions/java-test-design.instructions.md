---
description: 'Java test design: integrated-first strategy within a time budget, coverage, MC/DC and boundary techniques, structure, naming, determinism'
applyTo: '**/*Test.java, **/*IT.java, **/*Steps.java, **/*StepDefs.java'
---

# Java Test Design

Tests raise the reliability of the software and the team's confidence to change code.

## Strategy: most integrated within the time budget
- Write the most integrated test that runs within the suite's time limit; the closer to production, the higher the confidence.
- Test pure functions, value objects and domain rules by direct call (input → output), without doubles.
- Test application services with real collaborators; replace a slow or costly resource with a fake first, then a mock.
- Test adapters (repository, HTTP client, messaging) against the real technology (Testcontainers) when the cost is acceptable.
- Start a Spring context only for logic that needs it.
- When a test slows the build, reduce its integration depth and keep its assertions.

| Target | Test | Suffix |
|---|---|---|
| Pure functions, value objects, domain rules | Direct call | `*Test` |
| Application services, use cases | Real collaborators, fakes or mocks for slow resources | `*Test` |
| Adapters, full flows | Real infrastructure | `*IT` |

## Coverage and techniques
- Target 90% coverage or more, enforced by the build (JaCoCo `check`); coverage counts when tests assert on the decision paths.
- A test must fail when behavior changes.
- MC/DC: for a decision with three or more conditions, write one test per condition where flipping only that condition flips the decision.
- Boundary: for each comparison (`>`, `<`, `>=`), test the limit value, just below and just above.
- Property-based (jqwik) after the systematic tests, when the project already uses it.

## Structure and naming
- Follow Arrange / Act / Assert separated by blank lines; one behavior per test (several assertions are fine for the same behavior).
- Keep scenario-relevant data visible in the test; hide irrelevant setup in builders or object mothers (`anOrder()`, `canceledOrder()`), and vary one field with `with` methods of a `record`.
- Name tests by behavior and condition: `shouldRejectPaymentWhenOrderIsCanceled`; name classes `<Class>Test` or `<Class>IT`; group scenarios with `@Nested`.
- Use `@ParameterizedTest` (`@CsvSource`, `@MethodSource`, `@EnumSource`) for one behavior with many inputs, including boundaries.
- For a bug fix, write the failing regression test first.

```java
@Test
void shouldRejectPaymentWhenOrderIsCanceled() {
    var order = canceledOrder();

    var result = OrderRules.pay(order, "tx-1", NOW);

    assertEquals(new Result.Err<>(new OrderError.Canceled()), result);
}
```

## Behavior, not implementation
- Assert on return values, state changes and messages sent to collaborators.
- A refactoring that keeps behavior leaves every test green.

## Determinism
- Inject `Clock.fixed(...)` or pass an `Instant`.
- Use Awaitility for asynchronous conditions, `@TempDir` for files, and a fixed seed for random data.
- Keep tests independent of execution order and of shared mutable state.
- Link every `@Disabled` to an issue.
- Keep conditionals and loops out of test methods.

## Verify
- Unit tests: `mvn test` or `./gradlew test`; integration tests: `mvn verify` or `./gradlew check`.
- Each new or changed behavior has a test that fails without the change.
