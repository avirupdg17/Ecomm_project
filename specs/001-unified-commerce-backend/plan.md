# Implementation Plan: Unified Commerce Backend

**Branch**: `001-unified-commerce-backend` | **Date**: 2026-10-21 | **Spec**: [spec.md](./spec.md)

**Input**: Feature specification from `/specs/001-unified-commerce-backend/spec.md`

## Summary

Deliver six independent Spring Boot REST services for identity/profile,
catalog, inventory, cart/pricing, order management, and simulated payment.
Each service owns a private file-based H2 database and integrates only through
typed REST DTOs. Order Management coordinates checkout as a persisted saga;
Inventory uses optimistic locking to prevent overselling; Order and Payment
use idempotency keys and request hashes so retries cannot duplicate effects.

## Technical Context

**Language/Version**: Java 21

**Primary Dependencies**: Spring Boot, Spring Web MVC, Spring Data JPA, Spring
Security, Bean Validation, H2, JWT (HS256), RestClient, Actuator, JUnit 5

**Storage**: One file-based H2 database per service under
`./data/<service>db`; in-memory H2 for unit tests

**Testing**: JUnit 5, Spring Boot Test, MockMvc, repository tests, service unit
tests, REST contract tests, and multi-process integration tests

**Target Platform**: Linux/JVM processes; six independently runnable Spring
Boot services on ports 8081-8086

**Project Type**: Multi-service REST web backend

**Performance Goals**: Product search p95 under 2 seconds during normal
operation; successful checkout p95 under 3 minutes; downstream calls use a
1-second connect and 2-second read timeout; 100 concurrent last-unit attempts
must confirm exactly one order

**Constraints**: No shared database, distributed transaction, message broker,
cache, or external runtime dependency. Cross-service calls preserve
`Authorization` and `X-Correlation-Id`; security controls fail closed; all
financial and inventory mutations are auditable.

**Scale/Scope**: First release for individual customers and retail
administrators; six services, one administrator role, one business currency,
cart quantities 1-100, and 15-minute unpaid inventory reservations.

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

- **Service ownership**: PASS. Each service has a bounded responsibility,
  private H2 database, local transactions, and REST-only integration.
- **Simplicity/changeability**: PASS. Synchronous `RestClient` is used because
  the runtime prohibits brokers; DTOs and versioned contracts isolate changes.
- **Security/customer protection**: PASS. JWT authentication, role checks,
  validation, correlation-aware logs, secret configuration, and negative tests
  are planned; payment credentials are never stored.
- **Correctness/financial safety**: PASS. Inventory optimistic locking and
  all-or-nothing reservations prevent overselling; the Order saga compensates
  failures; Payment and Order idempotency records prevent duplicates.
- **Test-first/review**: PASS. Each service has unit, authorization, contract,
  failure, retry, and integration coverage planned before implementation.

**Post-design re-check**: PASS. The generated data model, contracts, and
quickstart preserve private persistence, explicit API boundaries, fail-closed
authorization, local transaction scope, saga compensation, and the required
inventory/payment invariants.

## Project Structure

### Documentation

```text
specs/001-unified-commerce-backend/
├── plan.md
├── research.md
├── data-model.md
├── contracts/
└── quickstart.md
```

### Source Code

```text
ecom-identity-service/
├── pom.xml
└── src/{main,test}/java/com/happiestminds/ecom/identity/
    ├── api/ service/ domain/ repository/
    ├── security/ client/ config/
    └── resources/application.yml
ecom-catalog-service/       # port 8082; same layered layout
ecom-inventory-service/     # port 8083; same layered layout
ecom-cart-service/          # port 8084; same layered layout
ecom-order-service/         # port 8085; same layered layout
ecom-payment-service/       # port 8086; same layered layout
```

**Structure Decision**: Six independently buildable Maven Spring Boot
projects, one per service, with consistent layered packages and isolated test
suites. Shared design artifacts live under the feature directory; each
service owns its DTOs, entities, repositories, clients, and tests.

## Complexity Tracking

No constitution violations require justification. The six projects and saga
are explicitly required by the specification and platform constraints.
