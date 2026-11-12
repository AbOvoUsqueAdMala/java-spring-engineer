# Architecture and compatibility

Read for setup, structural changes, concurrency choices, or upgrades.

## Respect the project's baseline

- Resolve effective versions from parent POM/BOM, Gradle plugins/platforms/catalogs, toolchain, wrapper, and CI. A declared Java language level, the JDK running the build, and the production runtime can differ. Reconcile discrepancies before using new APIs.
- Use documentation for the installed version. Boot major versions can change starter/module names, test imports, Jackson integration, Jakarta APIs, Security DSLs, and supported build tools. Check Boot, Spring Cloud, Spring Modulith, DGS, and other integration compatibility separately when used.
- For a new project, select a stable supported Boot release and a supported compatible JDK according to deployment constraints. Explain the choice; avoid milestone/snapshot releases without a reason. For an existing legacy service, contain the requested fix and identify unsupported components separately.
- Keep BOM alignment. A justified dependency override needs a reason and compatibility verification; a security fix is not an excuse to upgrade the entire stack blindly. Prefer existing Maven/Gradle wrappers to a global build-tool installation.

## Clean Architecture and MVC

Use Robert C. Martin's Clean Architecture as the application design foundation. Dependencies point toward domain policy and application use cases. Define infrastructure-facing interfaces at the inner boundary and implement them in outer adapters. Keep HTTP, ORM, and database-specific types outside the business core; wire adapters and transaction handling through Spring at the application boundary. Avoid creating layers that have no responsibility. [Primary explanation](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html).

Use Spring MVC at the web boundary. Controllers translate and validate transport input and call application use cases; services coordinate business operations and their transaction boundaries. Persistence adapters implement storage access with Hibernate. Response objects or templates represent output; do not expose managed entities as the public API. MVC organizes presentation, while Clean Architecture controls dependencies across the application. [Spring MVC documentation](https://docs.spring.io/spring-framework/reference/web/webmvc.html).

Organize substantial applications around business capabilities, with small exposed module APIs and hidden implementation. Within each business module, use the package-by-layer convention below. Packages can express the boundaries inside one deployable application; separate build modules are optional. Keep business rules independent of web and persistence frameworks and exercise them without an application server or database. For a local edit, improve the touched boundary without rebuilding the entire package structure.

For a modular architecture, make cycles and access to internal packages detectable with existing architecture tests. Spring Modulith can verify these constraints through `ApplicationModules.of(Application.class).verify()`; use it when module verification is useful and dependency compatibility is established. Do not retrofit it into every bug fix.

Microservices require a concrete reason: separate deployment/ownership, fault containment, or independently different scaling. Account for network failures, data ownership, contract evolution, and operating cost. Do not infer that Netflix's scale makes microservices appropriate for the current application.

## Package organization: package by layer

Use this package structure under the application's base package, or under each business module's base package when the application is modular:

```text
com.example.application
|-- controller       HTTP controllers
|-- service          Application services and use cases
|-- repository       Repository interfaces and persistence implementations
`-- model
    |-- dto          Request, response, and other transfer objects
    `-- entity       Persistence entities, including JPA entities
```

This layout is the user's required convention for work using this skill; Java and Spring do not mandate these package names. Java package declarations and source directories must agree. Create packages when they have real classes, not as empty scaffolding.

Within layer packages, group classes into business subpackages whenever the domain provides a coherent distinction, such as orders, payments, or inventory. For order-related classes, use `controller.orders`, `service.orders`, `repository.orders`, `model.dto.orders`, and `model.entity.orders`; payment-related classes belong in the corresponding `payments` subpackages. Use the same business vocabulary across applicable layers, while creating only the packages that have actual responsibilities. Do not force every business area to have every layer or split a small cohesive package merely for symmetry.

When an enclosing business module already provides the grouping, keep that boundary and avoid repeating its name: use `orders.controller` rather than `orders.controller.orders`. Add deeper business subpackages only when there is a distinct responsibility within that module, such as `orders.service.fulfillment`. This refines the layer convention and follows the same scoped-adoption rules below.

Keep DTOs and persistence entities as distinct types in their respective subpackages. Their shared `model` parent does not make them interchangeable or part of the business core; keep the dependency direction described above and map representations at the appropriate boundary. Do not expose managed entities as API responses.

Repository interfaces must be declared in a `repository` package or its business subpackages, with matching source directories. This includes Spring Data interfaces, custom repository contracts, and domain-owned repository ports. Do not place repository interfaces directly in `service`, `model`, or a generic `port` package.

Preserve business module boundaries: for example, use `orders.controller`, `orders.repository`, and `orders.model.dto` within an existing `orders` module. Do not merge module internals into application-wide layer packages. Where inner and outer boundaries are separate, place domain-owned repository interfaces in an inner package such as `orders.domain.repository`, and Spring Data interfaces and persistence adapters in an outer package such as `orders.infrastructure.repository`. Both boundaries use a `repository` package while dependencies continue pointing inward; domain contracts must not extend Spring Data interfaces or expose persistence-specific types.

Apply the convention to new applications and new classes within the task's scope. For existing code, a local bug fix does not by itself require moving the edited class or reorganizing unrelated packages. If adoption conflicts with enforced module boundaries, scanning configuration, or a published fully qualified class name, identify the concrete conflict and contain the exception. When package moves are within scope, update declarations, imports, affected Spring component/entity/repository scanning, configuration references, and test packages together, then run the relevant checks.

## Java choices

- Use records for suitable value/transport types, not automatically for JPA entities. Records are only shallowly immutable; copy mutable collections at ownership boundaries where callers must not mutate state.
- Express domain quantities deliberately: money needs a currency and defined decimal arithmetic/rounding; time-sensitive logic benefits from an injected `Clock` and explicit zone semantics. Preserve an existing serialization contract.
- Make nullability and ownership contracts consistent with the project's annotations and Java version. Avoid exposing mutable state or unstable dependency types in public APIs. Use standard JDK facilities when adequate rather than importing Guava merely because Google uses it.
- Use Lombok for mechanical boilerplate, following [code style and maintainability](code-style-and-maintainability.md). Keep domain invariants explicit and choose generated methods deliberately for persistence entities.

## Choose one execution model deliberately

| Context | Reasonable choice | Check |
|---|---|---|
| New web application with relational persistence | Spring MVC with Hibernate/JPA | Thin controllers, use-case transaction boundary, fetch plans and pool limits |
| Existing MVC with JDBC/JPA | Keep blocking request flow | Pool limits, query latency, deadlines |
| Localized change to existing WebFlux/R2DBC | Explain compatibility exception; keep reactive composition | No blocking calls on event loops; cancellation and bounded concurrency |
| Blocking I/O with supported virtual threads | Evaluate with representative load | Connection limits, memory, actual JDK behavior, observability |
| CPU-intensive work | Bound parallelism to available CPU | Contention and allocation; extra threads do not create CPU |

Do not pool virtual threads to throttle a scarce dependency; bound access to that dependency directly. Avoid large per-thread resource caches. Pinning behavior depends on the JDK, so check current documentation/JFR evidence instead of applying an old universal rule about `synchronized`. Prefer stable APIs; do not enable preview structured-concurrency APIs as a routine optimization.

Sources: [Boot requirements](https://docs.spring.io/spring-boot/system-requirements.html), [Modulith verification](https://docs.spring.io/spring-modulith/reference/verification.html), [OpenJDK JEP 444](https://openjdk.org/jeps/444), [Guava API compatibility notes](https://github.com/google/guava/blob/master/README.md), [DGS compatibility](https://github.com/Netflix/dgs-framework/blob/master/README.md).
