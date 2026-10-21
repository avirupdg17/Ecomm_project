---

description: "Executable, test-first implementation tasks for the unified commerce backend"
---

# Tasks: Unified Commerce Backend

**Input**: Design documents from `/specs/001-unified-commerce-backend/`

**Prerequisites**: [plan.md](./plan.md), [spec.md](./spec.md), [research.md](./research.md), [data-model.md](./data-model.md), [contracts/](./contracts/), [quickstart.md](./quickstart.md)

**Implementation order**: Identity and Catalog first; Inventory next; Cart & Pricing and Payment next; Order Management last.

**Test-first rule**: Every service's unit, contract, authorization, failure, concurrency, and retry tests below must be written and made to fail before the corresponding production task.

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Create six independently buildable Java 21 Spring Boot services without shared persistence.

- [ ] T001 Create the six Maven service directories and layered package/resource structure from plan.md in `ecom-identity-service/src/`, `ecom-catalog-service/src/`, `ecom-inventory-service/src/`, `ecom-cart-service/src/`, `ecom-order-service/src/`, and `ecom-payment-service/src/` (FR-001–FR-020)
- [ ] T002 [P] Configure Java 21, Spring Boot, MVC, Data JPA, Security, Validation, H2, Actuator, RestClient, and JUnit 5 dependencies in `ecom-identity-service/pom.xml`, `ecom-catalog-service/pom.xml`, `ecom-inventory-service/pom.xml`, `ecom-cart-service/pom.xml`, `ecom-order-service/pom.xml`, and `ecom-payment-service/pom.xml` (FR-001–FR-020)
- [ ] T003 [P] Configure ports 8081–8086, file-based H2 paths `./data/<service>db`, test H2 profiles, and environment-backed secrets in each service `src/main/resources/application.yml` (FR-001, FR-018, FR-020)
- [ ] T004 [P] Add service-local test fixtures and shared test profile configuration in `ecom-identity-service/src/test/resources/application-test.yml`, `ecom-catalog-service/src/test/resources/application-test.yml`, `ecom-inventory-service/src/test/resources/application-test.yml`, `ecom-cart-service/src/test/resources/application-test.yml`, `ecom-order-service/src/test/resources/application-test.yml`, and `ecom-payment-service/src/test/resources/application-test.yml` (FR-018, FR-020)

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Establish the cross-cutting behavior required by every service before service-specific work begins.

**⚠️ CRITICAL**: No user-story implementation may begin until this phase is complete.

- [ ] T005 [P] Write failing correlation-ID filter and error-response tests in `ecom-identity-service/src/test/java/com/happiestminds/ecom/identity/config/CorrelationIdTest.java`, then implement request generation/propagation and structured error mapping in `ecom-identity-service/src/main/java/com/happiestminds/ecom/identity/config/CorrelationIdFilter.java` and `ecom-identity-service/src/main/java/com/happiestminds/ecom/identity/api/ApiExceptionHandler.java` (FR-018–FR-020)
- [ ] T006 [P] Write failing correlation-ID and error-contract tests in each remaining service `src/test/java/com/happiestminds/ecom/<service>/config/CorrelationIdTest.java`, then implement the equivalent filter and exception handler in each service `src/main/java/com/happiestminds/ecom/<service>/config/` and `api/` packages (FR-018–FR-020)
- [ ] T007 [P] Write failing Actuator health contract tests in `ecom-identity-service/src/test/java/com/happiestminds/ecom/identity/api/HealthContractTest.java` and the five other services' `src/test/java/com/happiestminds/ecom/<service>/api/HealthContractTest.java`, then expose `/actuator/health` in each service's security/configuration files (FR-018, SC-002)
- [ ] T008 [P] Write failing JWT validation and fail-closed authorization tests in `ecom-identity-service/src/test/java/com/happiestminds/ecom/identity/security/JwtSecurityTest.java` and corresponding tests in every other service, then implement token validation, role checks, and service-token scope in each service `security/` package (FR-001, FR-002, FR-015–FR-017, FR-020)
- [ ] T009 [P] Write failing typed REST timeout, bearer propagation, correlation propagation, and non-2xx mapping tests in `ecom-order-service/src/test/java/com/happiestminds/ecom/order/client/RestClientConfigurationTest.java`, then implement the shared pattern in each service that calls another service under `client/` with 1-second connect and 2-second read timeouts (FR-006–FR-011, FR-020)
- [ ] T010 [P] Add failing repository transaction/audit baseline tests in each service `src/test/java/com/happiestminds/ecom/<service>/repository/TransactionAuditTest.java`, then configure local transaction boundaries and correlation-aware audit logging in each service `config/` and `service/` package (FR-018, Constitution IV)

