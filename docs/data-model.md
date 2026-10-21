| Entity | Attributes |
|---|---|
| User | id (UUID), email (unique, lower-cased), passwordHash (BCrypt), status (ACTIVE, LOCKED), roles (many-to-many with Role), createdAt, updatedAt |
| Role | id, name (CUSTOMER, ADMIN) |
| UserProfile | id, user (one-to-one), fullName, phone, addressLine1, city, postcode, updatedAt |

| Entity | Attributes |
|---|---|
| Category | id, name (unique), description, parent (optional self-reference) |
| Product | id, sku (unique, upper-case), name, description, category, active, createdAt, updatedAt |
| ProductPrice | id, product, amount (BigDecimal 12,2), currency (INR), validFrom, validTo (nullable); the current price is the row where now is inside the validity window |

| Entity | Attributes |
|---|---|
| InventoryStock | productId (primary key, logical reference), availableQuantity, reservedQuantity, version (@Version), updatedAt |
| StockReservation | id, productId, quantity, status (PENDING, CONFIRMED, RELEASED, EXPIRED), orderRef, expiresAt, createdAt, updatedAt, version; unique constraint on (orderRef, productId) |
| InventoryAdjustment | id, productId, delta (positive or negative), reason, actor, createdAt |

| Entity | Attributes |
|---|---|
| ShoppingCart | id, customerId (logical reference, one ACTIVE cart per customer), status (ACTIVE, CHECKED_OUT), discountCode (nullable), createdAt, updatedAt |
| CartItem | id, cart, productId, sku, productName, quantity (1..100), unitPrice (BigDecimal, fetched from Catalog when added); unique (cart_id, productId) |
| DiscountRule | id, code (unique), type (PERCENT, FIXED), value, minimumOrderTotal, active, validFrom, validTo |

| Entity | Attributes |
|---|---|
| Order | id, customerId, status (see state machine), subtotal, discountAmount, totalAmount, currency, reservationRef, paymentRef, idempotencyKey (unique per customer), requestHash (SHA-256 of the request body, for IDEMPOTENCY_CONFLICT), cartRef (cart id from the checkout summary), cartCheckedOut (boolean), failureReason, correlationId, createdAt, updatedAt, version |
| OrderItem | id, order, productId, sku, productName, quantity, unitPriceSnapshot, lineTotal |
| OrderStatusHistory | id, order, fromStatus, toStatus, reason, correlationId, changedAt |

| Entity | Attributes |
|---|---|
| PaymentTransaction | id, orderId, customerId (supplied by the Order Service; used for ownership checks), amount, currency, status (INITIATED, SUCCEEDED, DECLINED, TIMED_OUT, REFUNDED), idempotencyKey (unique), requestHash (SHA-256 of orderId+amount+currency), failureReason, createdAt, updatedAt |
| PaymentAttempt | id, payment, attemptNumber, outcome, simulatedDelayMs, attemptedAt |
| Refund | id, payment, amount, status (REQUESTED, COMPLETED, REJECTED), reason, createdAt |

| Requirement | How it is implemented |
|---|---|
| Primary and foreign keys | JPA @Id on every entity; @ManyToOne/@OneToMany within a service only |
| Unique constraints | @Table(uniqueConstraints = ...) on email, SKU, category name, discount code, idempotency keys, (orderRef, productId) |
| Entity validation | Bean Validation on DTOs (@NotBlank, @Email, @Positive, @DecimalMin) plus service-level business checks |
| Optimistic locking | @Version on InventoryStock, StockReservation, Order |
| Transaction boundaries | @Transactional on service methods; one local transaction per service call, never spanning a remote call |
| Appropriate indexing | @Index on sku, category_id, name, customer_id, order_id, product_id, status |
| Persistent state within the active execution environment | File-based H2 (jdbc:h2:file:./data/<service>db) on the runner during integration; in-memory for unit tests |
| Repeatable database initialisation | spring.jpa.hibernate.ddl-auto=update plus a DataSeeder CommandLineRunner that is idempotent (inserts only if missing) |
| Deterministic test data | Seeded admin user, two categories, four products, fixed SKUs and fixed stock values (see Reference E – Deterministic seed data (Goto Reference Section in this doc to view)) |
| Separate H2 per service | Each service has its own data/ directory and database name; no shared schema |
