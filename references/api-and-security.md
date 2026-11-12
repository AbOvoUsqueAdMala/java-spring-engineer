# API contracts and security

Read for web endpoints, serialization, authentication, authorization, and GraphQL. Preserve the project's public contract unless the task changes it.

## Boundary design

- Use explicit request/response types when needed to prevent overposting, accidental field exposure, persistence coupling, or lazy serialization. Do not copy a teaching application's direct entity binding into a public API without evaluating these risks.
- Validate nullability as well as ranges: many constraints accept null. Bound strings, collections, uploads, and pagination; validate nested input where relevant. Enforce business invariants in the use case, not exclusively in HTTP validation.
- Keep status codes and error shape consistent. For a suitable new Boot 3+ API, consider Spring `ProblemDetail`/RFC 9457 with stable error identifiers and safe validation details. Handle authentication/authorization errors at the appropriate Security filter-chain boundary; controller advice alone does not cover every failure.
- Preserve API compatibility when changing enums, nullability, dates, money, pagination, and schemas. Update the existing OpenAPI/contract tests when the interface changes. Document idempotency and concurrency semantics for commands; use conditional requests/version checks where the contract requires them.
- DGS is useful for a requested GraphQL API, not an automatic REST replacement. With GraphQL, consider query depth/complexity, field authorization, request-scoped data loading, and N+1 behavior. Verify DGS/Boot compatibility and test the schema/query contract.

## Authentication is not authorization

Use established Spring Security resource-server/session support and the existing identity provider; do not invent JWT signing/validation in a custom filter for a standard OAuth2 case.

For JWTs, verify the configured issuer, signature/allowed algorithms, validity times, audience requirements, key rotation, and authority mapping. Issuer configuration alone must not be assumed to validate the intended audience. Tests using a mocked authenticated principal bypass token decoding, so add decoder/integration checks when changing token validation.

For every protected operation, determine who may access this specific resource and tenant. Role checks alone can leave object-level authorization gaps. Derive tenant/owner identity from trusted context, scope database queries accordingly, and test another user's/tenant's resource. UUIDs and unguessable IDs are not access control.

## CSRF, CORS, and exposure

- Preserve CSRF protection for browser flows with cookies, sessions, or other credentials automatically attached by the browser. Calling an API "stateless" is insufficient justification to disable CSRF; bearer tokens in cookies still require analysis. A dedicated API using only explicitly supplied bearer headers may legitimately omit CSRF protection for that chain.
- CORS is a browser cross-origin policy, not authentication. Allow required origins/methods/headers; do not combine credentialed requests with wildcard origins. Preserve legitimate browser/SSO flows when choosing cookie settings.
- Configure `SecurityFilterChain` matchers and order for actual endpoint groups; verify unmatched paths. Do not expose all Actuator endpoints with a broad permit rule.
- Trust forwarded client/host headers only behind an explicitly configured trusted proxy that sanitizes them. A forwarding filter alone does not establish trust.
- Use placeholders/secret providers for credentials, parameterized SQL for values, and safe outbound URL handling for user-controlled destinations. Address upload/SSRF/password-storage details when that feature exists, rather than installing a generic security subsystem into an unrelated task.

Test both permitted and denied paths, including anonymous access, wrong role/scope, cross-tenant access, invalid input, and the applicable CSRF/CORS behavior. Keep test security enabled when it is the behavior under test.

Sources: [Spring Security JWT](https://docs.spring.io/spring-security/reference/servlet/oauth2/resource-server/jwt.html), [CSRF](https://docs.spring.io/spring-security/reference/servlet/exploits/csrf.html), [Zalando API/event guidelines](https://opensource.zalando.com/restful-api-guidelines/), [Netflix DGS](https://github.com/Netflix/dgs-framework/blob/master/README.md). Use portable contract principles from company guidelines; do not import company-specific hostnames, naming policies, or mandatory platforms.
