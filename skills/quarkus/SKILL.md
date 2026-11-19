---
name: quarkus
description: Engineering standards for Quarkus applications running in JVM mode on Java 21+ with Imperative/Virtual Threads and Hibernate ORM with Panache (Repository pattern). Invoke when building Jakarta REST resources, ArC CDI beans, Panache repositories, or Quarkus tests.
metadata:
  version: "1.0.0"
  domain: backend
  triggers: Quarkus, Java 21, Virtual Threads, Panache, Jakarta REST, ArC, Hibernate ORM
---

# Quarkus Engineering Standards & Agent Guidelines

## 1. Environment & Project Discovery

Before writing or modifying code, inspect the project root to align with existing conventions:

- **Build Tool**: Check for `pom.xml` (Maven) or `build.gradle` / `build.gradle.kts` (Gradle). Always prefer the project
  wrapper (`./mvnw` or `./gradlew`) or the Quarkus CLI (`quarkus`) if installed.
- **Target Stack**: This project targets **JVM mode** on **Java 21+** using **Imperative / Virtual Threads** and *
  *Hibernate ORM with Panache (Repository pattern)**.
- **Configuration**: Inspect `src/main/resources/application.properties` or `application.yaml`. Note profile prefixes (
  `%dev.`, `%test.`, `%prod.`) and Quarkus Dev Services settings (automatic zero-config containers for databases, Kafka,
  Redis, etc.).

## 2. Architecture & Package Structure

Follow a clean, layered, or package-by-feature architecture:

- **Resources (`resource` / `web`)**: Jakarta REST endpoints (`@Path`, `@GET`, `@POST`, etc.). Keep resources thin:
  handle HTTP routing, Jakarta Bean Validation (`@Valid`), and response mapping only. Never inject repositories or
  `EntityManager` directly into resources.
- **Services (`service`)**: `@ApplicationScoped` beans containing business logic and transaction boundaries (
  `@Transactional`).
- **Repositories (`repository`)**: `@ApplicationScoped` classes implementing `PanacheRepository<Entity>` (or
  `PanacheRepositoryBase<Entity, IdType>`). Keep custom queries readable using Panache query methods or HQL.
- **Entities (`entity` / `domain`)**: Standard JPA `@Entity` classes (do not extend `PanacheEntity` since the repository
  pattern is enforced).
- **DTOs (`dto`)**: Never expose JPA `@Entity` classes directly in REST payloads. Use immutable Java `record` types with
  Jakarta Validation annotations (`@NotBlank`, `@NotNull`, `@Size`).

## 3. Programming Model: Imperative & Virtual Threads

- **Virtual Threads**: Because this application uses blocking Hibernate ORM on Java 21+, annotate blocking REST resource
  classes or methods with `@RunOnVirtualThread` (from `io.smallrye.common.annotation.RunOnVirtualThread`) so requests
  execute on virtual threads instead of blocking the Vert.x event loop.
- **Imperative Style**: Write straightforward synchronous/imperative code (`T`, `List<T>`, `Optional<T>`). Do not
  introduce Mutiny reactive types (`Uni<T>`, `Multi<T>`) unless integrating with a specific reactive-only extension.
- **Virtual Thread Safety**: Avoid `synchronized` blocks around blocking I/O calls (use `ReentrantLock` if locking is
  required) and avoid heavy `ThreadLocal` caching to keep virtual threads lightweight and free from pinning.

## 4. Quarkus ArC (CDI) & Configuration Conventions

- **Dependency Injection**: Use constructor injection via `private final` fields on `@ApplicationScoped` beans. Avoid
  field injection (`@Inject` on non-private fields) except in test classes.
- **Bean Scopes**: Default to `@ApplicationScoped` for stateless services and repositories (ArC client proxies perform
  better than `@Singleton` and simplify mocking in tests).
- **Configuration Mapping**: Group related configuration properties using `@ConfigMapping(prefix = "...")` interfaces
  rather than scattering `@ConfigProperty` annotations across beans.
- **JVM Mode Flexibility**: Standard JVM reflection and dynamic libraries are permitted when needed, though build-time
  Quarkus extensions remain preferred for fast startup and low memory footprint.

## 5. Persistence & Error Handling

- **Panache Repository Pattern**:
    - Encapsulate all database queries inside dedicated `*Repository` classes implementing `PanacheRepository`.
    - Avoid `FetchType.EAGER` on `@OneToMany` and `@ManyToMany` relationships; use `FetchType.LAZY` and explicit
      `JOIN FETCH` queries to prevent N+1 query issues.
    - Never use `quarkus.hibernate-orm.database.generation=update` or `drop-and-create` in `%prod`. Use Flyway (
      `quarkus-flyway` with `db/migration/V__*.sql`) or Liquibase when present.
- **Exception Handling**: Centralize error responses using `@ServerExceptionMapper` methods (from `quarkus-rest`)
  returning standardized RFC 9457 `ProblemDetail` or structured JSON error payloads.

## 6. Testing Standards

Every feature or bug fix must include automated tests aligned with Quarkus testing conventions:

- **Unit Tests**: Use plain JUnit 5 and Mockito (`@ExtendWith(MockitoExtension.class)`) for fast, isolated service logic
  tests without booting Quarkus.
- **Integration & Endpoint Tests**: Use `@QuarkusTest` with **REST Assured** (`given().when().get(...)`) to test HTTP
  endpoints, virtual thread execution, validation, and serialization. Use `@InjectMock` (from `quarkus-junit5-mockito`)
  to stub repositories or external clients when needed.
- **Dev Services**: Rely on Quarkus Dev Services in `%test` mode for real database integration tests rather than
  spinning up manual Docker containers or mocking the persistence layer.

## 7. Verification Workflow

Before completing any task, verify compilation, ArC dependency wiring, and tests using the project wrapper:

- **Maven**: `./mvnw test-compile` (for quick build and CDI bean validation checks) and `./mvnw test` (to run the test
  suite).
- **Gradle**: `./gradlew testClasses` (for quick checks) and `./gradlew test` (to run the test suite).
