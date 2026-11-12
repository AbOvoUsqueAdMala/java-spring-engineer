# Testing and delivery

Choose evidence for the changed behavior, not for an arbitrary number of tests or coverage percentage. Reuse the repository's test stack and mandatory checks.

## Match the test to the failure boundary

| Changed behavior | Useful evidence |
|---|---|
| Pure domain logic | Fast unit tests for invariants, boundary values, and failures; no Spring context |
| HTTP validation, JSON, routing, permissions | Web slice with actual serialization and security rules; permitted and denied requests |
| SQL, mapping, constraints, locking | Persistence slice/integration test against the relevant DB engine, often Testcontainers |
| Transaction rollback, proxy advice, events | Integration through the actual bean boundary; inspect post-commit state and delivery behavior |
| Remote dependency policy | Stub server or controlled fake producing timeouts, transient/permanent errors, cancellation, and duplicate attempts |
| Module dependencies | Existing ArchUnit/Modulith verification where architecture is part of the change |
| Application wiring | Focused context or a startup/request smoke test when wiring changed |
| Performance claim | Reproducible before/after workload and relevant latency/throughput/allocation/query measurements |

Do not mock JPA or an HTTP client and then claim real query behavior, transaction safety, or transport timeout correctness. Avoid starting a complete application for every domain unit test. Select test APIs available in the project's version; do not copy a newer mock annotation/import into an older branch, or propagate a deprecated example into a migration.

Transactional tests can hide defects: test-managed rollback, an open persistence context, and unflushed SQL may mask missing production boundaries, constraints, or lazy-loading failures. Flush and cross a real commit boundary when necessary to prove the property. Database failure assertions may need a fresh transaction after a constraint violation.

For races, coordinate workers with latches/barriers or database state rather than arbitrary sleeps. For time-dependent logic use a controllable clock; for async assertions use bounded polling already supported by the project. Check side effects and final state, not just mocked invocation counts.

## Run what CI actually runs

Read build/CI configuration before choosing commands. Examples, only when the corresponding tasks exist:

| Environment | Maven | Gradle |
|---|---|---|
| POSIX | `./mvnw verify` | `./gradlew check` |
| PowerShell | `.\mvnw.cmd verify` | `.\gradlew.bat check` |

Start with relevant tests. Maven `test` may omit Failsafe integration tests; Gradle `check` may omit custom integration tasks if they are not wired into it. Inspect profiles, tags, disabled tests, reports, and container availability. A database service may be provided by CI rather than Testcontainers; preserve an equivalent established setup.

If a required runtime/service is unavailable, report precisely what ran and what remains unverified. Do not silently substitute H2 for a PostgreSQL migration test, suppress failing checks, or describe a skipped suite as passed. Once appropriate checks pass, repeat them only after meaningful changes or new evidence.

## Deliver proportionately

Review the diff for unintended API/configuration/schema changes. Keep documentation updates tied to changed behavior. For requested release/deployment work, include migration compatibility, a recovery/rollback approach, observability, and graceful lifecycle behavior relevant to the service. A code implementation alone does not establish production capacity or justify a live deployment.

Report concrete changes and executed checks with their results. Respect existing coverage/static-analysis gates; do not invent a universal 85%/100% target, force new build plugins, or add tests that only restate generated code.

Sources: [Boot Testcontainers integration](https://docs.spring.io/spring-boot/reference/testing/testcontainers.html), [Spring Modulith verification](https://docs.spring.io/spring-modulith/reference/verification.html), [Spring transaction behavior](https://docs.spring.io/spring-framework/reference/data-access/transaction/declarative/annotations.html).
