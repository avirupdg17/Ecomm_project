# E-Commerce Platform Architecture

## System shape

The platform is six independently deployable Spring Boot REST services. Each
service exposes its own API and owns its business data; cross-service workflows
are coordinated through synchronous HTTP calls.

See the [Mermaid architecture diagram](./architecture-diagram.md) for a
visual view of service boundaries, private databases, communication, and the
Order saga.

| Service | Port | Responsibility and boundary |
|---|---:|---|
| Identity | 8081 | Registration, authentication, profiles, and issuing/validating bearer tokens. It owns users and credentials; no other service reads identity data directly. |
| Catalog | 8082 | Product and category browsing, search, filtering, and administration. It owns product and category data; it does not reserve stock or price an order. |
| Inventory | 8083 | Stock levels, concurrency-safe reservations, release/restock, and fulfillment updates. It owns availability and reservation state; it does not own products, carts, or orders. |
| Cart | 8084 | Customer cart lines and cart pricing/discount calculation. It owns cart state and calculated totals; it does not create orders or reserve inventory. |
| Order | 8085 | Order creation, lifecycle, cancellation, idempotency, and workflow coordination. It owns order state and is the saga orchestrator; it does not directly mutate inventory or payment data. |
| Payment | 8086 | Simulated payment, refund, idempotency, and payment outcome tracking. It owns payment transactions; it does not transition order state except through the Order API's workflow. |

### Data ownership

| Rule | Required behavior |
|---|---|
| Database per service | Every service owns a separate H2 database. Schema migrations, transactions, repositories, and retention are local to that service. |
| No shared persistence | A service must never connect to, query, or update another service's H2 database, tables, files, or entities. |
| API-only integration | Cross-service data is obtained through the owning service's REST API. IDs and response DTOs cross boundaries; domain entities do not. |
| Consistency | Local transactions provide atomicity within a service. The Order saga and compensating calls handle consistency across services. |

## Service implementation structure

Each service follows the same layered package layout:

```text
com.happiestminds.ecom.<service>.{
  api, service, domain, repository, security, client, config
}
```

| Package | Responsibility |
|---|---|
| `api` | REST controllers, request/response DTOs, validation, and HTTP status mapping. |
| `service` | Use cases, local transactions, orchestration decisions, and idempotency handling. |
| `domain` | Service-owned entities, value objects, enums, and business rules. |
| `repository` | Persistence ports and Spring Data H2 implementations for local data only. |
| `security` | Bearer-token validation, authorization, and service-token handling. |
| `client` | Typed REST clients for other services; no repository or entity sharing. |
| `config` | Application, HTTP client, timeout, security, and infrastructure configuration. |

## Communication and security

Service-to-service calls are synchronous REST calls made with Spring's
`RestClient`. Clients must use a **1 second connect timeout** and a **2 second
read timeout**, map non-success responses explicitly, and preserve enough
context for retries and reconciliation.

| Concern | Contract |
|---|---|
| Correlation | Accept or create `X-Correlation-Id` at the API edge and propagate the same value on every downstream request. Include it in logs and error responses where applicable. |
| User requests | Propagate the caller's `Authorization: Bearer <token>` when a downstream call acts on behalf of that user. Each receiving service validates the token and authorizes its own resources. |
| Background work | Use a short-lived, explicitly scoped `SERVICE` token for scheduled, retry, reconciliation, or compensation calls that have no user request context. Never fabricate a user bearer token. |
| Boundaries | Clients exchange versioned DTOs and stable identifiers only. A timeout or non-2xx response is an explicit workflow outcome, not a silent success. |

## Order saga

Order Service is the single saga orchestrator. It persists each transition and
invokes Inventory and Payment through their APIs; it does not use a distributed
transaction. Calls and compensations must be idempotent so a retry cannot
reserve, charge, refund, or restock twice.

