---
description: 'JUnit 5 assertion rules: static imports, argument order, lazy messages, assertAll, exceptions, timeouts, type and collection assertions'
applyTo: '**/*Test.java, **/*IT.java, **/*Steps.java, **/*StepDefs.java'
---

# JUnit 5 Assertions

Use the assertion library the project already uses (JUnit or AssertJ) and keep its existing style. In a new project, use JUnit Jupiter as below.

## Imports and order
- Use explicit static imports: `import static org.junit.jupiter.api.Assertions.assertEquals;`; avoid wildcards and `org.junit.Assert` (JUnit 4).
- Pass `expected` first and `actual` second: `assertEquals(2, calculator.add(1, 1))`.
- Provide a delta for floating point: `assertEquals(0.3, 0.1 + 0.2, 1e-9)`.

## Messages
- Use a `String` literal for constant messages.
- Use a `Supplier<String>` for built messages, evaluated only on failure:
  `assertEquals(expected, actual, () -> "Expected %s but got %s".formatted(expected, actual));`

## Group, exceptions, timeouts
- Use `assertAll` for several properties of one result; split unrelated behaviors into separate tests.

```java
assertAll("person",
    () -> assertEquals("Jane", person.firstName()),
    () -> assertEquals(30,     person.age()));
```

- Use `assertThrows` for a type or subclass and `assertThrowsExactly` when the exact type is the contract; assert on the returned exception's message or cause when it carries meaning.
- Use `assertDoesNotThrow` when the absence of an exception is the contract; it returns the result for further assertions.
- Use `assertTimeout` for time limits. Use `assertTimeoutPreemptively` only when a hard abort is required: it runs in another thread, so `ThreadLocal` state such as `@Transactional` does not propagate.

## Types and collections
- Use `assertInstanceOf` (it returns the cast object) and `assertEquals` on records for value comparison.
- Use `assertIterableEquals` and `assertArrayEquals` for element-by-element comparison.

```java
SuccessResponse resp = assertInstanceOf(SuccessResponse.class, result);
assertEquals(200, resp.statusCode());
```

## Quality checks
- Prefer `assertEquals(42, result)` to `assertTrue(result == 42)`, so the failure shows both values.
- Assert on the value itself; use `assertNotNull` only when nullness is the behavior.
- Assert on collections directly instead of inside loops or lambdas that may never run.
- Give every test at least one assertion; let `AssertionError` propagate.