**Checkpoint**: All six services build independently, expose health/correlation behavior, reject unauthorized requests, and use only private local persistence.

---

## Phase 3: User Story 1 - Discover and Purchase Products (Priority: P1) 🎯 MVP — Identity and Catalog

**Goal**: Let a customer register, authenticate, maintain a profile, browse current products and prices, and give later checkout services authoritative customer/catalog APIs.

**Independent test**: Identity and Catalog tests pass with a newly registered customer, protected profile access, product/category search, current price, active status, and clear empty results.

### Identity tests first

- [ ] T011 [P] [US1] Write failing registration validation, duplicate lower-case email, BCrypt storage, and no-partial-profile tests in `ecom-identity-service/src/test/java/com/happiestminds/ecom/identity/service/RegistrationServiceTest.java` (FR-001, edge cases)
- [ ] T012 [P] [US1] Write failing login success, invalid-credential, locked-user, and JWT-claims tests in `ecom-identity-service/src/test/java/com/happiestminds/ecom/identity/api/AuthControllerTest.java` (FR-001–FR-002)
- [ ] T013 [P] [US1] Write failing owner/admin profile authorization and update-validation tests in `ecom-identity-service/src/test/java/com/happiestminds/ecom/identity/api/ProfileControllerTest.java` (FR-001–FR-002, FR-019)
- [ ] T014 [P] [US1] Write failing Identity REST contract tests for register, login, `/api/users/me`, and health in `ecom-identity-service/src/test/java/com/happiestminds/ecom/identity/api/IdentityContractTest.java` (FR-001–FR-002, contracts/identity.yaml)

### Identity implementation

- [ ] T015 [P] [US1] Implement User and UserProfile entities with unique lower-case email, `ACTIVE|LOCKED` status, `CUSTOMER|ADMIN` role, timestamps, and BCrypt password hashes in `ecom-identity-service/src/main/java/com/happiestminds/ecom/identity/domain/User.java` and `UserProfile.java` (FR-001–FR-002, data-model.md)
- [ ] T016 [P] [US1] Implement repositories and validation DTOs for registration, login, and profile updates in `ecom-identity-service/src/main/java/com/happiestminds/ecom/identity/repository/` and `api/` (FR-001–FR-002, FR-019)
- [ ] T017 [US1] Implement registration, authentication, and owner/admin profile services in `ecom-identity-service/src/main/java/com/happiestminds/ecom/identity/service/IdentityService.java` and `JwtTokenService.java` (FR-001–FR-002)
- [ ] T018 [US1] Implement `/api/users/register`, `/api/auth/login`, and `/api/users/me` controllers with contract status/error responses in `ecom-identity-service/src/main/java/com/happiestminds/ecom/identity/api/` (FR-001–FR-002, FR-019)

### Catalog tests first

- [ ] T019 [P] [US1] Write failing product/category uniqueness, uppercase SKU, positive scale-2 INR price, current-price validity, and inactive-product tests in `ecom-catalog-service/src/test/java/com/happiestminds/ecom/catalog/service/CatalogServiceTest.java` (FR-003, FR-015, data-model.md)
- [ ] T020 [P] [US1] Write failing name/category/availability search and explicit empty-result tests in `ecom-catalog-service/src/test/java/com/happiestminds/ecom/catalog/api/ProductSearchControllerTest.java` (FR-003, FR-019, SC-002)
- [ ] T021 [P] [US4] Write failing admin-only create/update/deactivate authorization tests in `ecom-catalog-service/src/test/java/com/happiestminds/ecom/catalog/api/AdminCatalogControllerTest.java` (FR-015, FR-019, SC-009)
- [ ] T022 [P] [US1] Write failing Catalog REST contract tests for products, search, categories, and health in `ecom-catalog-service/src/test/java/com/happiestminds/ecom/catalog/api/CatalogContractTest.java` (FR-003, FR-015, contracts/catalog.yaml)

