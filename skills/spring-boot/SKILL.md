---
name: spring-boot
description: Base engineering standards, architecture patterns, and build/test workflows for Java and Spring Boot applications. Invoke when creating endpoints, services, repositories, database migrations, security rules, or tests.
metadata:
  version: "1.0.0"
  domain: backend
  triggers: Spring Boot, Java, REST API, Spring Data JPA, Spring Security, Maven, Gradle
---

# Spring Boot Development Standards

## 1. Environment & Project Discovery

Before writing or modifying code, inspect the project root to align with existing conventions:

- **Build Tool**: Check for `pom.xml` (Maven) or `build.gradle` / `build.gradle.kts` (Gradle). Always prefer the
  project's wrapper (`./mvnw` or `./gradlew`) over global installations.
- **Java & Spring Version**: Check `<java.version>` / `sourceCompatibility` and the `spring-boot-starter-parent` (or
  Spring Boot plugin) version to ensure API compatibility (e.g., `jakarta.*` vs `javax.*`, Java `record` support).
- **Configuration**: Inspect `src/main/resources/application.yml` or `application.properties` (and profile variants like
  `application-dev.yml`) to understand active profiles, database drivers, and migration tools (Flyway/Liquibase).

## 2. Architecture & Package Structure

Follow a clean, layered, or package-by-feature architecture:

- **Controllers (`web` / `controller`)**: Handle HTTP routing, request validation (`@Valid`), and response mapping only.
  Never inject repositories directly into controllers.
- **Services (`service`)**: Contain business logic and transaction boundaries (`@Transactional` on public service
  methods; use `@Transactional(readOnly = true)` for read operations).
- **Repositories (`repository`)**: Extend `JpaRepository` or `ListCrudRepository`. Keep custom queries readable using
  Spring Data derived methods or explicit `@Query` annotations.
- **DTOs (`dto`)**: Never expose JPA `@Entity` classes directly in controller request/response payloads. Use immutable
  Java `record` types for DTOs with Jakarta Bean Validation annotations (`@NotBlank`, `@NotNull`, `@Size`).

## 3. Coding Conventions & Dependency Injection

- **Constructor Injection**: Always use constructor injection via `private final` fields (or Lombok's
  `@RequiredArgsConstructor` if Lombok is already in `pom.xml`/`build.gradle`). Avoid field injection (`@Autowired` on
  fields).
- **JPA Best Practices**:
    - Avoid `FetchType.EAGER` on `@OneToMany` and `@ManyToMany` associations; use `FetchType.LAZY` and `JOIN FETCH` or
      `@EntityGraph` to prevent N+1 query issues.
    - Never use `spring.jpa.hibernate.ddl-auto=update` or `create` in production profiles; rely on Flyway (
      `db/migration/V__*.sql`) or Liquibase changelogs when present.
- **Error Handling**: Centralize exception translation in a `@RestControllerAdvice` class returning standard RFC 9457
  `ProblemDetail` responses with appropriate HTTP status codes.
- **Configuration Properties**: Group custom properties using `@ConfigurationProperties` records rather than scattering
  `@Value` annotations across beans.

## 4. Testing Standards

Every new feature or bug fix must include automated tests matching the project's testing stack:

- **Unit Tests**: Use JUnit 5 and Mockito (`@ExtendWith(MockitoExtension.class)`) for fast, isolated service-layer tests
  without loading the Spring context.
- **Web Layer Tests**: Use `@WebMvcTest` with `MockMvc` to verify routing, serialization, validation errors, and
  security rules.
- **Persistence & Integration Tests**: Use `@DataJpaTest` or `@SpringBootTest` with Testcontainers (if configured in the
  project) rather than H2 when testing vendor-specific SQL.

## 5. Verification Workflow

Before finishing any task, run the appropriate verification command and ensure zero compilation or test failures:

- **Maven**: `./mvnw test-compile` (for quick syntax checks) and `./mvnw test` (to run the test suite).
- **Gradle**: `./gradlew testClasses` (for quick syntax checks) and `./gradlew test` (to run the test suite).
