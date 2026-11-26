---
description: 'Java project conventions: security, money and time types, exception mapping, transactions, dependencies, migrations, formatting'
applyTo: '**/*.java'
---

# Java Conventions

## Security
- Validate input at the boundary (`jakarta.validation` on request records).
- Build queries with parameters (named parameters, JPA criteria, prepared statements).
- Read secrets from configuration or the environment; keep them out of code, tests and logs.

## Types
- Represent money as `BigDecimal` or a `Money` record.
- Represent time as `Instant`, `LocalDate` or `Clock`-based values.
- Use records over Lombok `@Data`; use Lombok only where the project already does.

## Errors and transactions
- Map exceptions to HTTP responses in one place (`@RestControllerAdvice`, `ProblemDetail`).
- Place `@Transactional` on application service methods; controllers and domain functions stay outside the transaction boundary.

## Dependencies and data
- Prefer the JDK and libraries already in the build file; ask before adding a dependency.
- Change schemas through the migration tool in use (Flyway or Liquibase) with a new migration; applied migrations stay untouched.

## Style
- Format with the project formatter (Spotless, google-java-format) and leave untouched code as it is.
- Keep changes surgical: touch only what the task needs.