### Catalog implementation

- [ ] T023 [P] [US1] Implement Category, Product, and ProductPrice entities with unique category name/uppercase SKU, optional parent, active flag, scale-2 INR amount, and valid-from/to pricing in `ecom-catalog-service/src/main/java/com/happiestminds/ecom/catalog/domain/` (FR-003, FR-015, data-model.md)
- [ ] T024 [P] [US1] Implement catalog repositories and request/response validation DTOs in `ecom-catalog-service/src/main/java/com/happiestminds/ecom/catalog/repository/` and `api/` (FR-003, FR-015, FR-019)
- [ ] T025 [US1] Implement current-price resolution, search filtering, and product/category administration in `ecom-catalog-service/src/main/java/com/happiestminds/ecom/catalog/service/CatalogService.java` (FR-003, FR-015)
- [ ] T026 [US1] Implement public browse/search/read and ADMIN-only create/update/category controllers in `ecom-catalog-service/src/main/java/com/happiestminds/ecom/catalog/api/` (FR-003, FR-015, FR-019)

**Checkpoint**: Identity and Catalog are independently testable and satisfy the MVP discovery/account portion before downstream services are started.

---

## Phase 4: User Story 2 - Protect Limited Stock During Concurrent Checkout (Priority: P1) — Inventory

**Goal**: Reserve all requested lines atomically, prevent overselling, and restore stock exactly once on release or 15-minute expiry.

**Independent test**: With one available laptop, 100 concurrent reservation attempts produce exactly one successful reservation, no negative stock, and understandable conflicts for losers.

### Inventory tests first

- [ ] T027 [P] [US2] Write failing optimistic-lock repository tests for all-or-nothing multi-line reservations and no negative stock in `ecom-inventory-service/src/test/java/com/happiestminds/ecom/inventory/repository/InventoryConcurrencyTest.java` (FR-007, SC-004)
- [ ] T028 [P] [US2] Write failing 100-client last-unit race integration test asserting exactly one successful reservation and zero negative stock in `ecom-inventory-service/src/test/java/com/happiestminds/ecom/inventory/integration/TwoBuyerRaceTest.java` (FR-007, SC-004)
- [ ] T029 [P] [US2] Write failing reservation confirm/release/duplicate-release and 15-minute expiry tests in `ecom-inventory-service/src/test/java/com/happiestminds/ecom/inventory/service/ReservationLifecycleTest.java` (FR-008, FR-020)
- [ ] T030 [P] [US4] Write failing ADMIN adjustment authorization, negative-adjustment rejection, and audit-reason tests in `ecom-inventory-service/src/test/java/com/happiestminds/ecom/inventory/api/InventoryAdjustmentControllerTest.java` (FR-016, FR-018, SC-009)
- [ ] T031 [P] [US2] Write failing Inventory REST contract tests for stock, adjustment, reservation, confirm, release, and read endpoints in `ecom-inventory-service/src/test/java/com/happiestminds/ecom/inventory/api/InventoryContractTest.java` (FR-007–FR-008, FR-016, contracts/inventory.yaml)

### Inventory implementation

- [ ] T032 [P] [US2] Implement InventoryStock with `availableQuantity`, `reservedQuantity`, `version`, and timestamps plus StockReservation with unique `(orderRef, productId)`, `PENDING|CONFIRMED|RELEASED|EXPIRED`, item lines, and `expiresAt` 15 minutes after creation in `ecom-inventory-service/src/main/java/com/happiestminds/ecom/inventory/domain/` (FR-007–FR-008, data-model.md)
- [ ] T033 [P] [US2] Implement InventoryAdjustment audit entity, repositories, and validated reservation/adjustment DTOs in `ecom-inventory-service/src/main/java/com/happiestminds/ecom/inventory/domain/InventoryAdjustment.java`, `repository/`, and `api/` (FR-016, FR-018)
- [ ] T034 [US2] Implement optimistic-lock all-line reservation, bounded conflict retry, confirm/release idempotency, and expiry job in `ecom-inventory-service/src/main/java/com/happiestminds/ecom/inventory/service/InventoryService.java` and `ReservationExpiryJob.java` (FR-007–FR-008, FR-020)
- [ ] T035 [US2] Implement stock read, ADMIN adjustment, reservation lifecycle controllers and explicit 409 out-of-stock responses in `ecom-inventory-service/src/main/java/com/happiestminds/ecom/inventory/api/` (FR-007–FR-008, FR-016, FR-019)

