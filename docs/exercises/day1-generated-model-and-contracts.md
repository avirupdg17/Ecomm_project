# Day 1 generated model and contracts

The model is partitioned by service. Every identifier is a UUID string unless a
field is explicitly described otherwise. A logical reference is not a foreign
key: cross-service data is obtained only through the owning service's REST API.
Money uses `BigDecimal` with scale 2 and currency `INR`.

## Identity Service (8081)

### Entities

| Entity | Fields and types | Constraints and unique keys | Version | Relationships |
|---|---|---|---|---|
| User | `id: UUID`, `email: String`, `passwordHash: String`, `status: UserStatus`, `createdAt: Instant`, `updatedAt: Instant` | `email` required, lower-cased, unique; password stored as BCrypt; status is `ACTIVE` or `LOCKED` | none | one-to-many `UserRole` rows |
| Role | `id: UUID`, `name: RoleName` | `name` required; values `CUSTOMER`, `ADMIN`; unique | none | referenced by `UserRole` |
| UserRole | `id: UUID`, `userId: UUID`, `roleId: UUID` | unique `(userId, roleId)`; both IDs are local Identity IDs | none | many-to-one User and Role |
| UserProfile | `id: UUID`, `userId: UUID`, `fullName: String`, `phone: String`, `addressLine1: String`, `city: String`, `postcode: String`, `updatedAt: Instant` | `userId` required and unique; profile fields validated on write | none | one-to-one User |

### Indexes

| Entity | Index |
|---|---|
| User | unique `email`; `status` |
| UserProfile | unique `userId` |
| UserRole | unique `(userId, roleId)`; `roleId` |

### API contracts

| Method and path | Roles | Request fields | Response and success status | Error codes | Requirements |
|---|---|---|---|---|---|
| `POST /api/users/register` | none | `email`, `password`, `fullName` | `201 {id,email,roles:["CUSTOMER"]}` | `400` validation; `409 EMAIL_ALREADY_EXISTS` | US-IDN-01; AC-IDN-01 |
| `POST /api/auth/login` | none | `email`, `password` | `200 {accessToken,tokenType:"Bearer",expiresInSeconds,userId,roles}` | `401 INVALID_CREDENTIALS` | US-IDN-01; AC-IDN-02, AC-IDN-03 |
| `GET /api/users/me` | CUSTOMER, ADMIN | bearer token | `200 {id,email,roles,profile}` | `401` invalid/expired token | US-IDN-02; AC-IDN-04 |
| `PUT /api/users/me` | CUSTOMER, ADMIN | `fullName`, `phone`, `addressLine1`, `city`, `postcode` | `200` updated profile | `400` validation; `401` invalid token | US-IDN-02; AC-IDN-05 |
| `GET /api/users/{id}` | ADMIN | path `id` | `200` user | `401`, `403`, `404 USER_NOT_FOUND` | US-IDN-02; AC-IDN-06 |
| `GET /actuator/health` | none | none | `200 {status:"UP"}` | none | architecture health boundary |

## Catalog Service (8082)

### Entities

| Entity | Fields and types | Constraints and unique keys | Version | Relationships |
|---|---|---|---|---|
| Category | `id: UUID`, `name: String`, `description: String`, `parentId: UUID?` | `name` required and unique; `parentId` is a local self-reference | none | optional local self-reference to Category |
| Product | `id: UUID`, `sku: String`, `name: String`, `description: String`, `categoryId: UUID`, `active: boolean`, `createdAt: Instant`, `updatedAt: Instant` | `sku` required, upper-cased, unique; `categoryId` references local Category | none | many-to-one local Category; product ID is only a logical reference elsewhere |
| ProductPrice | `id: UUID`, `productId: UUID`, `amount: BigDecimal(12,2)`, `currency: String`, `validFrom: Instant`, `validTo: Instant?` | amount positive and scale 2; currency `INR`; current row has `now` within validity window | none | many-to-one local Product |

### Indexes

| Entity | Index |
|---|---|
| Category | unique `name`; `parentId` |
| Product | unique `sku`; `categoryId`; `name`; `active` |
| ProductPrice | `productId`; `(productId, validFrom, validTo)` |

### API contracts