| From | Event/outcome | To | Required action |
|---|---|---|---|
| `CREATED` | Reservation succeeds | `STOCK_RESERVED` | Request payment. |
| `CREATED` | Stock cannot be reserved | `STOCK_UNAVAILABLE` | Stop; no payment is attempted. |
| `CREATED` | Customer cancellation | `CANCELLED` (`INVENTORY_UNAVAILABLE`) | Record cancellation reason; release any applicable reservation. |
| `STOCK_RESERVED` | Payment processing starts | `PAYMENT_PENDING` | Persist payment correlation/idempotency data. |
| `PAYMENT_PENDING` | Payment succeeds | `CONFIRMED` | Finalize inventory and order. |
| `PAYMENT_PENDING` | Payment fails | `PAYMENT_FAILED` | Begin compensation. |
| `PAYMENT_FAILED` | Compensation required | `COMPENSATION_PENDING` | Release reservation and/or refund as applicable. |
| `PAYMENT_PENDING` | Reservation is lost | `COMPENSATION_PENDING` (`RESERVATION_LOST`) | Request a refund; then complete cancellation. |
| `COMPENSATION_PENDING` | Compensation completes | `CANCELLED` | Preserve the reason (`INVENTORY_UNAVAILABLE`, payment failure, or `RESERVATION_LOST`). |
| `CONFIRMED` | Customer cancellation | `CANCELLED` | Request refund and inventory restock according to local policy. |

## Runtime and delivery constraints

The runtime deliberately has no infrastructure beyond the Spring Boot
processes and their local H2 databases. The following are **prohibited runtime
dependencies**: Docker, Kubernetes, Kafka, RabbitMQ, Redis, and any external
database.

Container images and Kubernetes manifests may exist only as optional packaging
artifacts exercised inside GitHub Actions. They are not required, started, or
used by the application runtime or local service-to-service behavior.

## Boundary decisions

| Decision | Owning service | Data it owns | Data it only references (logical ID) | Called by | ADR |
|---|---|---|---|---|---|
| Identity and access management | Identity | Users, credentials, profiles, and token subjects | Order, cart, and audit actors (`userId`) | Customer, Admin, and every service validating a token | [ADR-004](./architecture-decisions/ADR-004-stateless-hs256-jwt.md) |
| Product and category catalog | Catalog | Products and categories | Cart lines and orders (`productId`) | Customer, Admin, and Cart | [ADR-002](./architecture-decisions/ADR-002-database-per-service-h2.md), [ADR-003](./architecture-decisions/ADR-003-synchronous-rest-client.md) |
| Stock and reservation management | Inventory | Availability, stock versions, reservations, and fulfillment updates | Products and orders (`productId`, `orderId`) | Order and checkout orchestration | [ADR-002](./architecture-decisions/ADR-002-database-per-service-h2.md), [ADR-006](./architecture-decisions/ADR-006-optimistic-inventory-locking.md) |
| Cart management and pricing | Cart | Cart lines, cart state, and calculated totals | Users and catalog products (`userId`, `productId`) | Customer and Order | [ADR-002](./architecture-decisions/ADR-002-database-per-service-h2.md), [ADR-003](./architecture-decisions/ADR-003-synchronous-rest-client.md) |
| Order lifecycle and persistence | Order | Orders, order transitions, idempotency records, and workflow correlation data | Users, cart contents, reservations, and payments (`userId`, `cartId`, `reservationId`, `paymentId`) | Customer and checkout orchestration | [ADR-002](./architecture-decisions/ADR-002-database-per-service-h2.md), [ADR-005](./architecture-decisions/ADR-005-idempotency-key-request-hash.md) |
| Payment processing and refunds | Payment | Payment transactions, outcomes, refunds, and idempotency records | Orders and users (`orderId`, `userId`) | Order and checkout orchestration | [ADR-002](./architecture-decisions/ADR-002-database-per-service-h2.md), [ADR-005](./architecture-decisions/ADR-005-idempotency-key-request-hash.md) |
| Checkout orchestration and compensation | Order Service | Saga state, transition history, downstream call results, and compensation decisions | Cart, inventory, and payment state (`cartId`, `reservationId`, `paymentId`) | Customer checkout requests; Inventory and Payment as downstream services | [ADR-003](./architecture-decisions/ADR-003-synchronous-rest-client.md), [ADR-005](./architecture-decisions/ADR-005-idempotency-key-request-hash.md) |
