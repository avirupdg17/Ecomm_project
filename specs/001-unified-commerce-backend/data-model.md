# Data Model: Unified Commerce Backend

Each section belongs to the named service. Cross-service references are
logical IDs only; no entity, repository, or database is shared.

## Identity & Profile (8081)

- **User**: `id`, unique lower-case `email`, `passwordHash`, `status`
  (`ACTIVE|LOCKED`), `role` (`CUSTOMER|ADMIN`), `createdAt`, `updatedAt`.
- **UserProfile**: `id`, `userId`, `fullName`, `phone`, `addressLine1`, `city`,
  `postcode`, `updatedAt`.
- Passwords use BCrypt hashes. Registration validates email/password; profile
  access is restricted to the owner or ADMIN.

## Catalog (8082)

- **Category**: `id`, unique `name`, `description`, optional `parentId`.
- **Product**: `id`, unique uppercase `sku`, `name`, `description`, `categoryId`,
  `active`, `createdAt`, `updatedAt`.
- **ProductPrice**: `id`, `productId`, `amount` (scale 2), `currency` (`INR`),
  `validFrom`, optional `validTo`.
- SKU/category uniqueness, positive prices, and current-price validity are
  enforced locally.

## Inventory (8083)

- **InventoryStock**: `productId`, `availableQuantity`, `reservedQuantity`,
  `version`, `updatedAt`.
- **StockReservation**: `id`, `orderRef`, item lines, `status`
  (`PENDING|CONFIRMED|RELEASED|EXPIRED`), `expiresAt` (15 minutes after
  creation), timestamps, `version`; `(orderRef, productId)` is unique.
- **InventoryAdjustment**: `id`, `productId`, signed `delta`, `reason`, `actor`,
  optional idempotent reference, `createdAt`.
- Reservation creation validates every line before changing stock and uses
  optimistic locking. Negative stock and partial reservations are rejected.

## Cart & Pricing (8084)

- **ShoppingCart**: `id`, `customerId`, `status` (`ACTIVE|CHECKED_OUT`),
  `discountCode`, timestamps; one ACTIVE cart per customer.
- **CartItem**: `id`, `cartId`, `productId`, SKU/name snapshot, `quantity` (1-100),
  `unitPrice`, `lineTotal`; `(cartId, productId)` is unique.
- **DiscountRule**: `id`, unique `code`, `type` (`PERCENT|FIXED`), `value`,
  minimum total, active flag, validity period.
- Checkout summary revalidates catalog price, active status, discount rules,
  currency, and totals before returning the order input.

## Order Management (8085)

- **Order**: `id`, `customerId`, `status`, monetary snapshots, `currency`,
  `reservationRef`, `paymentRef`, `cartRef`, `cartCheckedOut`, `failureReason`,
  `correlationId`, timestamps, `version`.
- **OrderItem**: order-local product/SKU/name snapshot, quantity, unit price,
  line total.
- **OrderStatusHistory**: order, from/to statuses, reason, correlation ID,
  timestamp.
- **IdempotencyRecord**: key, request identity, canonical request hash, original
  response, timestamps; key is unique in Order.
- States include `CREATED`, `STOCK_RESERVED`, `PAYMENT_PENDING`, `CONFIRMED`,
  `PAYMENT_FAILED`, `COMPENSATION_PENDING`, `CANCELLED`, and
  `STOCK_UNAVAILABLE`. Customer cancellation is allowed before delivery.

## Payment (8086)

- **PaymentTransaction**: `id`, `orderId`, `customerId`, amount, currency,
  `status` (`INITIATED|SUCCEEDED|DECLINED|TIMED_OUT|REFUNDED`), failure reason,
  timestamps.
- **PaymentAttempt**: payment, attempt number, simulated outcome/delay,
  timestamp.
- **Refund**: payment, amount, status (`REQUESTED|COMPLETED|REJECTED`), reason,
  timestamps; one successful refund per payment.
- **IdempotencyRecord**: key, canonical hash, original response, timestamps;
  key is unique in Payment. Successful refunds complete immediately, while
  explicitly simulated pending/failed outcomes remain visible.

## Cross-cutting rules

- Local transactions are bounded to one service; remote calls never assume
  remote atomicity.
- Material mutations produce audit/status history with actor and correlation
  ID where applicable.
- DTO validation handles shape/range constraints; service rules handle
  ownership, state transitions, uniqueness, idempotency, and compensation.
