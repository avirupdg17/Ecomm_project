---

applyTo: "**/src/test/**"

---

- JUnit 5 + AssertJ. Test class name = <ClassUnderTest>Test for unit tests,

 <Feature>IT for Spring Boot integration tests (run by Failsafe in the verify phase).

- Method names describe behaviour: shouldRejectDuplicateSku_whenSkuExists().

- Tests that prove an acceptance criterion or business rule start their @DisplayName with

 its IDs, e.g. @DisplayName("AC-INV-03 R-INV-01 rejects reservation above available stock").

 The CI traceability report and docs/traceability-matrix.md read these IDs.

- Arrange-Act-Assert with blank lines between sections. One behaviour per test.

- Unit tests mock repositories and clients with Mockito; no Spring context.

- Integration tests use @SpringBootTest + @AutoConfigureMockMvc with the H2 in-memory

 profile "test" (application-test.yml). Outbound clients are replaced with @MockitoBean (org.springframework.test.context.bean.override.mockito; @MockBean is deprecated since Spring Boot 3.4).

- Every endpoint gets: happy path, validation failure (400), unauthenticated (401),

 wrong role (403), not found (404), and invalid state (409) where applicable.

- Concurrency tests use an ExecutorService with a CountDownLatch and assert exact counts.

- Tests never call the network, never sleep more than 100 ms, and never depend on order.

- Never disable a failing test. Fix the code or the test with justification.

File: docs/business-rules.md (extract; complete it on Day 1)

 

# Business rules

Each rule has an ID, a statement, the error code returned when violated, and the HTTP status.

 

## Identity

R-IDN-01 Email is unique (case-insensitive). EMAIL_ALREADY_EXISTS, 409.

R-IDN-02 Password minimum 8 characters, stored only as BCrypt hash. VALIDATION_ERROR, 400.

R-IDN-03 A user can read or update only their own profile unless ADMIN. FORBIDDEN, 403.

R-IDN-04 New registrations receive role CUSTOMER only.

 

## Catalog

R-CAT-01 SKU is unique, upper-case, pattern [A-Z]{3}-[0-9]{3}. SKU_ALREADY_EXISTS, 409.

R-CAT-02 Price amount > 0 with at most 2 decimals; currency INR. INVALID_PRICE, 400.

R-CAT-03 Only ADMIN creates or updates products and categories. FORBIDDEN, 403.

R-CAT-04 Search is case-insensitive on name; pagination max size 100.

 

## Inventory

R-INV-01 availableQuantity and reservedQuantity are never negative. INSUFFICIENT_STOCK, 409.

R-INV-02 Reserve, confirm and release are atomic per product and protected by @Version.

R-INV-03 A reservation request with an existing orderRef returns the existing reservation.

R-INV-04 confirm allowed from PENDING or CONFIRMED (idempotent); release allowed from

        PENDING, RELEASED or EXPIRED (idempotent: RELEASED and EXPIRED return 200 unchanged);

        confirm from RELEASED/EXPIRED or release from CONFIRMED: INVALID_RESERVATION_STATE, 409.

R-INV-05 PENDING reservations expire after INVENTORY_RESERVATION_TTL_MINUTES (default 15).

R-INV-06 Adjustments by ADMIN or SERVICE only; a repeated adjustment with the same reference

        returns the original result and does not change stock again.

R-INV-07 Reservation create/confirm/release accept SERVICE and ADMIN only. FORBIDDEN, 403.

 

## Cart and pricing

R-CART-01 Quantity per item 1..100. INVALID_QUANTITY, 400.

R-CART-02 Only active products can be added. PRODUCT_NOT_FOUND, 404.

R-CART-03 unitPrice is re-read from Catalog at checkout; client totals are ignored.

R-CART-04 PERCENT discount = subtotal x value / 100 (HALF_UP, 2 dp). FIXED = min(value, subtotal).

R-CART-05 A discount applies only if active, within validity, and subtotal >= minimumOrderTotal.

          DISCOUNT_NOT_APPLICABLE, 422.

R-CART-06 total = subtotal - discountAmount, never below 0.00.

 

## Orders

R-ORD-01 Idempotency-Key header is required; same key returns the existing order (200);

        same key + different request body (hash): IDEMPOTENCY_CONFLICT, 422.

R-ORD-02 Legal transitions are only those in docs/architecture.md state table.

         INVALID_ORDER_STATE, 409.

R-ORD-03 An order is CONFIRMED only after payment SUCCEEDED and reservation CONFIRMED.

R-ORD-04 Payment DECLINED -> release reservation -> CANCELLED; release failure ->

         COMPENSATION_PENDING (recoverable).

R-ORD-05 A payment timeout is reconciled (query by orderId / same key) before any decision.

R-ORD-06 Cancellation allowed from CONFIRMED (refund + restock), PAYMENT_FAILED,

         COMPENSATION_PENDING.

R-ORD-07 Inventory unreachable during reservation: order CANCELLED (INVENTORY_UNAVAILABLE), 503.

R-ORD-08 Payment SUCCEEDED but reservation EXPIRED/RELEASED: refund, then CANCELLED

         (RESERVATION_LOST); never CONFIRMED without stock.

R-ORD-09 Marking the cart checked-out is best effort and never fails a CONFIRMED order.

 

## Payments

R-PAY-01 Idempotency-Key required; same key + same requestHash returns original (200);

        same key + different hash: IDEMPOTENCY_CONFLICT, 422.

R-PAY-02 Outcome: simulatedOutcome wins; else amount ending .99 DECLINED, .98 TIMEOUT

        behaviour, else SUCCEEDED.

R-PAY-03 Refund only from SUCCEEDED; a repeated refund returns the existing refund (look it up

        before the state check). INVALID_PAYMENT_STATE, 409.

R-PAY-04 No real payment provider is ever called.

R-PAY-05 Create and refund accept SERVICE and ADMIN only; a CUSTOMER may read only payments

        whose customerId equals the token subject. FORBIDDEN, 403.

 

Message from Harsha Rao Create .github/copilot-instructions.md and .github/instructions/*.instructions.md into each sibling repository ../*ecom-<service>-service for identity, catalog, inventory, cart, order and payment. In each copy replace cart with the service name and 8084 with 8081, 8082, 8083, 8084, 8085 and 8086 respectively. Use one short shell loop with mkdir -p, cp and sed. Show me the command before running it. Change nothing else. Create docs/engineering-standards.md with the sections Conventions, Approved dependencies, Approved AI model and CI. Fill Conventions and Approved dependencies verbatim from the pasted Reference C and Reference J text. Under Approved AI model write: GPT-5.4 mini. Under CI write one line: Builds and tests run on GitHub Actions (ci.yml, step 33). Do not add rules that are not in the doc.