**Checkpoint**: Inventory independently proves the two-buyer race, all-or-nothing behavior, and exactly-once stock restoration.

---

## Phase 5: User Story 1 - Discover and Purchase Products (Priority: P1) — Cart & Pricing

**Goal**: Manage an active customer cart, validate discounts, and revalidate catalog/stock-sensitive totals before order checkout.

**Independent test**: A customer can add/change/remove items, apply a valid discount, see subtotal/final total, and receive a changed-total acknowledgement requirement when catalog data changes.

### Cart & Pricing tests first

- [ ] T036 [P] [US1] Write failing active-cart ownership, item quantity 1–100, unique product line, and add/change/remove tests in `ecom-cart-service/src/test/java/com/happiestminds/ecom/cart/service/CartServiceTest.java` (FR-004, FR-019, data-model.md)
- [ ] T037 [P] [US1] Write failing discount eligibility, expiration, minimum-total, incompatible-cart, duplicate-application, subtotal, and final-total tests in `ecom-cart-service/src/test/java/com/happiestminds/ecom/cart/service/PricingServiceTest.java` (FR-005, FR-019)
- [ ] T038 [P] [US1] Write failing checkout-summary revalidation tests for price, active status, currency, inventory, and acknowledgement after total changes in `ecom-cart-service/src/test/java/com/happiestminds/ecom/cart/service/CheckoutSummaryTest.java` (FR-006, FR-020)
- [ ] T039 [P] [US1] Write failing customer ownership and ADMIN denial tests for cart endpoints in `ecom-cart-service/src/test/java/com/happiestminds/ecom/cart/api/CartAuthorizationTest.java` (FR-002, FR-019)
- [ ] T040 [P] [US1] Write failing Cart REST contract tests for current cart, item mutations, discount, checkout summary, and checked-out transition in `ecom-cart-service/src/test/java/com/happiestminds/ecom/cart/api/CartContractTest.java` (FR-004–FR-006, contracts/cart.yaml)

### Cart & Pricing implementation

- [ ] T041 [P] [US1] Implement ShoppingCart and CartItem with one ACTIVE cart per customer, product snapshots, quantity 1–100, unit price, and line total in `ecom-cart-service/src/main/java/com/happiestminds/ecom/cart/domain/` (FR-004, data-model.md)
- [ ] T042 [P] [US1] Implement DiscountRule with unique code, PERCENT/FIXED type, value, minimum total, active flag, validity period, repositories, and validation DTOs in `ecom-cart-service/src/main/java/com/happiestminds/ecom/cart/domain/DiscountRule.java`, `repository/`, and `api/` (FR-005, FR-015)
- [ ] T043 [US1] Implement catalog and inventory typed clients plus cart mutation, discount, pricing, checkout-summary, and checked-out services in `ecom-cart-service/src/main/java/com/happiestminds/ecom/cart/client/` and `service/` (FR-004–FR-006, FR-020)
- [ ] T044 [US1] Implement customer-scoped cart controllers and explicit invalid-discount/changed-total responses in `ecom-cart-service/src/main/java/com/happiestminds/ecom/cart/api/` (FR-004–FR-006, FR-019)

---

## Phase 6: User Story 1 - Discover and Purchase Products (Priority: P1) — Payment

**Goal**: Simulate success, decline, and timeout outcomes with durable attempts and idempotency so payment failure never appears as a confirmed sale.

**Independent test**: Repeating a payment request with the same key returns one transaction; a decline or timeout remains visible and is safe to retry without duplicate charge.

### Payment tests first

