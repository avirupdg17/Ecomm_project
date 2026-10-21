# Engineering standards

## Conventions

| Convention | Standard |
|---|---|
| Group / package | com.happiestminds.ecom.<service> with sub-packages api, service, domain, repository, security, client, config |
| Java version | 21 (records for DTOs, var allowed, no Lombok) |
| Spring Boot | 3.5.16, the final open-source 3.5 release (spring-boot-starter-parent; see Critical Notices and ADR-008), Jakarta EE namespaces |
| Build | Maven, mvn -B -ntp verify on GitHub Actions; no mvnw wrapper |
| Persistence | Spring Data JPA, H2; tests use jdbc:h2:mem, runtime uses jdbc:h2:file:./data/<service>db;LOCK_TIMEOUT=10000 |
| Money | java.math.BigDecimal with scale 2 and RoundingMode.HALF_UP; currency INR |
| IDs | UUID strings for all entity primary keys |
| Timestamps | java.time.Instant in UTC, fields createdAt, updatedAt |
| Error body | { "timestamp", "status", "error", "code", "message", "path", "correlationId", "details": [] } |
| Correlation | Header X-Correlation-Id: read if present, otherwise generated; put in MDC; propagated on every outbound call |
| Security | HS256 JWT, secret from JWT_SECRET env var, roles CUSTOMER, ADMIN, SERVICE |
| Health | Spring Boot Actuator /actuator/health (and /actuator/info) exposed without authentication |
| Tests | JUnit 5, Spring Boot Test, MockMvc, Mockito; integration tests named *IT and run in the verify phase |

## Approved dependencies

Copilot may only add dependencies from this list (all managed by the Spring Boot BOM unless a version is shown). Anything else requires trainer approval and an ADR.

Approved dependencies. Not approved: Lombok, MapStruct, Flyway/Liquibase, Resilience4j, Testcontainers, WireMock, OpenFeign, Kafka, Redis clients.

| Dependency | Use |
|---|---|
| spring-boot-starter-web | REST controllers |
| spring-boot-starter-validation | Bean Validation |
| spring-boot-starter-data-jpa | JPA repositories |
| spring-boot-starter-security | Spring Security |
| spring-boot-starter-actuator | Health and info endpoints |
| com.h2database:h2 (runtime) | Embedded database |
| io.jsonwebtoken:jjwt-api, jjwt-impl, jjwt-jackson version 0.12.6 | JWT creation and validation |
| spring-boot-starter-test (test) | JUnit 5, Mockito, AssertJ, MockMvc |
| org.springframework.security:spring-security-test (test) | @WithMockUser, JWT test helpers |
| maven-failsafe-plugin | Runs *IT integration tests in verify |

## Approved AI model

GPT-5.4 mini.

## CI

Builds and tests run on GitHub Actions (ci.yml, step 33).
