---
description: 'Prioritized design checks for Java: SOLID, Clean Code, postponing generalization (YAGNI), DRY, KISS, SoC, interfaces, cohesion and coupling, composition, encapsulation'
applyTo: '**/*.java'
---

# Design Principles for Java

Concrete checks for design and review, not slogans. Functional techniques are in `java-functional-programming.instructions.md`. Architecture is in `java-functional-oo-integration.instructions.md`.

Project conventions take precedence over these generic guidelines. When they conflict, follow the project and say so.

## Priority and Conflict Rule

Principles are listed in order of priority. When two conflict, the higher-ranked one wins.

- Apply a principle only as far as a concrete, current need exists.
- Principles 1 and 4 to 7 can push toward abstraction. Principle 3 limits them.

## 1. SOLID

- **S**: Give each class one reason to change. Separate orchestration from rules.
- **O**: Add variants through new `sealed` subtypes, functions or implementations once a real variation exists.
- **L**: Subtypes honor the parent contract. Do not strengthen preconditions.
- **I**: Keep interfaces small and role-focused.
- **D**: Depend on abstractions at boundaries. Inject dependencies through the constructor.

## 2. Clean Code

- Use intention-revealing names. Avoid abbreviations and noise words (`Manager`, `Helper`, `Data`).
- Keep each method at a single level of abstraction.
- Group long parameter lists into a `record`.
- Replace magic numbers and strings with named constants or enums.
- Write comments to explain *why*. Express *what* through code.

## 3. Postpone Generalization (YAGNI)

- Implement only what the current requirement needs.
- Do not extract an interface, a generic type or a reusable component before the second real use. Early extraction fixes parameters and return types around one use case.
- Duplicate once, then extract using what the duplication shows. Extract when implementing the next feature that needs the same behavior.
- Do not add extension hooks or configuration flags for hypothetical futures.
- Exception: ports to infrastructure (repository, gateway, clock).

## 4. DRY

- Extract duplicated business knowledge (rules, formulas) into one named function.
- Apply after principle 3: do not merge code that is only similar. Extract after the second occurrence is understood.

## 5. KISS

- Choose the simplest solution that satisfies the requirement.
- Prefer a plain method over a pattern, a lambda over a class, a `record` over a builder.
- Split nested streams, long `Optional` chains and deep `switch` expressions into named steps.
- Add a layer or class only when it earns its place. A controller or application service can be the use case itself.

## 6. Separation of Concerns (SoC)

- Keep I/O, business rules and presentation in separate layers.
- Do not mix rules with persistence, HTTP or serialization in the same method.
- Map between layers at the boundary (entity ↔ domain model ↔ DTO).

## 7. Program to Interfaces

- Declare variables, parameters and return types with the interface (`List`, `Map`, `Set`), not the implementation (`ArrayList`, `HashMap`).
- Define a port for each external dependency. Keep implementations in the infrastructure layer.
- For internal collaborators, apply principle 3 before extracting an interface.

## 8. High Cohesion and Low Coupling

Cohesion here means that what belongs together stays together. It is not only "methods that use the same fields".

- Check that each file in a module is about the same subject as the others. Code that is not about the module's subject moves out (for example, a markdown parser inside a course-registration service becomes its own service).
- Put behavior next to the data it uses. If a class has a date field and callers compare it with other dates, add the method to that class.
- Make a decision as close as possible to the origin of the data.
- A new service contains only code that is connected to the others in it.
- Pass a function only the data it needs, not the whole aggregate.
- Avoid shared mutable state and static mutable fields.

## 9. Composition over Inheritance

- Combine behavior with injected collaborators and functions.
- Use inheritance only for stable "is-a" relationships. Prefer `sealed` hierarchies over open class trees.
- Declare classes `final` unless designed for extension.

## 10. Encapsulation

- Declare fields `private final`. Expose behavior, not state.
- Return unmodifiable copies of collections (`List.copyOf`).
- Use the narrowest visibility. Make a type `public` only when another package needs it.

## Validation

- Review a change top-down through the principles. Stop at the first violation and fix it before checking lower-ranked ones.
- Build and tests: `mvn verify` or `./gradlew build`.
