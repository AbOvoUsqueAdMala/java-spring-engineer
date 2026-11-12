# Persistence and transactions

Read when changing stored state, queries, transactions, or schema evolution. Use Hibernate through JPA for relational application persistence, normally with Spring Data JPA. For a localized change in an incompatible existing stack, explain the compatibility exception instead of silently migrating the whole persistence layer.

## Hibernate first; direct SQL for demonstrated performance needs

- Use entity operations, Spring Data derived queries, JPQL/HQL, Criteria, and projections for routine application access. A JPQL `@Query` remains an ORM query; `nativeQuery = true`, native `EntityManager` queries, and handwritten JDBC/jOOQ SQL use the direct SQL exception.
- Before direct SQL, establish the latency/throughput target, representative data size, and measured bottleneck. Inspect generated SQL, plans, query counts, indexes, fetch strategies, projections, batching, and pagination. A preference for handwritten SQL or an assumption that it is faster is insufficient.
- Use direct SQL only for the specific critical operation when measurements show that a tuned Hibernate path cannot meet the required performance or that SQL is necessary for the required optimization. Keep it inside a focused persistence adapter; document why ORM is insufficient and verify the improvement under the same workload. If measurement is unavailable, state the limitation and do not claim a demonstrated benefit.
- Preserve transaction participation, parameter binding, tenant/access restrictions, and result mapping. Native SQL may bypass ORM filters. For bulk/native writes or mixed JDBC and JPA, handle pending ORM writes, stale managed state, and relevant caches deliberately; do not assume entity callbacks or optimistic version checks run automatically. Verify these details against the project's Hibernate version.
- This restriction concerns runtime application data access. SQL migrations, schema/index DDL, and diagnostic `EXPLAIN` queries remain appropriate tooling; do not route them through entity operations.

The performance-only restriction is the user's engineering policy. Hibernate supports HQL, Criteria, and native SQL; its documentation does not impose this restriction. See [query approaches and native SQL, sections 6 and 6.6](https://docs.hibernate.org/orm/7.1/introduction/html_single/#queries); use the documentation matching the project's version rather than adopting this linked version automatically.

## Define atomicity before annotations

Identify the invariant, participating rows, transaction manager, entry point, isolation needs, and what must happen on failure. Place a local transaction around the business operation that must commit atomically, generally a service/use-case boundary.

- With normal Spring proxy-based transaction handling, a call from a method to another method on the same instance bypasses transaction advice. Use an appropriate externally invoked boundary, a separate component, or `TransactionTemplate`; adding another annotation does not repair self-invocation.
- Check effective rollback rules, including configured global policy. Default unchecked exceptions/errors trigger rollback; checked exceptions need an applicable rollback rule. Catching and suppressing an exception can commit unintended partial work. `readOnly` is a hint/optimization, not a universal write prohibition.
- `REQUIRES_NEW` creates an independent transaction and may need an additional connection. It is not a general fix for partial writes or exhausted pools. Explain why an independent commit is desired.
- Async tasks do not automatically inherit the caller's transaction. Imperative thread-bound transactions and reactive transaction context are different models.
- Keep slow remote calls out of database transactions where feasible. Never claim that rolling back a local DB transaction undoes a completed HTTP call. See [reliability](reliability-and-observability.md) for multi-system effects.

## Queries and mappings

- Choose a fetch plan for the actual use case: DTO projection, entity graph, explicit join fetch, batching, or a dedicated query. Measure query count and execution plan when N+1 or latency is at issue. Changing every relationship to eager loading is not a solution.
- Be careful with collection fetch joins and pagination: they may multiply rows or cause in-memory pagination. Use an appropriate projection, batch strategy, or page IDs then fetch details; keep count and stable ordering correct.
- Bound page sizes, allowlist sort fields where exposed, and use a deterministic tie-breaker. Consider keyset pagination for deep, frequently changing datasets when the contract permits it.
- Map lazy state while the intended persistence context is available. Do not enable Open Session in View merely to hide missing fetches. For a new service API, prefer explicit fetches and DTO assembly within the use-case boundary; assess compatibility before changing an existing OSIV setting.
- Parameterize SQL values. Dynamic identifiers/order expressions need validated construction rather than string concatenation of client input. Verify indexes against real predicates and plans; do not add an index to every column.

## Concurrency and migrations

- Enforce uniqueness and referential invariants in the database. An application-level existence check followed by insert is racy; handle the database conflict deliberately.
- For read-modify-write races, use an appropriate optimistic version check, atomic conditional update, or justified pessimistic lock. Specify conflict behavior. Avoid blindly retrying an entire operation that already performed an external effect.
- Evolve production schemas through the existing Flyway/Liquibase process. Do not edit already applied versioned migrations or use automatic destructive schema generation in production.
- For deployments where versions overlap, use additive/expand-contract changes with a compatible backfill and deferred removal. Account for locks, large-table operations, and recovery. Transactional DDL behavior is database-specific.

## Prove the property

Use the production database engine/version family for SQL, locking, migration, and constraint-sensitive tests, commonly with Testcontainers. Exercise rollback through the Spring proxy and inspect committed state in a separate transaction. Test the unique-conflict or concurrent-update path with deterministic coordination. An H2-only test or mocked repository cannot establish PostgreSQL/MySQL transaction semantics.

Sources: [Spring transaction semantics](https://docs.spring.io/spring-framework/reference/data-access/transaction/declarative/annotations.html), [Boot Testcontainers](https://docs.spring.io/spring-boot/reference/testing/testcontainers.html). Query and migration choices above are engineering adaptations; validate against the actual ORM and database documentation when implementing them.
