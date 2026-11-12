---
description: 'Mockito and test double rules for Java tests: when to mock, stubbing vs verification, captors, strict stubs, spies and static mocking'
applyTo: '**/*Test.java, **/*IT.java, **/*Steps.java, **/*StepDefs.java'
---

# Mocks and Test Doubles in Java

Rules for using Mockito with JUnit 5. Test strategy is in `java-test-design.instructions.md`. Assertions are in `java-junit5-assertions.instructions.md`.

Project conventions take precedence over these generic guidelines. When they conflict, follow the project and say so.

## Test Doubles

| Double | Purpose | Use |
|---|---|---|
| Dummy | Fills a required parameter | Plain object or `null`-free placeholder |
| Stub | Returns canned answers | `when(...).thenReturn(...)` |
| Fake | Working lightweight implementation | In-memory repository |
| Mock | Verifies an interaction happened | `verify(...)` |
| Spy | Real object with partial override | Avoid |

## When to Mock

- Prefer the real collaborator. Mock or fake only a resource that is too slow or costly to start and prepare, such as an external service, a broker or a remote API.
- Prefer a **fake** (lightweight implementation with the same public behavior) when many tests share the same collaborator behavior or the interaction is stateful.
- Prefer a **mock** when you must force an error or verify that a side effect was requested.
- Mock **ports at the boundary of the code under test**: repositories, gateways, message publishers, external clients.
- Do not mock:
  - Pure functions, value objects, `record` types, DTOs, domain entities. Create real instances.
  - Collections, `String`, `Optional` or other JDK types.
  - The class under test.
  - Types you do not own (third-party clients). Wrap them in your own adapter, mock the adapter, and test the adapter with an integration test.
- A test that needs more than three mocks signals a class with too many responsibilities. Refactor the class.

## Setup

- Use `@ExtendWith(MockitoExtension.class)` with `@Mock`.
- Build the class under test explicitly in `@BeforeEach` with its constructor. Use `@InjectMocks` only for trivial cases.
- Keep Mockito strict stubs enabled (the extension default). Remove unused stubs instead of using `lenient()`.
- Inject a fixed `Clock` instead of mocking it.

## Stubbing

- Stub only what the scenario needs.
- Use `thenReturn` for values, `thenThrow` for failures, `thenAnswer` to echo arguments.
- Use `doThrow(...).when(mock).method()` for `void` methods.
- Use argument matchers for all arguments of a call or for none. Do not mix raw values and matchers (use `eq()`).

## Verification

- Verify **commands**: calls that cause side effects (`save`, `publish`, `send`).
- Do not verify **queries**: reads that you already stubbed. Assert on the result instead.
- Use `verify(mock, never())` for effects that must not happen.
- Omit `times(1)`. It is the default.
- Use `ArgumentCaptor` to inspect the object passed to a collaborator, then assert on it.
- Do not use `verifyNoMoreInteractions` as a blanket check. Use it only when additional calls would be a defect.
- Do not verify call order unless order is the behavior (`InOrder`).

## Spies and Static Mocking

- Avoid `spy`. Extract the part that needs replacing into a collaborator and mock that.
- Avoid `mockStatic` and `mockConstruction`. Inject the dependency instead. When unavoidable, open them in try-with-resources so they close after the test.

## Good

```java
@ExtendWith(MockitoExtension.class)
class OrderServiceTest {
    private static final Instant NOW = Instant.parse("2026-01-01T10:00:00Z");

    @Mock OrderRepository repository;
    private OrderService service;

    @BeforeEach
    void setUp() {
        service = new OrderService(repository, Clock.fixed(NOW, ZoneOffset.UTC));
    }

    @Test
    void shouldSavePaidOrderWhenOrderIsCreated() {
        when(repository.findById(1L)).thenReturn(Optional.of(createdOrder(1L)));
        when(repository.save(any(Order.class))).thenAnswer(inv -> inv.getArgument(0));

        service.pay(1L, "tx-1");

        var saved = ArgumentCaptor.forClass(Order.class);
        verify(repository).save(saved.capture());
        var paid = assertInstanceOf(OrderStatus.Paid.class, saved.getValue().status());
        assertEquals("tx-1", paid.transactionId());
    }
}
```

## Bad

```java
OrderRepository repository = mock(OrderRepository.class);
Order order = mock(Order.class);                       // mocking a domain object
when(order.status()).thenReturn(new OrderStatus.Created(NOW));
lenient().when(repository.findAll()).thenReturn(List.of());   // silences an unused stub

service.pay(1L, "tx-1");

verify(repository, times(1)).findById(1L);             // verifies a query
verifyNoMoreInteractions(repository);                  // couples test to implementation
```

## Validation

- Run: `mvn test` or `./gradlew test`.
- An `UnnecessaryStubbingException` means the stub is unused. Remove it.
- A test that breaks when the implementation is refactored without changing behavior over-specifies interactions. Reduce the verifications.