| Method and path | Roles | Request fields | Response and success status | Error codes | Requirements |
|---|---|---|---|---|---|
| `POST /api/categories` | ADMIN | `name`, `description`, `parentId?` | `201` category | `400` validation; `409 CATEGORY_ALREADY_EXISTS` | US-CAT-02; AC-CAT-04 |
| `GET /api/categories` | none | none | `200` category list | none | US-CAT-01; AC-CAT-01 |
| `POST /api/products` | ADMIN | `sku`, `name`, `description`, `categoryId`, `price:{amount,currency}`, `active` | `201` product with current price | `400 INVALID_PRICE` or validation; `409 SKU_ALREADY_EXISTS` | US-CAT-02; AC-CAT-04 |
| `GET /api/products` | none | query `page`, `size`, `sort` | `200 {content,page,size,totalElements,totalPages}` | `400` invalid pagination/sort | US-CAT-01; AC-CAT-01, AC-CAT-03 |
| `GET /api/products/{id}` | none | path `id` | `200` product | `404 PRODUCT_NOT_FOUND` | US-CAT-01; AC-CAT-01, AC-CAT-06 |
| `PUT /api/products/{id}` | ADMIN | create fields; SKU immutable | `200` updated product | `400` validation; `404 PRODUCT_NOT_FOUND`; `409 SKU_ALREADY_EXISTS` | US-CAT-02; AC-CAT-05, AC-CAT-06 |
| `GET /api/products/search` | none | query `q`, `categoryId`, `minPrice`, `maxPrice`, `active`, pagination | `200` page | `400` invalid filters | US-CAT-01; AC-CAT-02, AC-CAT-03 |
| `GET /api/products/by-sku/{sku}` | none | path `sku` | `200` product | `404 PRODUCT_NOT_FOUND` | US-CAT-01; AC-CAT-01 |

## Inventory Service (8083)

### Entities

| Entity | Fields and types | Constraints and unique keys | Version | Relationships |
|---|---|---|---|---|
| InventoryStock | `productId: UUID`, `availableQuantity: int`, `reservedQuantity: int`, `updatedAt: Instant` | quantities non-negative; `productId` primary key and logical Catalog reference | `@Version version: long` | none; no Product foreign key |
| StockReservation | `id: UUID`, `orderRef: UUID`, `productId: UUID`, `quantity: int`, `status: ReservationStatus`, `expiresAt: Instant`, `createdAt: Instant`, `updatedAt: Instant` | quantity positive; unique `(orderRef, productId)`; statuses `PENDING`, `CONFIRMED`, `RELEASED`, `EXPIRED` | `@Version version: long` | productId and orderRef are logical references |
| InventoryAdjustment | `id: UUID`, `productId: UUID`, `delta: int`, `reason: String`, `reference: String?`, `actor: String`, `createdAt: Instant` | reference is used for idempotent repeated adjustments | none | logical product reference |

### Indexes

| Entity | Index |
|---|---|
| InventoryStock | primary `productId` |
| StockReservation | unique `(orderRef, productId)`; `status`; `expiresAt`; `productId` |
| InventoryAdjustment | `productId`; `reference` |

### API contracts

| Method and path | Roles | Request fields | Response and success status | Error codes | Requirements |
|---|---|---|---|---|---|
| `GET /api/inventory/{productId}` | any authenticated | path `productId` | `200 {productId,availableQuantity,reservedQuantity,version}` | `404 STOCK_NOT_FOUND` | US-INV-01; AC-INV-03 |
| `POST /api/inventory/{productId}/adjustments` | ADMIN, SERVICE | `delta`, `reason`, `reference?` | `200` new stock | `400` validation; `409 NEGATIVE_STOCK` | US-INV-02; AC-INV-05, AC-INV-06, AC-ORD-11 |
| `POST /api/inventory/reservations` | SERVICE, ADMIN | `orderRef`, `items:[{productId,quantity}]` | `201 {reservationId,status:"PENDING",expiresAt,items}`; repeated order returns `200` existing reservation | `400`; `409 INSUFFICIENT_STOCK` | US-INV-01, US-INV-03; AC-INV-01, AC-INV-02, AC-INV-03, AC-INV-07 |
| `POST /api/inventory/reservations/{id}/confirm` | SERVICE, ADMIN | path `id` | `200 {reservationId,status:"CONFIRMED"}` | `404`; `409 INVALID_RESERVATION_STATE` | US-INV-02, US-INV-03; AC-INV-05 |
| `POST /api/inventory/reservations/{id}/release` | SERVICE, ADMIN | path `id` | `200 {reservationId,status:"RELEASED"}` | `404`; `409 INVALID_RESERVATION_STATE` | US-INV-02, US-INV-03; AC-INV-04, AC-INV-08 |
| `GET /api/inventory/reservations/{id}` | any authenticated | path `id` | `200` reservation | `404` | US-INV-01; AC-INV-01 |

