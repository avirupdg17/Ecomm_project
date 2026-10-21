# Business requirements

## Business objectives

- Deliver a secure, reliable microservices-based commerce backend that lets customers browse products, manage carts, place orders, make simulated payments, and track order progress.
- Keep catalog, inventory, order, and payment workflows consistent under concurrency, retries, failures, and timeouts.
- Support administrators managing products, categories, inventory, and transactional outcomes without breaking service boundaries or traceability.

## Actors

- Customer
- Administrator
- Inventory Service
- Order Service
- Payment Service

## Functional requirements (by service)

### Identity

- US-IDN-01 As a customer, I want to register and authenticate so that I can create an account and access the platform securely.
  - AC-IDN-01 Given I submit a valid registration request, when the request is processed, then the API returns 201 Created.
  - AC-IDN-02 Given I submit valid login credentials for an existing account, when authentication succeeds, then the API returns 200 OK.
  - AC-IDN-03 Given I submit invalid credentials, when authentication is rejected, then the API returns 401 Unauthorized.
- US-IDN-02 As a customer, I want to view and update my profile so that my account details remain current.
  - AC-IDN-04 Given I request my profile with valid authentication, when the profile is found, then the API returns 200 OK.
  - AC-IDN-05 Given I submit a valid profile update, when the update is applied, then the API returns 200 OK.
  - AC-IDN-06 Given I request a profile that does not exist, when it cannot be found, then the API returns 404 Not Found.

### Catalog

- US-CAT-01 As a customer, I want to browse, search, and filter the product catalog so that I can find products that match my needs.
  - AC-CAT-01 Given the catalog contains products, when I request the catalog or search results, then the API returns 200 OK.
  - AC-CAT-02 Given I provide search terms or filters, when matching products exist, then the API returns 200 OK with only matching products.
  - AC-CAT-03 Given no products match my search or filters, when the request is processed, then the API returns 200 OK with an empty result set.
- US-CAT-02 As an administrator, I want to manage products and categories so that the catalog stays accurate.
  - AC-CAT-04 Given I submit a valid product or category create request, when it is processed, then the API returns 201 Created.
  - AC-CAT-05 Given I submit a valid product or category update request, when it is processed, then the API returns 200 OK.
  - AC-CAT-06 Given I request a product or category that does not exist, when it cannot be found, then the API returns 404 Not Found.

### Inventory

- US-INV-01 As a customer, I want stock to be reserved when I place an order so that I know the item is held for me.
  - AC-INV-01 Given sufficient stock is available, when I request a reservation, then the API returns 200 OK.
  - AC-INV-02 Given the final unit is being reserved concurrently, when only one reservation can succeed, then the successful request returns 200 OK and the conflicting request returns 409 Conflict.
  - AC-INV-03 Given stock is insufficient, when I request a reservation, then the API returns 409 Conflict.
- US-INV-02 As a customer, I want reserved stock to be released or updated after the order outcome is known so that availability stays accurate.
  - AC-INV-04 Given a payment fails, when the order is compensated, then the reservation is released and the API returns 200 OK.
  - AC-INV-05 Given a payment succeeds, when the order is completed, then inventory is updated and the API returns 200 OK.
  - AC-INV-06 Given the inventory record cannot be found, when an update is requested, then the API returns 404 Not Found.
- US-INV-03 As a service client, I want reservation and release endpoints so that downstream services can manage stock directly.
  - AC-INV-07 Given a service client submits a valid reservation request, when stock is available, then the API returns 200 OK.
  - AC-INV-08 Given a service client requests reservation release or restock after compensation or cancellation, when the request is processed, then the API returns 200 OK.

### Cart and Pricing

- US-CART-01 As a customer, I want to add, update, and remove items in my shopping cart so that I can prepare my order.
  - AC-CART-01 Given I add a valid product to my cart, when the request is processed, then the API returns 200 OK.
  - AC-CART-02 Given I update an existing cart line or remove an item, when the request is processed, then the API returns 200 OK.
  - AC-CART-03 Given I request a cart that does not exist, when it cannot be found, then the API returns 404 Not Found.
- US-CART-02 As a customer, I want prices and discounts to be calculated for my cart so that I can see the amount I will pay.
  - AC-CART-04 Given items and eligible discounts are present in the cart, when pricing is calculated, then the API returns 200 OK with the calculated total.
  - AC-CART-05 Given a cart has no eligible discounts, when pricing is calculated, then the API returns 200 OK with the undiscounted total.
  - AC-CART-06 Given the cart cannot be priced because required cart data is missing, when the request is processed, then the API returns 400 Bad Request.

