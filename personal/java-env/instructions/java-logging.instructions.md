---
description: 'Logging rules for Java: when to log, which level, structured format and fields, error handling without duplicate logs'
applyTo: '**/*.java'
---

# Logging for Java

Log systematically so humans and tools can form and check hypotheses when a problem appears. Every developer must know when to log and at which level.

Project conventions take precedence over these generic guidelines. When they conflict, follow the project and say so.

## When and Which Level

| Situation | Level |
|---|---|
| State change (save, update, delete) | `INFO`, one entry before and one after |
| Call to an external service (API, other systems) | `INFO`, one entry before and one after |
| Error handled and the application flow continues | `ERROR` |
| Decision paths (`if`, loops) where a problem is likely | `DEBUG`, sparingly |

- Do not log `ERROR` and rethrow. The upper layer decides what to do, and duplicate logs of one problem hide the real flow.
- When wrapping or remapping an exception, throw it and let the layer that handles it log it.
- Use `DEBUG` with restraint. Not every statement needs a log.

## How to Log

- Use the project's standard logging component. It forces relevant information and serializes the log in the agreed format.
- If the project has none, ask before creating one. A small builder that accumulates fields and emits a structured (JSON) log is the suggested approach.
- Required fields are defined by the team. Suggested base (5W): who, when, where, what, why.
- Include at least one piece of relevant context in every log entry.
- The caller chooses the level. The component does not.

## Where Not to Log

- Pure functions of the domain core do not log. Log in the application service or adapter that calls them.

## Good

```java
public Order pay(Long id, String txId) {
    log.info(entry("paying order").with("orderId", id).with("txId", txId));
    var paid = repository.save(OrderRules.pay(load(id), txId, clock.instant()));
    log.info(entry("order paid").with("orderId", id));
    return paid;
}
```

## Bad

```java
try {
    repository.save(order);
} catch (DataAccessException e) {
    log.error("failed", e);            // logs and rethrows: duplicate logs upstream
    throw new OrderPersistenceException(e);
}
```

## Validation

- Every state change and every external call has an `INFO` entry before and after.
- No `catch` block both logs at `ERROR` and rethrows.
- Logs use the project's standard format.