## Cart and Pricing Service (8084)

### Entities

| Entity | Fields and types | Constraints and unique keys | Version | Relationships |
|---|---|---|---|---|
| ShoppingCart | `id: UUID`, `customerId: UUID`, `status: CartStatus`, `discountCode: String?`, `createdAt: Instant`, `updatedAt: Instant` | one `ACTIVE` cart per customer; statuses `ACTIVE`, `CHECKED_OUT`; customerId is logical Identity reference | none | one-to-many local CartItem |
| CartItem | `id: UUID`, `cartId: UUID`, `productId: UUID`, `sku: String`, `productName: String`, `quantity: int`, `unitPrice: BigDecimal(12,2)` | quantity 1..100; unique `(cartId, productId)`; product fields are snapshots from Catalog | none | many-to-one local ShoppingCart; productId is logical Catalog reference |
| DiscountRule | `id: UUID`, `code: String`, `type: DiscountType`, `value: BigDecimal(12,2)`, `minimumOrderTotal: BigDecimal(12,2)`, `active: boolean`, `validFrom: Instant`, `validTo: Instant` | code unique; type `PERCENT` or `FIXED`; values and validity are validated | none | none |

### Indexes

| Entity | Index |
|---|---|
| ShoppingCart | unique active customer lookup; `customerId`; `status` |
| CartItem | unique `(cartId, productId)`; `productId` |
| DiscountRule | unique `code`; `active`; `(validFrom, validTo)` |

### API contracts

| Method and path | Roles | Request fields | Response and success status | Error codes | Requirements |
|---|---|---|---|---|---|
| `GET /api/carts/current` | CUSTOMER | bearer subject | `200` active cart | `401`; `404` cart not found | US-CART-01; AC-CART-03 |
| `POST /api/carts/items` | CUSTOMER | `productId`, `quantity` | `201` cart | `400 INVALID_QUANTITY`; `404 PRODUCT_NOT_FOUND` | US-CART-01; AC-CART-01 |
| `PUT /api/carts/items/{itemId}` | CUSTOMER | `quantity` | `200` cart | `400 INVALID_QUANTITY`; `404` | US-CART-01; AC-CART-02 |
| `DELETE /api/carts/items/{itemId}` | CUSTOMER | path `itemId` | `204` | `404` | US-CART-01; AC-CART-02 |
| `POST /api/carts/discount` | CUSTOMER | `code` | `200` cart with discount | `404 DISCOUNT_NOT_FOUND`; `422 DISCOUNT_NOT_APPLICABLE` | US-CART-02; AC-CART-04, AC-CART-05 |
| `POST /api/carts/checkout-summary` | CUSTOMER, SERVICE | optional `customerId` for SERVICE | `200` priced summary | `422 EMPTY_CART`; `400` missing data | US-CART-02; AC-CART-04, AC-CART-06 |
| `POST /api/carts/{cartId}/checked-out` | SERVICE | path `cartId` | `200` marks cart checked out | `404`; `409` invalid cart state | US-ORD-01; AC-ORD-02 |
| `POST /api/discounts` / `GET /api/discounts` | ADMIN | DiscountRule fields for POST | `200`/`201` DiscountRule rows | `400`; `409` duplicate code | US-CART-02; AC-CART-04 |

## Order Service (8085)

### Entities