- [ ] T045 [P] [US1] Write failing payment success, decline, timeout, attempt-history, and unsupported-outcome tests in `ecom-payment-service/src/test/java/com/happiestminds/ecom/payment/service/PaymentServiceTest.java` (FR-010, FR-019)
- [ ] T046 [P] [US1] Write failing same-key/same-hash replay and same-key/different-hash conflict tests in `ecom-payment-service/src/test/java/com/happiestminds/ecom/payment/service/PaymentIdempotencyTest.java` (FR-011, SC-005)
- [ ] T047 [P] [US1] Write failing failed-payment integration test asserting no successful charge and a visible declined/timed-out transaction in `ecom-payment-service/src/test/java/com/happiestminds/ecom/payment/integration/FailedPaymentTest.java` (FR-010–FR-011, quickstart.md)
- [ ] T048 [P] [US1] Write failing refund success, pending, failed, and duplicate-refund tests in `ecom-payment-service/src/test/java/com/happiestminds/ecom/payment/service/RefundServiceTest.java` (FR-014, SC-005)
- [ ] T049 [P] [US1] Write failing Payment REST contract tests for create/list/read/refund and idempotency headers in `ecom-payment-service/src/test/java/com/happiestminds/ecom/payment/api/PaymentContractTest.java` (FR-010–FR-014, contracts/payment.yaml)

### Payment implementation

- [ ] T050 [P] [US1] Implement PaymentTransaction, PaymentAttempt, Refund, and unique idempotency entities with `INITIATED|SUCCEEDED|DECLINED|TIMED_OUT|REFUNDED` and refund statuses in `ecom-payment-service/src/main/java/com/happiestminds/ecom/payment/domain/` (FR-010–FR-014, data-model.md)
- [ ] T051 [P] [US1] Implement payment/refund repositories, canonical request hashing, and validated request/response DTOs in `ecom-payment-service/src/main/java/com/happiestminds/ecom/payment/repository/` and `api/` (FR-010–FR-014)
- [ ] T052 [US1] Implement idempotent simulated payment attempts, explicit delay/outcome handling, and immediate/pending/failed refund rules in `ecom-payment-service/src/main/java/com/happiestminds/ecom/payment/service/PaymentService.java` and `RefundService.java` (FR-010–FR-014, FR-020)
- [ ] T053 [US4] Implement payment create/list/read/refund controllers with original-response replay and `IDEMPOTENCY_CONFLICT` responses in `ecom-payment-service/src/main/java/com/happiestminds/ecom/payment/api/` (FR-010–FR-014, FR-019)

**Checkpoint**: Cart and Payment are independently testable; failed payment leaves no completed charge and all retry outcomes are idempotent.

---

## Phase 7: User Story 3 - Manage Orders After Purchase (Priority: P1) — Order Management last

**Goal**: Orchestrate checkout as a persisted saga, expose order history, compensate payment/inventory failures, support cancellation/refund, and recover after downstream timeout.

**Independent test**: A checkout creates one order only, succeeds through reservation/payment/cart completion, or records a recoverable failure with compensation; customers can inspect and cancel eligible orders exactly once.

### Order tests first

- [ ] T054 [P] [US3] Write failing order state-transition, status-history, snapshot, and customer/admin visibility tests in `ecom-order-service/src/test/java/com/happiestminds/ecom/order/service/OrderLifecycleTest.java` (FR-009, FR-012–FR-013, FR-018)
- [ ] T055 [P] [US3] Write failing checkout idempotency replay and request-hash conflict tests in `ecom-order-service/src/test/java/com/happiestminds/ecom/order/service/OrderIdempotencyTest.java` (FR-011, SC-005)
- [ ] T056 [P] [US3] Write failing saga success test covering Cart summary, Inventory reserve/confirm, Payment success, Cart checked-out, and CONFIRMED order in `ecom-order-service/src/test/java/com/happiestminds/ecom/order/service/CheckoutSagaSuccessTest.java` (FR-006, FR-009–FR-011)
- [ ] T057 [P] [US3] Write failing failed-payment saga test asserting PAYMENT_FAILED/CANCELLED outcome, inventory release, no confirmed order, and no successful charge in `ecom-order-service/src/test/java/com/happiestminds/ecom/order/service/FailedPaymentCompensationTest.java` (FR-008, FR-010, FR-014)
- [ ] T058 [P] [US3] Write failing downstream-timeout retry/recovery test asserting persisted `COMPENSATION_PENDING`, one retryable saga step, and no duplicate reservation/payment/refund in `ecom-order-service/src/test/java/com/happiestminds/ecom/order/service/TimeoutRecoveryTest.java` (FR-011, FR-020, SC-005)
- [ ] T059 [P] [US3] Write failing concurrent two-buyer order integration test with one inventory unit asserting exactly one CONFIRMED order and one 409/no-charge result in `ecom-order-service/src/test/java/com/happiestminds/ecom/order/integration/TwoBuyerOrderRaceTest.java` (FR-007, SC-004)
- [ ] T060 [P] [US3] Write failing cancellation authorization, delivered-order rejection, one-refund, one-restock, and repeated-cancellation tests in `ecom-order-service/src/test/java/com/happiestminds/ecom/order/service/OrderCancellationTest.java` (FR-012–FR-014, SC-007)
- [ ] T061 [P] [US3] Write failing Order REST contract tests for checkout, read/history, customer list, cancel, recover, and idempotency responses in `ecom-order-service/src/test/java/com/happiestminds/ecom/order/api/OrderContractTest.java` (FR-009–FR-014, contracts/order.yaml)

