# Code style and maintainability

Read whenever authoring or changing Java code, including tests and examples. These are the user's required engineering standards.

## Java style with tab indentation

Use actual tab characters (`U+0009`) for indentation in Java source, tests, and examples, with one tab per block nesting level and additional tabs for continuation indentation. Do not emulate indentation with one, two, or more spaces. The editor's tab display width is separate from the stored character; preserve a consistent project setting. Spaces between tokens remain normal, and formatting must not change whitespace inside string literals or text blocks.

Use the [official Google guide](https://google.github.io/styleguide/javaguide.html) for the remaining compatible rules: UTF-8, a 100-character line limit with the guide's exceptions, explicit imports, braces for control flow, naming, and documentation. Use `UpperCamelCase` for classes, `lowerCamelCase` for methods and variables, and `UPPER_SNAKE_CASE` for constants as defined by the guide. The user's tab requirement takes precedence over the guide's whitespace and indentation rules; do not describe this customized style as strict Google Java Style.

Use a formatter/checker configuration that preserves required tab indentation. When formatting configuration changes are within scope, align the editor, formatter, and lint rules, then run the relevant checks. Remove conflicting space-indentation rules from that configuration rather than maintaining two incompatible requirements. Do not select or run a formatter that rewrites these tabs to spaces. If an enforced build configuration cannot be aligned within the task's scope, identify the concrete conflict instead of silently reverting to spaces or disabling checks. Keep unrelated reformatting outside a localized change. Formatting does not replace naming or design review.

Relevant sections: [imports, 3.3](https://google.github.io/styleguide/javaguide.html#s3.3-import-statements), [formatting, 4](https://google.github.io/styleguide/javaguide.html#s4-formatting), [naming, 5](https://google.github.io/styleguide/javaguide.html#s5-naming), and [Javadoc, 7](https://google.github.io/styleguide/javaguide.html#s7-javadoc).

## Full names that explain intent

- Name every authored variable, parameter, class, and method for its domain concept, value, or responsibility. Use full words; do not create abbreviations such as `svc`, `repo`, `cfg`, `req`, `resp`, `cnt`, or `tmp`, or single-letter local names. Prefer `orderIndex`, `retryAttemptCount`, `customerIdentifier`, `PaymentService`, and `calculateInvoiceTotal`.
- Make units, collection contents, and boolean meaning apparent where needed: `timeoutMilliseconds`, `pendingInvoices`, `isPaymentConfirmed`. Avoid vague `data`, `object`, `manager`, or `process` when a precise business name is available.
- Prefer descriptive generic type names such as `ResponseT`, permitted by Google Style. Keep standard external API/type names and contract-required identifiers intact; map them to meaningful internal names rather than breaking compatibility.
- Apply this stricter naming policy even where Google's examples use short names. Rename internal symbols consistently within scope; preserve public API payloads, database identifiers, and other published contracts unless a compatible migration is part of the task.

## Clean Code in each layer

Use Robert C. Martin's Clean Code principles throughout the application: focused classes and methods, clear responsibilities, one coherent level of abstraction, and explicit side effects. Express business invariants in domain operations, separate transport/mapping from policy, and remove duplicated business knowledge when there is a real shared concept. Prefer simple composition and SOLID principles over speculative abstractions. Comments explain decisions and constraints that names and code cannot convey; avoid narrating obvious statements.

Use [Clean Architecture and MVC](architecture-and-compatibility.md#clean-architecture-and-mvc) for dependency direction and boundaries. These principles do not require a fixed number of layers or a separate interface for every class. Apply the user's method-size rule below.

## Methods fit one screen

Keep each handwritten method visible on one ordinary editor screen at normal readable zoom. Use the project's established screen-height convention when known. Otherwise, use a maximum of 40 physical source lines, counting from the first signature line through the closing brace and including blank lines and comments, as this skill's working convention. Honor a stricter project method-length limit. Forty lines is a practical default for this skill, not a universal measurement of screen height.

When a method exceeds this limit, extract coherent business steps into clearly named helper methods, normally private methods of the same class. Java has no named methods nested directly inside other methods; the original method calls the extracted helpers. Each helper must satisfy the same size rule and express a meaningful responsibility. Move a responsibility to another component only when its ownership warrants it.

Preserve behavior, validation order, exception handling, transaction boundaries, and other side effects when extracting. Moving an annotation onto a private or self-invoked helper does not establish a new Spring proxy boundary; apply the existing transaction and reliability rules. Run checks appropriate to the changed behavior. Do not meet the limit by compressing statements, deleting useful explanations, shrinking the font, or hiding a large implementation in a lambda or anonymous class.

## Lombok for mechanical boilerplate

- Use Lombok broadly where it preserves the intended semantics: `@RequiredArgsConstructor` with final dependencies for constructor injection, focused `@Getter`/`@Setter`, `@Slf4j` for logging, and `@Builder` for suitable construction. Keep validation, authorization, state transitions, and other business behavior explicit.
- Resolve a Lombok version compatible with the actual JDK and platform. Configure annotation processing for the build and relevant test source sets, and verify generated code with the normal compiler. IDE support alone does not prove the build works.
- Generate setters only where mutation is permitted. Builders and generated constructors must preserve required validation and invariants. Retain suitable existing records instead of converting them into classes just to add Lombok.
- Do not put `@Data` on JPA entities. It bundles setters, equality, hashing, and string representation that may be unsuitable for managed identity and associations. Use focused annotations and explicit lifecycle-safe equality where needed; exclude associations and sensitive fields from generated string output, and never derive entity hashing from mutable fields.
- Provide JPA's required public/protected no-argument constructor, using `@NoArgsConstructor(access = AccessLevel.PROTECTED)` when appropriate. Do not force final fields to null/zero merely to make generation compile. Avoid `@Value` on JPA entities; its final/immutable shape is unsuitable for portable JPA entities and common proxy strategies.

Sources: [Lombok constructor generation](https://projectlombok.org/features/constructor), [@Data behavior](https://projectlombok.org/features/Data), [@Value behavior](https://projectlombok.org/features/Value). The entity restrictions above are this skill's engineering safeguards, not a claim that Lombok itself forbids these annotations.
