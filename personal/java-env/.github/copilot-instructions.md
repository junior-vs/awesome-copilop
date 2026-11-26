# Repository Instructions (Java)

> Fill the `<...>` placeholders for this project and delete what does not apply.

## Stack

- Java 21, `<Spring Boot version>`, build with `<Maven | Gradle>`.
- Tests: JUnit 5, Mockito, `<AssertJ | JUnit assertions>`, Testcontainers for adapters.
- Packages: `<root.package>` with `domain` (pure rules), `application` (use cases), `infrastructure` (adapters, JPA), `web` (controllers).

## Commands (non-interactive only)

| Task | Maven | Gradle |
| --- | --- | --- |
| Unit tests | `mvn -B test` | `./gradlew test --console=plain` |
| Full build + integration tests | `mvn -B verify` | `./gradlew check --console=plain` |
| Format | `mvn -B spotless:apply` | `./gradlew spotlessApply` |

- Run the unit tests of the touched module before declaring a task done; run the full build before opening a pull request.
- Leave long-running processes (`spring-boot:run`, `bootRun`) to the user.

## Boundaries

- Keep `.env*`, `application-*.yml` with credentials, `.git/` and generated sources untouched.
- Ask before adding a dependency, deleting a file or editing applied database migrations.
- Keep changes surgical: touch only what the task needs.

## Precedence

Project conventions found in the code override the generic rules below; say so when they differ.

## Rules by topic (loaded automatically by file pattern)

Design and production code (`**/*.java`):

- [Design principles](instructions/java-design.instructions.md)
- [Functional style](instructions/java-functional.instructions.md)
- [Functional core / OO shell](instructions/java-architecture.instructions.md)
- [Complexity limits (Cognitive Complexity, CDD)](instructions/java-complexity.instructions.md)
- [Logging](instructions/java-logging.instructions.md)
- [Security, types, transactions, dependencies](instructions/java-conventions.instructions.md)

Tests (`*Test.java`, `*IT.java`, `*Steps.java`):

- [Test design](instructions/java-test-design.instructions.md)
- [JUnit 5 assertions](instructions/java-test-assertions.instructions.md)
- [Mockito and test doubles](instructions/java-test-mocks.instructions.md)

## Agent

- [`java-engineer`](agents/java-engineer.agent.md): end-to-end Java work (plan, implement, test, measure complexity, verify).