### Order implementation

- [ ] T062 [P] [US3] Implement Order, OrderItem, OrderStatusHistory, and unique idempotency entities with required states, monetary snapshots, correlation ID, reservation/payment/cart references, and optimistic version in `ecom-order-service/src/main/java/com/happiestminds/ecom/order/domain/` (FR-009, FR-011–FR-014, data-model.md)
- [ ] T063 [P] [US3] Implement repositories, canonical checkout hashing, request DTOs, and response DTOs in `ecom-order-service/src/main/java/com/happiestminds/ecom/order/repository/` and `api/` (FR-009–FR-014)
- [ ] T064 [P] [US3] Implement typed Cart, Inventory, and Payment clients with propagated bearer/service token and correlation ID in `ecom-order-service/src/main/java/com/happiestminds/ecom/order/client/` (FR-006–FR-011, FR-020)
- [ ] T065 [US3] Implement persisted checkout saga state machine, compensation ordering, idempotent recovery, and timeout handling in `ecom-order-service/src/main/java/com/happiestminds/ecom/order/service/CheckoutSagaService.java` and `CompensationService.java` (FR-007–FR-011, FR-020)
- [ ] T066 [US4] Implement customer order history/read and ADMIN reconciliation queries for orders, payments, cancellations, refunds, and statuses in `ecom-order-service/src/main/java/com/happiestminds/ecom/order/service/OrderQueryService.java` (FR-012, FR-017–FR-018)
- [ ] T067 [US3] Implement cancellation-before-delivery, refund/restock compensation, and duplicate-action protection in `ecom-order-service/src/main/java/com/happiestminds/ecom/order/service/CancellationService.java` (FR-013–FR-014, SC-007)
- [ ] T068 [US3] Implement order checkout, read/history, cancel, and recover controllers with clear 409/503 next-step responses in `ecom-order-service/src/main/java/com/happiestminds/ecom/order/api/` (FR-009–FR-014, FR-019)

**Checkpoint**: All four user stories are independently testable, with Order Management completing the required final integration without shared databases or distributed transactions.

---

## Phase 8: Polish & Cross-Cutting Concerns

**Purpose**: Validate the full platform against the quickstart, measurable outcomes, security rules, and operational diagnostics.

- [ ] T069 [P] Update each service README and API documentation with endpoint ownership, authentication/role requirements, idempotency behavior, failure states, and local run commands in `ecom-identity-service/README.md`, `ecom-catalog-service/README.md`, `ecom-inventory-service/README.md`, `ecom-cart-service/README.md`, `ecom-order-service/README.md`, and `ecom-payment-service/README.md` (FR-001–FR-020)
- [ ] T070 [P] Add cross-service authorization negative tests proving another customer cannot read orders/payment/refunds and non-ADMIN users cannot mutate catalog/stock/transactions in `ecom-order-service/src/test/java/com/happiestminds/ecom/order/integration/AuthorizationBoundaryTest.java` and `ecom-payment-service/src/test/java/com/happiestminds/ecom/payment/integration/AuthorizationBoundaryTest.java` (FR-002, FR-015–FR-017, SC-009)
- [ ] T071 [P] Add audit completeness assertions for stock, order, payment, cancellation, refund, discount, and admin changes in `ecom-order-service/src/test/java/com/happiestminds/ecom/order/integration/AuditTrailTest.java` and the owning services' audit tests (FR-018, SC-010)
- [ ] T072 [P] Add performance and acceptance tests for product-search p95 under 2 seconds, successful checkout p95 under 3 minutes, and 1-second connect/2-second read timeout behavior in `ecom-catalog-service/src/test/java/com/happiestminds/ecom/catalog/integration/PerformanceAcceptanceTest.java` and `ecom-order-service/src/test/java/com/happiestminds/ecom/order/integration/PerformanceAcceptanceTest.java` (SC-002–SC-003)
- [ ] T073 Run the complete six-service quickstart, health checks, end-to-end smoke test, failed-payment scenario, timeout retry, 15-minute expiry, refund retry, and two-buyer race from `specs/001-unified-commerce-backend/quickstart.md`; record reproducible commands and expected evidence in `docs/acceptance-criteria.md` (SC-001–SC-010)
- [ ] T074 Run independent Maven test suites for all six services and verify no service reads another service's database, no secret/token/password is logged, and all production changes have corresponding tests in each service's `pom.xml`, `src/test/`, and `src/main/` (Constitution I, III, V)