### Order

- US-ORD-01 As a customer, I want to create an order and track its state transitions so that I know my purchase progress.
  - AC-ORD-01 Given a valid cart and reserved stock, when I create an order, then the API returns 201 Created.
  - AC-ORD-02 Given an order exists, when its state changes as part of processing, then the API returns 200 OK.
  - AC-ORD-03 Given I request an order that does not exist, when it cannot be found, then the API returns 404 Not Found.
- US-ORD-02 As a customer, I want to cancel an order and receive a simulated refund so that I can reverse an unwanted purchase.
  - AC-ORD-04 Given a cancellable order exists, when I cancel it, then the API returns 200 OK.
  - AC-ORD-05 Given a cancelled order triggers a simulated refund, when the refund is processed, then the API returns 200 OK.
  - AC-ORD-06 Given an order is already completed or cancelled, when I request cancellation again, then the API returns 409 Conflict.
- US-ORD-03 As a customer, I want payment failures to move my order to a failed state and release the reservation so that I am not charged for an unsuccessful order.
  - AC-ORD-07 Given payment processing fails, when the order is updated, then the API returns 200 OK.
  - AC-ORD-08 Given payment fails, when compensation is applied, then the order is not marked CONFIRMED, enters COMPENSATION_PENDING, and the reservation is released.
  - AC-ORD-09 Given an order is already in a terminal failure state, when the failure transition is requested again, then the API returns 409 Conflict.
- US-ORD-04 As a customer, I want order retries and payment timeout reconciliation to return the original result so that I do not get duplicate outcomes.
  - AC-ORD-10 Given an order request is retried after a timeout with the same idempotency key, when the original transaction exists, then the API returns the original transaction instead of creating a duplicate order or payment.
  - AC-ORD-11 Given a cancelled order triggers a simulated refund and inventory restock, when the cancellation is processed, then the API returns 200 OK.

### Payment

- US-PAY-01 As a customer, I want to make a simulated payment for my order so that I can complete checkout.
  - AC-PAY-01 Given a valid payment request, when the payment is approved, then the API returns 200 OK.
  - AC-PAY-02 Given a payment request fails, when the payment is declined, then the API returns 402 Payment Required.
  - AC-PAY-03 Given a payment request cannot be processed because the order is unknown, when the request is processed, then the API returns 404 Not Found.
- US-PAY-02 As a service client, I want payment requests to be idempotent so that retries after timeouts do not double-charge.
  - AC-PAY-04 Given a payment request is retried after a timeout with the same idempotency key, when the original payment exists, then the API returns the original transaction instead of charging twice.

## Non-functional requirements

### Security

- NFR-SEC-01 The API uses stateless HS256 JWT authentication signed with JWT_SECRET and supports CUSTOMER, ADMIN, and SERVICE roles.
- NFR-SEC-02 Administrative endpoints enforce role-based authorization with @PreAuthorize("hasRole('ADMIN')").
- NFR-SEC-03 Public access is limited to /actuator/health and /actuator/info.
- NFR-SEC-04 Passwords are hashed with BCryptPasswordEncoder and never stored in plaintext.
- NFR-SEC-05 All request DTOs are validated with Bean Validation and controllers use @Validated.
- NFR-SEC-06 Product/category writes, inventory adjustments, discount management, and user lookup are protected administrative endpoints.
- NFR-SEC-07 Orders, carts, profiles, and payments validate customer ownership by comparing customerId to the token subject unless the caller is ADMIN or SERVICE.
- NFR-SEC-08 The application returns safe exception responses with a standard error body and no stack traces, SQL, or internal class names.
- NFR-SEC-09 passwordHash is never serialized, logs never print tokens or passwords, and Actuator exposes only health and info.
- NFR-SEC-10 Secrets such as JWT_SECRET, SEED_ADMIN_PASSWORD, and service URLs are loaded from environment variables and GitHub Actions secrets.
- NFR-SEC-11 Important operations retain audit information including InventoryAdjustment.actor, OrderStatusHistory, PaymentAttempt, and createdAt/updatedAt timestamps.
- NFR-SEC-12 Service-to-service calls use signed JWTs with a SERVICE subject, and only SERVICE or ADMIN can drive reservation and payment write endpoints.

### Reliability

