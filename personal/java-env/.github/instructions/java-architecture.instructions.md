---
description: 'Functional core and object-oriented shell in Java: layers, effects at the edge, functions as dependencies, use-case classes, request DTOs, JPA mapping'
applyTo: '**/*.java'
---

# Java Architecture: Functional Core, Object-Oriented Shell

- Model state and identity (entities, connections, lifecycle) as objects.
- Write rules, calculations and validations as pure functions.
- Keep purity where it helps clarity.

## Layers

```
Controller / Handler    shell: I/O, HTTP
Application service     orchestrates: fetch, call rule, persist
Pure domain functions   core: rules, calculations, validations
```

- Controllers and application services orchestrate and run effects (database, network, events, clock).
- Keep the domain core free of imports from infrastructure (repository, `Clock`, web framework).
- Inject `Clock`, repositories and gateways through the constructor; pass the current time to rules as a parameter.

```java
public Result<Order, OrderError> pay(Long id, String txId) {
    return repository.findById(id)                                        // I/O
        .map(order -> OrderRules.pay(order, txId, clock.instant()))       // pure
        .map(result -> result.map(repository::save))                     // I/O
        .orElseGet(() -> new Result.Err<>(new OrderError.NotFound(id)));
}
```

## Functions as dependencies
- Receive a `Function`, `Predicate` or `Supplier` for a single behavior without a domain name (pricing rule, filter, formatter).
- Create a dedicated interface for a contract with a domain name, several methods, or a port to infrastructure.

## Use-case classes
- Give each use case its own class (`NewAccountController`, `NewAccountUseCase`); one public method is typical.
- Keep the class 100% cohesive: every field is used by every method, and fields are dependencies only (repositories, gateways, `Clock`).
- Group operations by use case, not by resource name.
- Write stateless operations as static methods.

## Request DTOs
- Declare request and form DTOs as `record`.
- Allow validation rules, derived values and conversion to the domain object (`toModel(...)`) on the DTO's own data.
- Keep I/O, repository access and rules that need other aggregates out of DTOs.

## Persistence (JPA)
- Keep JPA entities (mutable, no-arg constructor) inside the persistence layer.
- Map to an immutable domain `record` at the boundary; pure functions receive only the `record`.

## Patterns as functions
Strategy → function argument. Command → closure. Template Method → higher-order function. Observer → stream or callback. Decorator → composition. Builder → `with` copy. Visitor → `switch` on a `sealed` type.

## Verify
- Check that no class in the domain package imports infrastructure packages (enforce with ArchUnit).