---

## Dependencies & Execution Order

### Phase dependencies

- **Setup (Phase 1)**: No dependencies; T002–T004 can run in parallel after T001 establishes directories.
- **Foundational (Phase 2)**: Depends on Phase 1; T005–T010 can run in parallel and block all user stories.
- **Identity and Catalog (Phase 3)**: Depends on Phase 2; Identity tasks T011–T018 and Catalog tests T019–T022 can run in parallel, while each service follows tests before implementation.
- **Inventory (Phase 4)**: Starts after Phase 3; T027–T035 depend on Identity/Catalog contracts for authenticated product references.
- **Cart & Pricing and Payment (Phases 5–6)**: Start after Inventory; all Cart tests/implementation and Payment tests/implementation can run in parallel with each other, but each service remains test-first.
- **Order Management (Phase 7)**: Starts only after Cart, Inventory, and Payment contracts are implemented and tested; T054–T068 are the final service work.
- **Polish (Phase 8)**: Depends on all service checkpoints.

### Requirement-to-story map

- **US1 / FR-001–FR-006, FR-009–FR-011**: Identity, Catalog, Cart & Pricing, Payment, and checkout integration.
- **US2 / FR-007–FR-008**: Inventory consistency, reservation expiry, and Order race integration.
- **US3 / FR-012–FR-014**: Order query/cancellation plus Payment refund and Inventory compensation.
- **US4 / FR-015–FR-018**: Catalog and Inventory administration, Payment/Order reconciliation, and audit history.
- **FR-019–FR-020**: Foundational error/correlation behavior and all service retry/failure tests.

### Parallel opportunities

- **Identity + Catalog**: T011–T014 and T019–T022 are parallel test tracks; T015–T018 and T023–T026 are parallel service implementation tracks after their own tests fail.
- **Cart & Pricing + Payment**: T036–T044 and T045–T053 can run in parallel after Inventory.
- **Independent test files**: All `[P]` tests in a phase can be authored concurrently because they target separate files and do not depend on production implementation.
- **Order integration**: T056–T061 can run concurrently as failing specifications, then T062–T068 implement the shared saga model/services in dependency order.

### Required failure and race coverage

- **Two-buyer race**: T028 proves Inventory's 100-client last-unit invariant; T059 proves the end-to-end Order result.
- **Failed payment**: T047 proves Payment outcomes; T057 proves Order compensation and absence of a confirmed sale.
- **Timeout retry**: T058 proves persisted recovery and no duplicate effects for interrupted downstream calls.

---

## Implementation Strategy

### MVP first

1. Complete Setup and Foundational phases.
2. Build and validate Identity and Catalog.
3. Build Inventory and validate the last-unit race.
4. Build Cart & Pricing and Payment, including failed-payment behavior.
5. Build Order Management last and validate the complete customer purchase flow.
6. Stop and demo User Story 1 after T068 plus its required race/failure/retry tests pass.

### Incremental delivery

1. Deliver Identity + Catalog as the authenticated discovery baseline.
2. Add Inventory as a separately validated oversell-prevention increment.
3. Add Cart & Pricing and Payment in parallel, preserving typed REST contracts.
4. Add Order Management as the sole saga orchestrator and validate compensation/recovery.
5. Finish cross-cutting acceptance, audit, security, and quickstart evidence.
