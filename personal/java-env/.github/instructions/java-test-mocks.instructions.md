---
description: 'Mockito and test doubles for Java: when to mock, stubbing, verification, captors, strict stubs, spies, static mocking'
applyTo: '**/*Test.java, **/*IT.java, **/*Steps.java, **/*StepDefs.java'
---

# Mocks and Test Doubles

## When to use which double
| Double | Use |
|---|---|
| Real collaborator | Default |
| Fake (in-memory implementation) | Many tests share the behavior, or the interaction is stateful |
| Stub (`when(...).thenReturn(...)`) | Canned answers |
| Mock (`verify(...)`) | Force an error, or verify that a side effect was requested |

- Mock or fake only a resource that is too slow or costly to start: external service, broker, remote API.
- Mock ports at the boundary of the code under test: repositories, gateways, publishers, external clients.
- Create real instances of pure functions, value objects, `record` types, DTOs, domain entities, collections, `String` and `Optional`.
- Wrap third-party clients in your own adapter, mock the adapter, and cover the adapter with an integration test.
- Treat more than three mocks in one test as a sign the class has too many responsibilities, and refactor the class.
- Use a fixed `Clock` (`Clock.fixed(...)`) in place of a mock.

## Setup
- Use `@ExtendWith(MockitoExtension.class)` with `@Mock`; build the class under test in `@BeforeEach` with its constructor.
- In Spring tests, use `@MockitoBean` (`@MockBean` is deprecated).
- Keep strict stubs on and remove unused stubs instead of `lenient()`.

## Stubbing and verification
- Stub only what the scenario needs: `thenReturn` for values, `thenThrow` for failures, `thenAnswer` to echo arguments, `doThrow(...).when(mock)` for `void`.
- Use matchers for all arguments of a call or for none (`eq()` wraps raw values).
- Verify commands (`save`, `publish`, `send`) and assert on results for queries you stubbed.
- Use `verify(mock, never())` for effects that must not happen; `times(1)` is the default.
- Inspect passed objects with `ArgumentCaptor`, then assert on the captured value.
- Use `verifyNoMoreInteractions` and `InOrder` only when extra calls or order are the defect.
- Replace `spy` by extracting the part to replace into a collaborator. For `mockStatic` and `mockConstruction`, inject the dependency instead; when unavoidable, open them in try-with-resources.

## Example

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

## Verify
- `UnnecessaryStubbingException` means an unused stub: remove it.
- A test that breaks on a behavior-preserving refactoring over-specifies interactions: reduce the verifications.
