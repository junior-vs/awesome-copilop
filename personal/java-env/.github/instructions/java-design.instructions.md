---
description: 'Java design checks in priority order: SOLID, Clean Code, YAGNI, DRY, KISS, separation of concerns, interfaces, cohesion, composition, encapsulation'
applyTo: '**/*.java'
---

# Java Design

Apply a principle only as far as a concrete, current need exists. Principles are ranked 1 to 10 and the lower number wins a conflict. Exception: YAGNI (3) limits the abstraction pushed by 1, 4, 5, 6 and 7.

## 1. SOLID
- S: give each class one reason to change; keep orchestration apart from rules.
- O: add variants as new `sealed` subtypes, functions or implementations once a real variation exists.
- L: subtypes honor the parent contract and never strengthen preconditions.
- I: keep interfaces small and role-focused.
- D: depend on abstractions at boundaries; inject through the constructor.

## 2. Clean Code
- Name by intent, using domain words instead of `Manager`, `Helper`, `Data`.
- Keep one level of abstraction per method.
- Group long parameter lists into a `record`.
- Use named constants and enums for magic values.
- Comment the *why*; let code express the *what*.

## 3. YAGNI
- Implement only the current requirement.
- Extract an interface, generic type or reusable component at the second real use, shaped by what both uses show.
- Add extension points and flags when a requirement asks for them.
- Exception: ports to infrastructure (repository, gateway, clock).

## 4. DRY
- Extract duplicated business knowledge (rules, formulas) into one named function once the second occurrence is understood. Code that is only similar stays separate.

## 5. KISS
- Choose the simplest solution: a method over a pattern, a lambda over a class, a `record` over a builder.
- Split nested streams, long `Optional` chains and deep `switch` expressions into named steps.
- Add a layer or class only when it earns its place; a controller or application service can be the use case itself.

## 6. Separation of Concerns
- Keep I/O, business rules and presentation in separate layers; map at the boundary (entity ↔ domain model ↔ DTO).
- Keep persistence, HTTP and serialization out of rule methods.

## 7. Program to Interfaces
- Declare variables, parameters and returns as `List`, `Map`, `Set`.
- Define one port per external dependency, implemented in the infrastructure layer.
- For internal collaborators, apply YAGNI first.

## 8. Cohesion and Coupling
- Keep every file in a module on the module's subject; move unrelated code to its own service (a markdown parser leaves a course-registration service).
- Put behavior next to the data it uses (a class with a date field owns the date comparisons).
- Decide as close as possible to the origin of the data.
- Pass a function only the data it needs, not the whole aggregate.
- Share immutable state only.

## 9. Composition over Inheritance
- Combine behavior with injected collaborators and functions.
- Reserve inheritance for stable "is-a" relations; prefer `sealed` hierarchies; declare classes `final` unless designed for extension.

## 10. Encapsulation
- Declare fields `private final`; expose behavior, not state.
- Return unmodifiable collections (`List.copyOf`).
- Use the narrowest visibility.
