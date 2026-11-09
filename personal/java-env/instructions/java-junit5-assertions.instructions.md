---
description: 'JUnit 5 (Jupiter) assertion rules: static imports, argument order, lazy messages, assertAll, exceptions, timeouts, type and collection assertions'
applyTo: '**/*Test.java, **/*IT.java, **/*Steps.java, **/*StepDefs.java'
---

# JUnit 5 Assertions

Rules for writing assertions with JUnit Jupiter (5.8+ and 6; the `org.junit.jupiter.api.Assertions` API is the same). Test structure and strategy are in `java-test-design.instructions.md`. Mocks are in `java-mockito-mocks.instructions.md`.

If the project already standardizes on AssertJ, use AssertJ for new assertions. Never rewrite a project's existing assertion style to match this file.

## Imports

- Use explicit static imports from `org.junit.jupiter.api.Assertions`.
- Do not use wildcard imports unless the team convention requires it.
- Do not mix `org.junit.Assert` (JUnit 4) with JUnit 5.

```java
// Bad
Assertions.assertEquals(expected, actual);
import static org.junit.jupiter.api.Assertions.*;

// Good
import static org.junit.jupiter.api.Assertions.assertEquals;
```

## Argument Order

- Pass `expected` first and `actual` second.
- Provide a delta for floating-point comparisons.

```java
// Bad: swapped, failure message is misleading
assertEquals(calculator.add(1, 1), 2);

// Good
assertEquals(2, calculator.add(1, 1));
assertEquals(0.3, 0.1 + 0.2, 1e-9);
```

## Failure Messages

- Use a `String` literal for constant messages.
- Use a `Supplier<String>` when the message is built with formatting or object inspection, so it is evaluated only on failure.

```java
// Bad: message built even when the assertion passes
assertEquals(expected, actual, "Expected %s but got %s".formatted(expected, actual));

// Good
assertEquals(expected, actual, () -> "Expected %s but got %s".formatted(expected, actual));
assertTrue(isActive, "User account must be active");
```

## Group Assertions: assertAll

- Use `assertAll` to check several properties of the same result. All assertions run even if one fails.
- Do not group assertions about unrelated behaviors. Split them into separate tests.

```java
assertAll("person",
    () -> assertEquals("Jane", person.firstName()),
    () -> assertEquals("Doe",  person.lastName()),
    () -> assertEquals(30,     person.age())
);
```

## Exceptions

- Use `assertThrows` to accept the type or any subclass.
- Use `assertThrowsExactly` (5.8+) when the exact type is part of the contract.
- Assert on the returned exception (message or cause) when it carries meaning.
- Use `assertDoesNotThrow` only when the absence of an exception is the contract. It returns the result for further assertions.

```java
var ex = assertThrows(ArithmeticException.class, () -> calculator.divide(1, 0));
assertEquals("/ by zero", ex.getMessage());

int result = assertDoesNotThrow(() -> service.calculate(data));
assertEquals(100, result);
```

## Timeouts

- Use `assertTimeout` to check that execution finishes within a limit.
- Use `assertTimeoutPreemptively` only when a hard abort is required. It runs in a separate thread, so `ThreadLocal` state (for example `@Transactional`) does not propagate.

## Types and Collections

- Use `assertInstanceOf` (5.8+) instead of `assertTrue(x instanceof T)`. It returns the cast object.
- Use `assertIterableEquals` and `assertArrayEquals` for element-by-element comparison.

```java
SuccessResponse resp = assertInstanceOf(SuccessResponse.class, result);
assertEquals(200, resp.statusCode());

assertIterableEquals(expectedList, actualList);
```

## Avoid

- `assertTrue(result == 42)`. Use `assertEquals(42, result)` to see both values in the failure.
- `assertNotNull(result)` as the only check when the value can be asserted.
- Catching `AssertionError` to hide a failure.
- Assertions inside loops or lambdas that can silently never run. Assert on the collection instead.
- Tests without assertions.
