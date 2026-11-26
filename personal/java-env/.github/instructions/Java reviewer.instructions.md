---
description: 'Java code review checklist for pull requests: blocking issues first, then design, tests and style; output format for review comments'
applyTo: '**/*.java'
excludeAgent: 'cloud-agent'
---

# Java Code Review

Review the diff, not the whole file. Report only what a linter, the compiler or CI does not already catch. Order comments by severity and give the file, the rule and a concrete fix.

## Blocking (request changes)

- Secrets, tokens or personal data in code, tests or logs.
- SQL or JPQL built by string concatenation; unvalidated input reaching persistence or the filesystem.
- Money as `double`/`float`; time read from `Instant.now()` inside rules.
- Domain code importing infrastructure (repository, `Clock`, web framework, JPA entity).
- Business rules mixed with I/O in one method; rules that mutate their arguments.
- A `catch` that logs at `ERROR` and rethrows; swallowed exceptions; `null` returned as an error signal.
- `@Transactional` on a controller or a domain function.
- New dependency or edited applied migration without a stated reason.
- Changed behavior with no test that fails without the change; bug fix without a regression test.

## Important (fix before merge)

- Method with Cognitive Complexity above 15, file above 200 lines, or logic added to a method already near its limit.
- Branching on variants with `if / else if` where a `sealed` type + `switch` fits; `default` on a `switch` over a sealed type.
- Mutable state shared across calls; setters on new classes; `Optional` in fields or parameters.
- Abstraction with a single use (interface, generic type, hook, flag): apply YAGNI.
- Duplicated business rule in two places: one named function.
- Class mixing unrelated responsibilities; behavior placed away from the data it uses.
- External call without `INFO` entries before and after; non-parameterized log messages.
- Public API returning mutable collections or implementation types (`ArrayList`).

## Tests

- Assertions that check nothing (`assertNotNull` alone, `assertTrue(a == b)`, no assertion).
- Tests tied to implementation: private methods, call order, `verifyNoMoreInteractions` without a reason, verified queries.
- Mocks of records, DTOs, entities, collections or the class under test; more than three mocks in one test.
- `Thread.sleep`, `Instant.now()`, unseeded random data, shared mutable state, `@Disabled` without an issue.
- Conditionals or loops inside test methods; one test covering several unrelated behaviors.
- Missing boundary cases for new comparisons; missing test per branch of a new `switch`.

## Suggestions (mention briefly)

- Names that hide intent (`Manager`, `Helper`, `Data`); magic numbers; comments that restate the code.
- A lambda longer than a few lines that deserves a named method.
- Long parameter lists that belong in a `record`.

## Comment format

`[blocking|important|suggestion] File:line — rule violated. Fix: <one concrete change>.`
Close with one line: approve, approve with suggestions, or request changes, plus the count per severity.
