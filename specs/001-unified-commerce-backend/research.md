# Research: Unified Commerce Backend

## Decision: Six independent Spring Boot services

- **Decision**: Keep Identity, Catalog, Inventory, Cart & Pricing, Order
  Management, and Payment as separate Maven projects on ports 8081-8086.
- **Rationale**: This preserves service ownership, independent builds, private
  persistence, and explicit REST boundaries required by the constitution.
- **Alternatives considered**: A single application or shared database would
  simplify local calls but violate ownership and couple releases and failures.

## Decision: Java 21, Spring Boot, and H2

- **Decision**: Use Java 21 with Spring MVC, Spring Data JPA, Spring Security,
  Bean Validation, Actuator, H2, and JUnit 5. Use one file-based H2 database
  per service and in-memory H2 for unit tests.
- **Rationale**: This provides typed REST controllers, local transactions,
  validation, health endpoints, and independently testable services without
  external runtime infrastructure.
- **Alternatives considered**: A reactive stack adds complexity to a bounded
  synchronous workflow; a shared or external database violates ownership and
  first-release infrastructure constraints.

## Decision: Synchronous RestClient integration

- **Decision**: Use typed Spring `RestClient` clients with 1-second connect and
  2-second read timeouts, explicit non-2xx mapping, bearer/service token
  propagation, and `X-Correlation-Id` propagation.
- **Rationale**: Checkout and payment require immediate outcomes, while runtime
  constraints prohibit Kafka, RabbitMQ, Redis, and external services.
- **Alternatives considered**: Messaging adds prohibited infrastructure and
  asynchronous semantics; direct database access violates ownership.

## Decision: JWT authentication and authorization

- **Decision**: Identity issues stateless HS256 bearer tokens. Each service
  validates tokens and authorizes its own resources; background work uses a
  scoped SERVICE token.
- **Rationale**: Stateless tokens keep services independently deployable while
  enforcing customer ownership and the single ADMIN role at every boundary.
- **Alternatives considered**: Shared sessions add a dependency; trusting
  caller-supplied user IDs fails the security boundary.

## Decision: Optimistic inventory locking

- **Decision**: Inventory stock and reservations carry version fields. A
  reservation validates every requested line before atomically decrementing
  available stock, with bounded retry on optimistic-lock conflicts.
- **Rationale**: This prevents overselling under concurrent last-unit attempts
  without distributed locks or shared persistence.
- **Alternatives considered**: Pessimistic locks reduce concurrency; distributed
  locks add infrastructure prohibited by the runtime constraints.

## Decision: Persisted Order saga

- **Decision**: Order Management is the sole saga orchestrator. It persists
  every transition, calls Inventory and Payment, and performs idempotent
  release, refund, and restock compensation after failure or timeout.
- **Rationale**: Local transactions cannot span services, so explicit
  compensation preserves customer-visible consistency and recovery.
- **Alternatives considered**: Distributed transactions are unavailable over
  REST and violate autonomy; an event broker is prohibited.

## Decision: Idempotency keys with request hashes

- **Decision**: Order and Payment creation require an idempotency key and store
  a canonical request hash plus original response. Same key and hash returns
  the original result; same key with a different hash returns a conflict.
- **Rationale**: Retries after timeouts cannot create duplicate orders, charges,
  reservations, refunds, or compensation effects.
- **Alternatives considered**: Client-generated IDs alone do not detect changed
  payloads or prove that the first attempt completed.

## Decision: Operational diagnostics

- **Decision**: Every service exposes Actuator health, accepts or creates
  `X-Correlation-Id`, includes it in logs and error responses, and records saga,
  idempotency, compensation, and audit transitions.
- **Rationale**: Correlation IDs and health endpoints provide the required
  diagnostic baseline without central infrastructure.
- **Alternatives considered**: Distributed tracing infrastructure is outside
  first-release scope.
