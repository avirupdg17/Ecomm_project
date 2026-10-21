# API contracts and seed data

## Identity and User Profile Service (port 8081)

### Identity API contract

| Endpoint | Auth | Request → Response |
|---|---|---|
| POST /api/users/register | none | {email, password, fullName} → 201 {id, email, roles:["CUSTOMER"]}; 409 EMAIL_ALREADY_EXISTS; 400 on invalid email or password shorter than 8 characters |
| POST /api/auth/login | none | {email, password} → 200 {accessToken, tokenType:"Bearer", expiresInSeconds, userId, roles}; 401 INVALID_CREDENTIALS |
| GET /api/users/me | CUSTOMER / ADMIN | → 200 {id, email, roles, profile:{fullName, phone, addressLine1, city, postcode}} |
| PUT /api/users/me | CUSTOMER / ADMIN | {fullName, phone, addressLine1, city, postcode} → 200 updated profile; 400 on validation failure |
| GET /api/users/{id} | ADMIN | → 200 user (administrative lookup); 404 USER_NOT_FOUND |
| GET /actuator/health | none | → 200 {status:"UP"} |

## Product Catalog Service (port 8082)

### Catalog API contract

| Endpoint | Auth | Request → Response |
|---|---|---|
| POST /api/categories | ADMIN | {name, description, parentId?} → 201 category; 409 CATEGORY_ALREADY_EXISTS |
| GET /api/categories | none | → 200 list |
| POST /api/products | ADMIN | {sku, name, description, categoryId, price:{amount, currency}, active} → 201 product with current price; 409 SKU_ALREADY_EXISTS; 400 INVALID_PRICE when amount ≤ 0 or more than 2 decimals |
| GET /api/products | none | query page (0-based), size (default 20, max 100), sort (e.g. name,asc) → 200 {content:[...], page, size, totalElements, totalPages} |
| GET /api/products/{id} | none | → 200 product; 404 PRODUCT_NOT_FOUND |
| PUT /api/products/{id} | ADMIN | same body as create (SKU immutable) → 200; 404; 409 |
| GET /api/products/search | none | query q (name contains, case-insensitive), categoryId, minPrice, maxPrice, active, plus pagination → 200 page |
| GET /api/products/by-sku/{sku} | none | → 200 product (used by Cart to validate SKUs) |

## Inventory Service (port 8083)

### Inventory API contract

| Endpoint | Auth | Request → Response |
|---|---|---|
| GET /api/inventory/{productId} | any authenticated | → 200 {productId, availableQuantity, reservedQuantity, version}; 404 STOCK_NOT_FOUND |
| POST /api/inventory/{productId}/adjustments | ADMIN / SERVICE | {delta, reason, reference?} → 200 new stock (creates the stock row if missing); a repeated request with the same reference returns the original result without applying the delta again; 409 NEGATIVE_STOCK if result would be below zero |
| POST /api/inventory/reservations | SERVICE / ADMIN | {orderRef, items:[{productId, quantity}]} → 201 {reservationId, status:"PENDING", expiresAt, items}; 409 INSUFFICIENT_STOCK with the offending productId in details; a repeated request with the same orderRef returns the existing reservation (200) |
| POST /api/inventory/reservations/{id}/confirm | SERVICE / ADMIN | → 200 {reservationId, status:"CONFIRMED"}; idempotent (confirming a CONFIRMED reservation returns 200 unchanged); 409 INVALID_RESERVATION_STATE if RELEASED or EXPIRED |
| POST /api/inventory/reservations/{id}/release | SERVICE / ADMIN | → 200 {reservationId, status:"RELEASED"}; idempotent (releasing twice does not add stock twice); releasing an EXPIRED reservation also returns 200 unchanged (its stock was already returned by the expiry job); 409 if CONFIRMED |
| GET /api/inventory/reservations/{id} | any authenticated | → 200 reservation; 404 |

## Shopping Cart and Pricing Service (port 8084)

### Cart and Pricing API contract

| Endpoint | Auth | Request → Response |
|---|---|---|
| GET /api/carts/current | CUSTOMER | → 200 the customer's ACTIVE cart (created on first access): {id, customerId, items:[...], subtotal, discountCode, discountAmount, total, currency} |
| POST /api/carts/items | CUSTOMER | {productId, quantity} → 201 cart; validates product via Catalog (active must be true); 400 INVALID_QUANTITY for quantity < 1 or > 100; 404 PRODUCT_NOT_FOUND; adding an existing product increases its quantity |
| PUT /api/carts/items/{itemId} | CUSTOMER | {quantity} → 200 cart; 400 on invalid quantity |
| DELETE /api/carts/items/{itemId} | CUSTOMER | → 204 |
| POST /api/carts/discount | CUSTOMER | {code} → 200 cart with discount applied; 404 DISCOUNT_NOT_FOUND; 422 DISCOUNT_NOT_APPLICABLE (inactive, expired, below minimum total) |
| POST /api/carts/checkout-summary | CUSTOMER / SERVICE | optional {customerId} when called by SERVICE → 200 {cartId, customerId, items:[{productId, sku, productName, quantity, unitPrice, lineTotal}], subtotal, discountCode, discountAmount, total, currency} with prices re-validated against Catalog; 422 EMPTY_CART |
| POST /api/carts/{cartId}/checked-out | SERVICE | → 200 marks the cart CHECKED_OUT after order confirmation (a new ACTIVE cart is created on next access) |
| POST /api/discounts / GET /api/discounts | ADMIN | manage DiscountRule rows |

