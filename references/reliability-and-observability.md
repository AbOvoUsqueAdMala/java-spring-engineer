# Reliability and observability

Read for remote integrations, messaging, async work, caches, overload, or production behavior. Add mechanisms only for relevant failure modes.

## Remote calls

For each dependency, define a caller deadline, connect/request/read timeout behavior supported by the chosen client, concurrency budget, retry ownership, and failure contract. Include connection-pool acquisition, DNS/TLS behavior, queueing, and retries in the end-to-end budget; a timeout wrapper may not actually cancel the underlying work.

- Retry only identified transient failures and only when the operation is safe to repeat. A timeout means the outcome may be unknown, not that the server did nothing. Respect provider retry semantics and `Retry-After` when applicable.
- Bound attempts and total elapsed time; use backoff with jitter. Choose one retry owner along a call path, accounting for SDK, HTTP client, service, and broker retries. Nested policies multiply load.
- Do not retry all exceptions, validation errors, or authorization failures. Preserve cancellation/interruption; do not use blocking sleep on reactive event loops.
- Separate circuit breakers and bulkheads by meaningful dependency/failure domain. A circuit breaker does not itself limit concurrent work; a rate limit does not alone bound in-flight requests when latency grows.
- Use an existing compatible library/integration such as Resilience4j when appropriate. Inspect aspect/decorator order: placing retries inside versus outside the breaker or transaction changes measured failures and side effects. Do not use arbitrary thresholds copied from a tutorial.
- A fallback must have a valid business meaning. Do not report a successful payment/order when the outcome is unknown; return or persist an explicit pending/failed state consistent with the contract.

## Idempotency and delivery

For repeated commands that can create duplicate effects, define a stable idempotency key, scope (tenant/actor and operation), payload identity, retention window, and concurrency behavior. Persist the key/result atomically with the local state transition using a database uniqueness constraint or equivalent durable mechanism. Reusing a key with a different payload must not silently succeed. An in-memory map is insufficient across restarts or replicas.

If a database update and event publication must survive crashes together, use an existing transactional outbox or a compatible durable publication mechanism. Write the state and publication record in the same local transaction. A plain `AFTER_COMMIT` listener leaves a crash gap. The relay can publish duplicates: consumers need deduplication/idempotent state transitions and deliberate acknowledgment timing, retry/dead-letter handling, and replay behavior.

Spring Modulith's event publication registry is an option for its documented integration boundaries; inspect the configured persistence/externalization and recovery semantics. Do not equate it with universal cross-system exactly-once delivery. Broker transactions alone do not atomically cover arbitrary database writes and external APIs. Use a state machine/saga and compensation only where the business workflow requires them.

## Overload, caches, and async work

Bound queues, worker concurrency, client/DB pools, and fan-out. Reject or shed work before memory and latency become unbounded; choose HTTP/gRPC failure semantics consistent with the service contract. Netflix's adaptive concurrency work motivates observing in-flight demand and latency; adding its specific library requires a justified workload and compatible integration.

For caching, define ownership of invalidation, TTL, capacity, staleness, and tenant-sensitive keys. Avoid unbounded maps keyed by user IDs/IPs. Consider stampedes and cache failure behavior. A local cache or limiter has per-instance semantics; do not present it as a fleet-wide guarantee.

For async/scheduled work, define whether loss on process exit is acceptable, how failures are surfaced, and what prevents unwanted duplicate work across replicas. Use durable jobs/messaging only when that durability is required. Propagate trace/security context intentionally without leaking one request's state into another.

## Observe and recover

Reuse the existing Micrometer/OpenTelemetry/logging stack. Capture request rate, errors, latency distribution, dependency latency, saturation, pool waits, rejected work, and retry counts as relevant. Use bounded metric labels such as route templates and dependency names; do not label metrics with arbitrary user/order IDs or raw URLs. Keep credentials and sensitive payloads out of logs/traces.

Liveness should describe whether the process can recover without a restart, not fail because a shared downstream service is unavailable. Readiness reflects ability to serve; select shared dependency checks deliberately to avoid removing every replica together. Protect Actuator access and details. On requested lifecycle work, coordinate graceful shutdown, traffic draining, in-flight jobs, and termination deadlines. Validate SLOs/load behavior in an appropriate test environment before making capacity claims.

Sources: [Amazon retry guidance](https://aws.amazon.com/builders-library/timeouts-retries-and-backoff-with-jitter/), [Netflix concurrency limits](https://github.com/Netflix/concurrency-limits/blob/main/README.md), [Resilience4j](https://github.com/resilience4j/resilience4j/blob/master/README.adoc), [Modulith events](https://docs.spring.io/spring-modulith/reference/events.html), [Actuator endpoints and probes](https://docs.spring.io/spring-boot/reference/actuator/endpoints.html). Hystrix is in [maintenance mode](https://github.com/Netflix/Hystrix/blob/master/README.md); do not introduce it into a new service as a current default.