| Entity | Fields and types | Constraints and unique keys | Version | Relationships |
|---|---|---|---|---|
| Order | `id: UUID`, `customerId: UUID`, `status: OrderStatus`, `subtotal: BigDecimal(12,2)`, `discountAmount: BigDecimal(12,2)`, `totalAmount: BigDecimal(12,2)`, `currency: String`, `reservationRef: UUID?`, `paymentRef: UUID?`, `idempotencyKey: String`, `requestHash: String`, `cartRef: UUID?`, `cartCheckedOut: boolean`, `failureReason: String?`, `correlationId: String`, `createdAt: Instant`, `updatedAt: Instant` | idempotency key unique per customer; request hash SHA-256; totals non-negative; all external IDs logical references | `@Version version: long` | one-to-many local OrderItem and OrderStatusHistory |
| OrderItem | `id: UUID`, `orderId: UUID`, `productId: UUID`, `sku: String`, `productName: String`, `quantity: int`, `unitPriceSnapshot: BigDecimal(12,2)`, `lineTotal: BigDecimal(12,2)` | quantity positive; price snapshot immutable; orderId local | none | many-to-one local Order; productId logical Catalog reference |
| OrderStatusHistory | `id: UUID`, `orderId: UUID`, `fromStatus: OrderStatus?`, `toStatus: OrderStatus`, `reason: String?`, `correlationId: String`, `changedAt: Instant` | transition record required for every state change | none | many-to-one local Order |

### Indexes

| Entity | Index |
|---|---|
| Order | unique `(customerId, idempotencyKey)`; `customerId`; `status`; `createdAt`; `correlationId` |
| OrderItem | `orderId`; `productId` |
| OrderStatusHistory | `orderId`; `toStatus`; `changedAt` |

### API contracts

| Method and path | Roles | Request fields | Response and success status | Error codes | Requirements |
|---|---|---|---|---|---|
| `POST /api/orders` | CUSTOMER | `Idempotency-Key` UUID; optional `simulatedPaymentOutcome` (`SUCCESS`, `DECLINED`, `TIMEOUT`) | `201` order; repeated same request `200` original | `409 INSUFFICIENT_STOCK`; `422 EMPTY_CART` or `IDEMPOTENCY_CONFLICT`; `503 DEPENDENCY_UNAVAILABLE` | US-ORD-01, US-ORD-03, US-ORD-04; AC-ORD-01, AC-ORD-07, AC-ORD-08, AC-ORD-10 |
| `GET /api/orders/{id}` | owner, ADMIN | path `id` | `200` order with statusHistory | `403`; `404` | US-ORD-01; AC-ORD-02, AC-ORD-03 |
| `GET /api/orders/customer/{customerId}` | owner, ADMIN | path `customerId`, pagination | `200` newest-first page | `403`; `404` | US-ORD-01; AC-ORD-02 |
| `POST /api/orders/{id}/cancel` | owner, ADMIN | path `id` | `200` CANCELLED order; refund and restock are idempotent | `403`; `404`; `409 INVALID_ORDER_STATE` | US-ORD-02; AC-ORD-04, AC-ORD-05, AC-ORD-06, AC-ORD-11 |
| `POST /api/orders/{id}/recover` | ADMIN, SERVICE | path `id` | `200` reruns pending step safely | `404`; `409 INVALID_ORDER_STATE`; `503 DEPENDENCY_UNAVAILABLE` | US-ORD-03, US-ORD-04; AC-ORD-08, AC-ORD-10 |

## Payment Service (8086)

### Entities

| Entity | Fields and types | Constraints and unique keys | Version | Relationships |
|---|---|---|---|---|
| PaymentTransaction | `id: UUID`, `orderId: UUID`, `customerId: UUID`, `amount: BigDecimal(12,2)`, `currency: String`, `status: PaymentStatus`, `idempotencyKey: String`, `requestHash: String`, `failureReason: String?`, `createdAt: Instant`, `updatedAt: Instant` | amount positive and scale 2; currency `INR`; idempotency key unique; request hash SHA-256 of orderId+amount+currency; orderId/customerId logical references | none | one-to-many local PaymentAttempt and Refund |
| PaymentAttempt | `id: UUID`, `paymentId: UUID`, `attemptNumber: int`, `outcome: String`, `simulatedDelayMs: long`, `attemptedAt: Instant` | unique `(paymentId, attemptNumber)`; attempt number positive | none | many-to-one local PaymentTransaction |
| Refund | `id: UUID`, `paymentId: UUID`, `amount: BigDecimal(12,2)`, `status: RefundStatus`, `reason: String`, `createdAt: Instant` | amount positive and no greater than successful payment; statuses `REQUESTED`, `COMPLETED`, `REJECTED` | none | many-to-one local PaymentTransaction |

