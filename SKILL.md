---
name: java-spring-engineer
description: "Implement, debug, refactor, and review Java backend and Spring Boot applications with attention to transactions, persistence, API contracts, security, concurrency, and service reliability. Use for Java/Spring backend features, defects, performance investigations, tests, and requested migrations; not for Android, frontend-only work, or unrelated infrastructure administration."
---

# Java / Spring Engineer

Deliver the requested change using the engineering standards below, within the project's compatibility constraints and the user's scope. Apply the public engineering lessons of Spring, Netflix, Google, Amazon, and Zalando according to the actual workload; these are not claims of company endorsement or a mandate to reproduce their platforms.

## Required engineering standards

- **Hibernate:** use Hibernate through JPA, normally with Spring Data JPA, for relational application persistence. Use direct SQL only for a performance-critical operation with a demonstrated need for maximum optimization or throughput/latency requirements that the tuned ORM path cannot meet. Measure and isolate the exception; ordinary CRUD stays with Hibernate. JPQL/HQL and Criteria are ORM queries, not direct SQL. See the persistence reference for the decision and verification rules.
- **Java style with tab indentation:** use actual tab characters (`U+0009`) for Java indentation, with one tab per block nesting level; do not emulate indentation with spaces. Use the [Google Java Style Guide](https://google.github.io/styleguide/javaguide.html) for other compatible formatting, import, naming, and documentation rules. The user's tab requirement overrides Google's whitespace and indentation rules in authored code and examples; this is a customized style, not strict Google Java Style.
- **Meaningful full names:** never shorten names merely to save typing. Variables, parameters, classes, and methods must express their domain meaning or responsibility. Use `customerRepository`, `paymentRequest`, and `calculateInvoiceTotal`, not `repo`, `req`, or `calc`. This naming requirement is stricter than Google's examples; preserve externally fixed names and compatibility-sensitive contracts.
- **Clean Code and Clean Architecture:** use Robert C. Martin's principles as the foundation throughout the application: focused responsibilities, readable behavior, explicit invariants, and dependencies directed toward business rules. Separate domain policy, application use cases, and external adapters without adding empty layers or an interface for every class.
- **Methods fit one screen:** keep handwritten methods within one standard editor screen at normal readable zoom. When no project screen-height convention is known, use 40 source lines from the method signature through its closing brace, including blank lines and comments, as this skill's working default; honor a stricter project limit. Split longer methods by responsibility into clearly named helper methods, which must satisfy the same limit. See the code-quality reference for extraction safeguards.
- **Lombok:** use Lombok to eliminate as much mechanical boilerplate as safely possible, including constructor injection, accessors, logging, and suitable builders. Configure compatible annotation processing. Keep validation and domain behavior explicit, and follow the entity safeguards in the code-quality reference.
- **MVC:** use Spring MVC for web applications, with thin controllers, application services/use cases, and a separate response/view representation. Keep business rules and persistence implementation out of controllers. REST responses may be JSON; MVC does not require HTML templates.
- **Package by layer:** organize Java packages by technical role: controllers in `controller`, application services/use cases in `service`, repository interfaces and implementations in `repository` or its business subpackages, DTOs in `model.dto`, and persistence entities in `model.entity`. Repository interfaces include Spring Data interfaces and domain-owned repository contracts; each belongs in a `repository` package within its owning module or architectural boundary. Preserve module boundaries and keep DTOs and entities as separate types. This is the user's required package convention, not a Java or Spring requirement. See the architecture reference for layout and scoped adoption.
- **Business subpackages:** within layer packages, group classes into subpackages by business area or use case whenever a coherent grouping exists. For example, use `controller.orders`, `service.orders`, `repository.orders`, `model.dto.orders`, and `model.entity.orders`. Use consistent business names across layers. Keep small cohesive packages simple and avoid empty groups or redundant nesting when the enclosing module already expresses the business area.

Apply these standards to new applications and code created or changed within the task. Existing conventions are not a reason to silently ignore them. If an established execution model, enforced build rule, or compatibility contract conflicts, identify the concrete conflict and contain the exception. A localized change does not authorize a wholesale stack migration, mass renaming, or unrelated reformatting.

## Establish the relevant baseline

Inspect repository instructions, build files and wrappers, toolchain/CI configuration, and one nearby implementation with its tests. Identify Java and Spring versions, module boundaries and package structure, MVC versus WebFlux, persistence technology, authentication model, migration mechanism, and the commands that actually execute the relevant tests. Inspect only what matters to the task.

Preserve compatible working conventions, public contracts, dependency management, and the user's scope while applying the required standards. A feature request does not imply a framework upgrade, a wholesale architecture migration, a deployment, or an infrastructure purchase. For new projects, choose a compatible supported baseline from official documentation; do not hardcode a moving "latest" version. Explain material choices briefly and proceed when reasonable defaults suffice.

## Work from behavior to evidence

- **Feature or defect:** identify the observable behavior and failure boundary; implement the smallest coherent change. Reproduce a defect and add a regression check when useful.
- **Review:** trace concrete execution paths and report actionable findings with location, trigger, impact, and a practical fix. Distinguish violations of explicit user/project standards from optional style preferences; do not turn optional preferences into defects or edit code unless requested.
- **Performance:** measure the bottleneck and workload first. Distinguish CPU, allocation/GC, database, downstream latency, and queue saturation; do not promise gains from an unmeasured rewrite.
- **Migration:** establish source and target compatibility, read the matching migration notes, and verify changed integrations. Do not mix migration work into an unrelated fix.

## Decision rules

1. Prefer a modular monolith for a new small service unless independent deployment, ownership, isolation, or scale justifies distribution. Preserve existing service boundaries. Apply Clean Architecture's dependency direction with proportionate boundaries; DDD, CQRS, Kafka, discovery servers, and GraphQL require identified needs.
2. Keep business invariants and multi-step transaction ownership explicit. Prefer constructor injection and focused components; avoid introducing an interface for every class or forwarding layers with no responsibility.
3. Use MVC with Hibernate for new blocking web applications. For a localized change to an existing reactive service, explain the compatibility exception and preserve its execution model; never put blocking Hibernate calls on an event loop. Reactive request paths need nonblocking dependencies or explicitly bounded isolation. Virtual threads do not increase database capacity or provide backpressure.
4. Model public API payloads separately from persistence internals where needed to control exposure and compatibility. Bound collection queries and request sizes. Preserve existing error contracts; use Spring ProblemDetail for a suitable new REST contract on a compatible version.
5. Treat remote effects, local transactions, and message delivery as separate failure domains. An annotation, retry, or broker guarantee does not make all of them atomic. Read the reliability reference before changing these interactions.
6. Use Boot/platform BOMs and repository toolchains. Do not replace managed versions, add dependencies, or enable preview features merely to imitate an example.

## Load only the relevant references

| Work involves | Read |
|---|---|
| Writing or changing Java code, names, formatting, Clean Code, Lombok | [Code style and maintainability](references/code-style-and-maintainability.md) |
| Project creation, package organization, module boundaries, Java execution model, version migration | [Architecture and compatibility](references/architecture-and-compatibility.md) |
| Hibernate/JPA, direct SQL exceptions, transactions, schema changes, concurrent writes | [Persistence and transactions](references/persistence-and-transactions.md) |
| Remote calls, retries, queues, events, caches, overload, service operations | [Reliability and observability](references/reliability-and-observability.md) |
| REST/GraphQL contracts, authorization, Spring Security | [API and security](references/api-and-security.md) |
| Test selection, regression evidence, performance verification, release checks | [Testing and delivery](references/testing-and-delivery.md) |
| Why these rules differ from public skills; provenance or refreshing guidance | [Sources and comparison, in Russian](references/sources-and-decisions.md) |

For plain Java work, use the Java-specific portions; do not add Spring. Follow the user's conversation language and the repository's code/documentation language.

## Completion

Run the focused checks and the repository's required verification for the changed behavior. Confirm which integration tests actually ran; a successful compile or skipped suite is not a passing integration test. Report the result, important design decisions, executed checks, and remaining material limitations. Preserve established quality gates without inventing universal coverage percentages. Prefer explicit uncertainty over an unsupported claim of production readiness.
