---
description: 'Test engineering for Java: test levels, structure, naming, determinism, test data, parameterized tests, boundary analysis and what to cover'
applyTo: '**/*Test.java, **/*IT.java, **/*Steps.java, **/*StepDefs.java'
---

# Test Design for Java

Strategy and structure of tests. Assertion syntax is in `java-junit5-assertions.instructions.md`. Mocks are in `java-mockito-mocks.instructions.md`. Existing project conventions take precedence over the defaults below.

## Choose the Test Level

| Level | Target | Dependencies | Suffix |
|---|---|---|---|
| Unit | Pure functions, value objects, domain rules | None. No mocks, no Spring context | `*Test` |
| Component | Application services and orchestration | Mocks or fakes for ports | `*Test` |
| Integration | Adapters (repository, HTTP client, messaging) | Real infrastructure via Testcontainers | `*IT` |
| End-to-end | Critical user flows | Full system | `*IT` |

- Write many unit tests, fewer component tests, few integration and end-to-end tests.
- Test pure functions of the functional core as plain input → output.
- Test an adapter against the real technology. Do not mock the database, HTTP server or broker to test its own adapter.
- Do not start a Spring context for logic that does not need it.

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

- **Fast**: unit tests run in milliseconds.
- **Isolated**: no dependency on execution order or on other tests.
- **Repeatable**: same result on every run and environment.
- **Self-validating**: pass or fail through assertions, never by reading output.
- **Timely**: written with or just before the code. For a bug fix, write the failing regression test first.

## Test Behavior, Not Implementation

- Assert on observable outcomes: return values, state changes, messages sent to collaborators.
- Do not assert on private methods, call order or internal structure.
- A refactoring that keeps behavior must not break tests.

## Determinism

- Use `Clock.fixed(...)` or pass `Instant` as a parameter. Do not call `Instant.now()` in tests or in code under test.
- Do not use `Thread.sleep`. Use Awaitility for asynchronous conditions.
- Use `@TempDir` for files.
- Do not share mutable state between tests (static fields, singletons).
- Do not use random data without a fixed seed.

## Test Data

- Create data with test builders or object mothers (`anOrder()`, `canceledOrder()`).
- Set only the fields relevant to the scenario. Let defaults fill the rest.
- Use the `with` methods of immutable `record` types to vary one field.
- Do not copy large literals across tests.

## Parameterized Tests

- Use `@ParameterizedTest` with `@CsvSource`, `@MethodSource` or `@EnumSource` when the same behavior is checked with different inputs.
- Do not use parameterization to merge different behaviors into one test.

```java
@ParameterizedTest
@CsvSource({"0, 0", "100, 90", "1000, 850"})
void shouldApplyTieredDiscount(int amount, int expected) {
    assertEquals(expected, Discount.apply(amount));
}
```

## What to Cover

- Cover each equivalence class of input and its boundaries (min, max, just below, just above, empty, null where allowed).
- Cover every variant of a `sealed` type and every branch of its `switch`.
- Cover error paths and invalid input, not only the happy path.
- Treat coverage as an indicator. Do not write tests only to raise a number.

## Avoid

- Tests that depend on execution order.
- Logic (`if`, loops) inside tests.
- `@Disabled` without a linked issue.
- Overly broad tests that fail for several unrelated reasons.
- Asserting on `toString()` or log output unless it is the contract.
- Catching exceptions manually. Use `assertThrows`.

## Validation

- Run unit tests: `mvn test` or `./gradlew test`.
- Run integration tests: `mvn verify` or `./gradlew check`.
- A new or changed behavior has at least one test that fails without the change.