- NFR-REL-01 The Inventory Service protects against overselling with optimistic locking (@Version) plus a single retry on OptimisticLockException.
- NFR-REL-02 Order creation and Payment operations handle repeated requests safely: the same idempotency key never produces a repeated logical transaction.
- NFR-REL-03 Every RestClient has a 1000 ms connect timeout and a 2000 ms read timeout so a hanging dependency never blocks the caller indefinitely.
- NFR-REL-04 Idempotent GET, confirm, and release calls retry at most twice with 500 ms backoff, and the initial payment POST is never retried.
- NFR-REL-05 Downstream failures produce a standard error body with machine-readable codes, including 503 DEPENDENCY_UNAVAILABLE.
- NFR-REL-06 Every order, reservation, and payment state is written to H2 before the next remote call.
- NFR-REL-07 Compensation releases on payment failure, refunds and restocks on cancellation, and uses COMPENSATION_PENDING when compensation itself fails.
- NFR-REL-08 POST /api/orders/{id}/recover and the scheduled recovery job resume from the persisted state.

### Observability

- NFR-OBS-01 Actuator exposes /actuator/health and /actuator/info.
- NFR-OBS-02 Application logs use a JSON-like single-line pattern with ts, level, service, correlationId, logger, and message, and log INFO for state changes, WARN for compensation, and ERROR for failed recovery.
- NFR-OBS-03 CorrelationIdFilter and MDC populate correlation IDs, and RestClient interceptors forward X-Correlation-Id.
- NFR-OBS-04 Standard error bodies include correlationId so a client error can be matched to server logs.
- NFR-OBS-05 GET /api/orders/{id} includes statusHistory, and GET /api/payments/{id} includes attempts.
- NFR-OBS-06 Surefire/Failsafe XML and the GitHub Actions job summary are uploaded as artifacts.
- NFR-OBS-07 Integration workflow uploads every service log file on failure.

## Out of scope

- Wish lists.
- Stacked coupons or coupon stacking.
- Real card numbers or stored card data.
- Any capability not explicitly covered by the functional requirements or References E/O.

## Open questions

- Ambiguity – Should compensation expose a visible interim state? – Resolution – Yes; use COMPENSATION_PENDING while the failed payment is being reconciled. – Source – Reference E and Reference O.
- Ambiguity – Should cancellation also restore inventory? – Resolution – Yes; cancellation must trigger both a simulated refund and a stock restock. – Source – Reference O.
- Ambiguity – What should happen when an order or payment request times out and is retried? – Resolution – Return the original transaction by idempotency key and do not charge or create a duplicate. – Source – Reference E and Reference O.
- Ambiguity – Are reservation and payment flows exposed only through service-to-service endpoints? – Resolution – Model them as service endpoints that downstream services can call directly. – Source – Reference E.

## Source

Functional requirements were shared by the client, gathered and analysed by our business analyst.

Source: "business_analyst_raw_requirements.doc"

### Functional outcomes

The finished application must support all of the following. Each one maps to a service and to an end-to-end scenario in Reference O (Goto Reference Section in this doc to view).

- Customer registration and authentication.
- User profile management.
- Product catalog and category management.
- Product search and filtering.
- Stock availability and inventory reservations.
- Shopping cart management.
- Pricing and discount calculation.
- Order creation and order-state transitions.
- Simulated payment processing.
- Payment failure and compensation.
- Order cancellation and simulated refunds.
- Inventory updates following successful transactions.
- Administrative operations.
- Operational logging and diagnostics.

### Enterprise retail commerce platform

A retail organisation currently manages product availability, customer information and order processing through disconnected systems. As order volumes increase, the organisation experiences inventory inconsistencies, duplicate checkout requests, unsuccessful payments and difficulty coordinating changes across backend components.

The business requires a modern microservices-based e-commerce backend. The new platform must allow customers to browse products, maintain shopping carts, place orders, make simulated payments and track their orders. Administrators must manage products, inventory and relevant transactional information. The architecture must provide reliable service integration while remaining maintainable, secure and adaptable to changing requirements.

### The central engineering challenge

Diagram: see Figure 3 in the Appendix – Diagrams and flows.

A customer wants to purchase the final available laptop in inventory. At the same time, another customer attempts to purchase the same product. The system must ensure that only one customer receives a valid reservation. If the selected customer's payment fails, the inventory must become available again. If the payment succeeds but the Order Service encounters a timeout, the system must avoid creating an incorrect second payment or losing the confirmed payment.

This scenario drives three design decisions you will implement:

- Optimistic locking in the Inventory Service (a @Version column and an atomic update) so that concurrent reservations cannot oversell.
- Compensation in the Order Service so that a failed payment releases the reservation and the order moves to a failure state, never to CONFIRMED.
- Idempotency keys in the Order and Payment Services so that a retried request after a timeout returns the original transaction instead of charging twice.
