---
description: 'How to combine functional programming with object-oriented design in Java: functional core, OO shell, layers, dependencies as functions and persistence mapping'
applyTo: '**/*.java'
---

# Functional and Object-Oriented Integration in Java

Architecture-level guidance for combining both paradigms. Language techniques are in `java-functional-programming.instructions.md`.

## Core Approach

**Functional core, object-oriented shell** (*Functional Core, Imperative Shell*).

- State and identity (entities, connections, lifecycle) → **objects**.
- Transformation and decision (rules, calculations, validations) → **pure functions**.
- Do not force full purity where it hurts clarity.

## Layers

```
Controller / Handler        (shell: I/O, HTTP)
   ↓
Application service         (orchestrates: fetch, call rule, persist)
   ↓
Pure domain functions       (core: rules, calculations, validations)
```

- Controllers and application services orchestrate and execute effects (database, network, events, clock).
- The domain core does not import infrastructure classes (repository, `Clock`, web framework).
- Inject `Clock`, repositories and gateways through the constructor. Pass the current time to rules as a parameter.

### Good

```java
@Service
public class OrderService {
    private final OrderRepository repository;
    private final Clock clock;

    public OrderService(OrderRepository repository, Clock clock) {
        this.repository = repository;
        this.clock = clock;
    }

    public Result<Order, OrderError> pay(Long id, String txId) {
        return repository.findById(id)                                  // I/O
            .map(o -> OrderRules.pay(o, txId, clock.instant()))         // pure
            .map(r -> r.map(repository::save))                          // I/O
            .orElseGet(() -> new Result.Err<>(new OrderError.NotFound(id)));
    }
}
```

## Functions as Dependencies

- Receive a `Function`, `Predicate` or `Supplier` when the collaborator is a single behavior with no domain name (pricing rule, filter, formatter).
- Create a dedicated interface when the contract has a domain name, more than one method, or is a port to infrastructure (repository, gateway).

## OO Pattern Equivalents

| OO pattern | Functional equivalent |
|---|---|
| Strategy | Function passed as argument |
| Command | Function / closure |
| Template Method | Higher-order function |
| Observer | Streams / callbacks |
| Decorator | Function composition |
| Builder | Immutable copy (`with`) |
| Visitor | Pattern matching on `sealed` type |

## Persistence (JPA)

- Keep JPA entities (mutable, no-arg constructor) inside the persistence layer.
- Map to an immutable domain `record` at the boundary. Pure functions receive only the `record`.

## Testing

- Test the core without mocks.
- Use mocks only in the shell (services, repositories, gateways).
- Inject a fixed `Clock` in shell tests (`Clock.fixed(...)`).

## Avoid

- Business rules mixed with I/O in the same method.
- Mutable anemic objects changed by several services.
- Shared mutable state "protected" only by being inside an object.
- Domain core depending on repositories, `Clock` or framework classes.

## Validation

- Check that no class in the domain package imports infrastructure packages.
- Build and tests: `mvn verify` or `./gradlew build`.