## Order Management Service (port 8085)

### Order API contract

| Endpoint | Auth | Request → Response |
|---|---|---|
| POST /api/orders | CUSTOMER | header Idempotency-Key (required, UUID) and optional body {simulatedPaymentOutcome: "SUCCESS" or "DECLINED" or "TIMEOUT"} → 201 {id, status, items, subtotal, discountAmount, totalAmount, currency, reservationRef, paymentRef, createdAt}; 200 with the existing order when the same key is repeated; 422 IDEMPOTENCY_CONFLICT when the same key is reused with a different request body; a declined payment still returns 201 with status: CANCELLED (or COMPENSATION_PENDING) and failureReason, because the order resource exists for audit; 409 INSUFFICIENT_STOCK; 422 EMPTY_CART; 503 DEPENDENCY_UNAVAILABLE when a downstream service cannot be reached (order left in a recoverable state, see Reference F (Goto Reference Section in this doc to view)) |
| GET /api/orders/{id} | owner or ADMIN | → 200 order with statusHistory; 403 if another customer's order; 404 |
| GET /api/orders/customer/{customerId} | owner or ADMIN | → 200 paginated list, newest first |
| POST /api/orders/{id}/cancel | owner or ADMIN | → 200 order CANCELLED (refund + stock restored through an idempotent adjustment with reference cancel-<orderId> if it was CONFIRMED); 409 INVALID_ORDER_STATE for states that cannot be cancelled |
| POST /api/orders/{id}/recover | ADMIN / SERVICE | → 200 re-runs the pending step for orders in PAYMENT_PENDING or COMPENSATION_PENDING, and re-sends the cart checked-out call for CONFIRMED orders whose cartCheckedOut is false (safe to call repeatedly) |

## Payment Simulation Service (port 8086)

### Payment API contract

| Endpoint | Auth | Request → Response |
|---|---|---|
| POST /api/payments | SERVICE / ADMIN | header Idempotency-Key (required) and body {orderId, customerId, amount, currency, simulatedOutcome?} → 201 {id, orderId, amount, currency, status, failureReason, createdAt}; same key + same request → 200 original transaction; same key + different request → 422 IDEMPOTENCY_CONFLICT |
| GET /api/payments/{id} | CUSTOMER (own payments only) / SERVICE / ADMIN | → 200 transaction with attempts |
| GET /api/payments?orderId= | CUSTOMER (own payments only) / SERVICE / ADMIN | → 200 list of transactions for the order (used for reconciliation) |
| POST /api/payments/{id}/refund | SERVICE / ADMIN | {amount?, reason} → 201 {refundId, status:"COMPLETED"}; 409 INVALID_PAYMENT_STATE unless status is SUCCEEDED; repeat refund of the same payment returns the existing refund (the existing refund is looked up before the state check, because the payment is REFUNDED by then) |

## Deterministic seed data (all services)

| Service | Seed data |
|---|---|
| Identity | Admin user admin@ecom.test with password from SEED_ADMIN_PASSWORD (default Admin#12345), roles ADMIN + CUSTOMER; roles CUSTOMER and ADMIN |
| Catalog | Categories Laptops, Accessories; products with fixed IDs so that other services can seed against them: LAP-001 "ProBook 14" 55,000.00 INR, LAP-002 "UltraSlim 13" 72,999.99 INR (decline-trigger price), ACC-001 "USB-C Hub" 1,499.00 INR, ACC-002 "Wireless Mouse" 899.00 INR |
| Inventory | Stock for LAP-001 = 5, LAP-002 = 1 (the "last laptop"), ACC-001 = 50, ACC-002 = 100, seeded against the same fixed product IDs as Catalog (11111111-0000-0000-0000-000000000001 to …0004 in the order LAP-001, LAP-002, ACC-001, ACC-002). No start-up call to Catalog is needed. The integration scripts create their own products and stock through the API, so they never depend on this seed. |
| Cart | Discount rules WELCOME10 and FLAT200 as defined in Reference E – Shopping Cart and Pricing Service (Goto Reference Section in this doc to view) |
| Order, Payment | No seed data |

## Standard error body

```json
{
  "timestamp": "2026-10-20T09:15:30.123Z",
  "status": 409,
  "error": "Conflict",
  "code": "INSUFFICIENT_STOCK",
  "message": "Requested quantity exceeds available stock",
  "path": "/api/inventory/reservations",
  "correlationId": "7f3b1e2c-...",
  "details": [
    {
      "field": "items[0].productId",
      "issue": "available=1, requested=2"
    }
  ]
}
```
