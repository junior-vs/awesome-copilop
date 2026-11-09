---
description: 'Prioritized design principles for Java: SOLID, Clean Code, DRY, KISS, SoC, program to interfaces, coupling/cohesion, composition over inheritance, encapsulation, YAGNI'
applyTo: '**/*.java'
---

# Design Principles for Java

Decision criteria for design and review. Functional techniques are in `java-functional-programming.instructions.md`. Architecture is in `java-functional-oo-integration.instructions.md`.

## Priority and Conflict Rule

Principles are listed in order of priority. When two conflict, the higher-ranked one wins.

- Apply a principle only as far as a concrete, current need exists.
- Do not introduce an abstraction to satisfy a principle when the code is already simple and has one use case.

## 1. SOLID

- **S**: Give each class and function one reason to change. Separate orchestration (service) from rules (pure function).
- **O**: Add variants through new `sealed` subtypes, functions or implementations. Do not edit stable code to add a case that a new type can handle.
- **L**: Make subtypes honor the parent contract. Do not strengthen preconditions or throw unexpected exceptions.
- **I**: Keep interfaces small and role-focused.
- **D**: Depend on abstractions. Inject dependencies through the constructor.

## 2. Clean Code

- Use intention-revealing names. Avoid abbreviations and noise words (`Manager`, `Helper`, `Data`).
- Keep each method at a single level of abstraction and short enough to read without scrolling.
- Group long parameter lists into a `record`.
- Replace magic numbers and strings with named constants or enums.
- Write comments to explain *why*. Express *what* through code.

## 3. DRY

- Extract duplicated business knowledge (rules, formulas) into one named function or method.
- Do not merge code that is only coincidentally similar. Wait for the third occurrence before abstracting.

## 4. KISS

- Choose the simplest solution that satisfies the requirement.
- Prefer a plain method over a pattern, a lambda over a class, a `record` over a builder.
- Split nested streams, long `Optional` chains and deep `switch` expressions into named steps.

## 5. Separation of Concerns (SoC)

- Keep I/O, business rules and presentation in separate layers.
- Do not mix rules with persistence, HTTP or serialization in the same method.
- Map between layers at the boundary (entity ↔ domain model ↔ DTO).

## 6. Program to Interfaces

- Declare variables, parameters and return types with the interface (`List`, `Map`, `Set`), not the implementation (`ArrayList`, `HashMap`).
- Define a port (interface) for each external dependency. Keep implementations in the infrastructure layer.

### Good

```java
public class ReportService {
    private final ReportRepository repository;   // interface

    public ReportService(ReportRepository repository) {
        this.repository = repository;
    }
}
```

### Bad

```java
public class ReportService {
    private final JpaReportRepositoryImpl repository = new JpaReportRepositoryImpl();
}
```

## 7. Low Coupling and High Cohesion

- Keep in one class or package only what changes together.
- Pass a function only the data it needs, not the whole aggregate.
- Avoid shared mutable state and static mutable fields.

## 8. Composition over Inheritance

- Combine behavior with injected collaborators and functions.
- Use inheritance only for stable "is-a" relationships. Prefer `sealed` hierarchies over open class trees.
- Declare classes `final` unless designed for extension.

## 9. Encapsulation

- Declare fields `private final`. Expose behavior, not state.
- Return unmodifiable copies of collections (`List.copyOf`).
- Use the narrowest visibility. Make a type `public` only when another package needs it.

## 10. YAGNI

- Implement only what the current requirement needs.
- Do not add single-implementation interfaces, extension hooks or configuration flags for hypothetical futures. Exception: ports to infrastructure (principle 6).
- Introduce an abstraction when a second real use case appears.

## Validation

- Review a change top-down through the principles. Stop at the first violation and fix it before checking lower-ranked ones.
- Build and tests: `mvn verify` or `./gradlew build`.