### Indexes

| Entity | Index |
|---|---|
| PaymentTransaction | unique `idempotencyKey`; `orderId`; `customerId`; `status` |
| PaymentAttempt | unique `(paymentId, attemptNumber)`; `paymentId` |
| Refund | `paymentId`; `status` |

### API contracts

| Method and path | Roles | Request fields | Response and success status | Error codes | Requirements |
|---|---|---|---|---|---|
| `POST /api/payments` | SERVICE, ADMIN | `Idempotency-Key`; `orderId`, `customerId`, `amount`, `currency`, `simulatedOutcome?` | `201` transaction; repeated same request `200` original | `400`; `404 ORDER_NOT_FOUND`; `402 PAYMENT_DECLINED`; `422 IDEMPOTENCY_CONFLICT` | US-PAY-01, US-PAY-02; AC-PAY-01, AC-PAY-02, AC-PAY-03, AC-PAY-04 |
| `GET /api/payments/{id}` | owner CUSTOMER, SERVICE, ADMIN | path `id` | `200` transaction with attempts | `403`; `404` | US-PAY-01; AC-PAY-03 |
| `GET /api/payments?orderId=` | owner CUSTOMER, SERVICE, ADMIN | query `orderId` | `200` order transactions | `403`; `404` | US-PAY-02; AC-PAY-04 |
| `POST /api/payments/{id}/refund` | SERVICE, ADMIN | optional `amount`, required `reason` | `201 {refundId,status:"COMPLETED"}`; repeated refund returns existing refund | `404`; `409 INVALID_PAYMENT_STATE` | US-ORD-02; AC-ORD-05, AC-ORD-11 |

## Shared error response

| Field | Type | Constraint |
|---|---|---|
| `timestamp` | ISO-8601 string | response creation time |
| `status` | integer | HTTP status |
| `error` | string | HTTP error name |
| `code` | string | machine-readable service error code |
| `message` | string | safe client-facing message; no stack trace, SQL, or class names |
| `path` | string | request path |
| `correlationId` | string | propagated request correlation identifier |
| `details` | array of objects | optional validation or domain details, such as `field` and `issue` |

## Assumptions

| # | Assumption |
|---|---|
| 1 | `docs/business-requirements.md`, `docs/architecture.md`, `docs/data-model.md`, and `docs/api-contracts.md` are the governing repository inputs; where an acceptance criterion says 200 but the detailed contract says 201 for creation, the detailed endpoint contract is recorded and the discrepancy is not silently changed. |
| 2 | The requirements' references to products, users, orders, reservations, carts, and payments crossing services are logical UUID strings, never database foreign keys or shared JPA entities. |
| 3 | The standard error body is JSON with the fields listed above; individual error codes are included only where the requirements or detailed contract names them. |
| 4 | “Owner” means the authenticated JWT subject matches the resource's customerId; ADMIN and explicitly authorised SERVICE callers may bypass ownership where the endpoint contract says so. |
| 5 | Pagination parameters and sort allow-lists are implementation details of the catalog/order APIs; the documented defaults and maximums are contract constraints, not additional entities. |
| 6 | A cart item stores the Catalog response as a price/name/SKU snapshot, while checkout revalidates current Catalog prices; an order stores immutable item price snapshots. |
| 7 | Idempotency uniqueness is scoped as stated: `(customerId, idempotencyKey)` for orders and a service-local unique key for payments and adjustments. |
| 8 | `@Version` is required on InventoryStock, StockReservation, and Order because the architecture requires optimistic concurrency protection; other entities do not require versioning unless later requirements add it. |
| 9 | `Instant` represents timestamps, `BigDecimal(12,2)` represents monetary values, and integer quantities are non-negative except InventoryAdjustment.delta, which may be positive or negative subject to the resulting stock remaining non-negative. |
