---
description: 'Java logging: levels, SLF4J parameterized messages, context fields, sensitive data, no duplicate error logs'
applyTo: '**/*.java'
---

# Java Logging

Log so that a human or a tool can form and check hypotheses when a problem appears.

## Levels
| Situation | Level |
|---|---|
| Call to an external service (API, other system) | `INFO`, one entry before and one after |
| State change (save, update, delete) | `INFO` after; `DEBUG` before when it helps diagnose failures |
| Error handled while the flow continues | `ERROR` |
| Decision path where a problem is likely | `DEBUG`, sparingly |

## How
- Use the project's standard logging component. When none exists, use SLF4J and ask before creating a wrapper.
- Write parameterized messages: `log.info("order paid orderId={}", id)`.
- Include at least one identifying context field (id, who, what) in every entry; the team may define more (who, when, where, what, why).
- Let the caller choose the level.
- Mask secrets, tokens, passwords and personal data before logging.

## Errors
- Log an exception at the layer that handles it. When you wrap or remap an exception, throw it without logging it.
- Keep `ERROR` and rethrow apart: a `catch` block either handles (and logs) or rethrows.

## Placement
- Log in application services and adapters. Pure domain functions stay silent.

## Example

```java
public Result<Order, OrderError> pay(Long id, String txId) {
    log.info("paying order orderId={} txId={}", id, txId);
    var result = findAndPay(id, txId);
    log.info("payment finished orderId={} outcome={}", id, result.getClass().getSimpleName());
    return result;
}
```

## Verify
- Every external call has `INFO` entries before and after.
- No `catch` block logs at `ERROR` and rethrows.
