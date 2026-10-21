# Quickstart Validation Guide

## Prerequisites

- Java 21 and Maven 3.9+.
- Six service projects under the repository root.
- A writable repository directory for each service's file-based H2 database.
- Non-default `JWT_SECRET` and `SEED_ADMIN_PASSWORD` environment variables for
  non-test runs.

## Build and run

From each service directory, run:

```bash
mvn test
mvn spring-boot:run
```

Start services on ports 8081 through 8086: Identity, Catalog, Inventory, Cart
& Pricing, Order Management, and Payment. Confirm
`GET http://localhost:<port>/actuator/health` returns `{"status":"UP"}`.

## End-to-end smoke test

1. Register a customer through Identity and log in for a bearer token.
2. Search Catalog for an active product and inspect its current price.
3. Add the product to Cart and request a checkout summary.
4. Submit Order checkout with a UUID `Idempotency-Key`.
5. Verify Inventory reservation, Payment transaction, confirmed Order, and
   checked-out Cart through their documented APIs.
6. Repeat the identical Order request with the same key; verify the original
   order is returned without a second payment or reservation.

Contract summaries are in [`contracts/`](./contracts/), and entity/state rules
are in [`data-model.md`](./data-model.md).

## Negative and concurrency checks

- Reuse an idempotency key with a changed body; expect
  `IDEMPOTENCY_CONFLICT` and no new order/payment.
- Seed one available unit and submit two checkouts concurrently; expect exactly
  one confirmed order, one out-of-stock result, and no negative stock.
- Simulate payment decline; expect a persisted failed/cancelled order, released
  reservation, no successful charge, and a recoverable compensation state after
  a downstream timeout.
- Advance the expiry clock or run the expiry job; verify an unpaid reservation
  becomes `EXPIRED` after 15 minutes and stock is restored once.
- Cancel an undelivered confirmed order; verify one refund and one restock.
  Repeat cancellation/refund and verify no duplicate effects.
- Attempt another customer's order or an ADMIN-only endpoint with the wrong
  role; expect denial and unchanged data.

## Acceptance evidence

Capture API responses, correlation IDs, service logs, and database assertions
for these scenarios. Measurable targets are in
[`../../docs/acceptance-criteria.md`](../../docs/acceptance-criteria.md).
