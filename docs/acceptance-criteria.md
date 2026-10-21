# Acceptance criteria

## Identity

- [ ] AC-IDN-01 Given I submit a valid registration request, when the request is processed, then the API returns 201 Created (201 Created)
- [ ] AC-IDN-02 Given I submit valid login credentials for an existing account, when authentication succeeds, then the API returns 200 OK (200 OK)
- [ ] AC-IDN-03 Given I submit invalid credentials, when authentication is rejected, then the API returns 401 Unauthorized (401 Unauthorized)
- [ ] AC-IDN-04 Given I request my profile with valid authentication, when the profile is found, then the API returns 200 OK (200 OK)
- [ ] AC-IDN-05 Given I submit a valid profile update, when the update is applied, then the API returns 200 OK (200 OK)
- [ ] AC-IDN-06 Given I request a profile that does not exist, when it cannot be found, then the API returns 404 Not Found (404 Not Found)

## Catalog

- [ ] AC-CAT-01 Given the catalog contains products, when I request the catalog or search results, then the API returns 200 OK (200 OK)
- [ ] AC-CAT-02 Given I provide search terms or filters, when matching products exist, then the API returns 200 OK with only matching products (200 OK)
- [ ] AC-CAT-03 Given no products match my search or filters, when the request is processed, then the API returns 200 OK with an empty result set (200 OK)
- [ ] AC-CAT-04 Given I submit a valid product or category create request, when it is processed, then the API returns 201 Created (201 Created)
- [ ] AC-CAT-05 Given I submit a valid product or category update request, when it is processed, then the API returns 200 OK (200 OK)
- [ ] AC-CAT-06 Given I request a product or category that does not exist, when it cannot be found, then the API returns 404 Not Found (404 Not Found)

## Inventory

- [ ] AC-INV-01 Given sufficient stock is available, when I request a reservation, then the API returns 200 OK (200 OK)
- [ ] AC-INV-02 Given the final unit is being reserved concurrently, when only one reservation can succeed, then the successful request returns 200 OK and the conflicting request returns 409 Conflict (200 OK; 409 Conflict)
- [ ] AC-INV-03 Given stock is insufficient, when I request a reservation, then the API returns 409 Conflict (409 Conflict)
- [ ] AC-INV-04 Given a payment fails, when the order is compensated, then the reservation is released and the API returns 200 OK (200 OK)
- [ ] AC-INV-05 Given a payment succeeds, when the order is completed, then inventory is updated and the API returns 200 OK (200 OK)
- [ ] AC-INV-06 Given the inventory record cannot be found, when an update is requested, then the API returns 404 Not Found (404 Not Found)
- [ ] AC-INV-07 Given a service client submits a valid reservation request, when stock is available, then the API returns 200 OK (200 OK)
- [ ] AC-INV-08 Given a service client requests reservation release or restock after compensation or cancellation, when the request is processed, then the API returns 200 OK (200 OK)

## Cart and Pricing

- [ ] AC-CART-01 Given I add a valid product to my cart, when the request is processed, then the API returns 200 OK (200 OK)
- [ ] AC-CART-02 Given I update an existing cart line or remove an item, when the request is processed, then the API returns 200 OK (200 OK)
- [ ] AC-CART-03 Given I request a cart that does not exist, when it cannot be found, then the API returns 404 Not Found (404 Not Found)
- [ ] AC-CART-04 Given items and eligible discounts are present in the cart, when pricing is calculated, then the API returns 200 OK with the calculated total (200 OK)
- [ ] AC-CART-05 Given a cart has no eligible discounts, when pricing is calculated, then the API returns 200 OK with the undiscounted total (200 OK)
- [ ] AC-CART-06 Given the cart cannot be priced because required cart data is missing, when the request is processed, then the API returns 400 Bad Request (400 Bad Request)

## Order

- [ ] AC-ORD-01 Given a valid cart and reserved stock, when I create an order, then the API returns 201 Created (201 Created)
- [ ] AC-ORD-02 Given an order exists, when its state changes as part of processing, then the API returns 200 OK (200 OK)
- [ ] AC-ORD-03 Given I request an order that does not exist, when it cannot be found, then the API returns 404 Not Found (404 Not Found)
- [ ] AC-ORD-04 Given a cancellable order exists, when I cancel it, then the API returns 200 OK (200 OK)
- [ ] AC-ORD-05 Given a cancelled order triggers a simulated refund, when the refund is processed, then the API returns 200 OK (200 OK)
- [ ] AC-ORD-06 Given an order is already completed or cancelled, when I request cancellation again, then the API returns 409 Conflict (409 Conflict)
- [ ] AC-ORD-07 Given payment processing fails, when the order is updated, then the API returns 200 OK (200 OK)
- [ ] AC-ORD-08 Given payment fails, when compensation is applied, then the order is not marked CONFIRMED and enters COMPENSATION_PENDING while the reservation is released (200 OK)
- [ ] AC-ORD-09 Given an order is already in a terminal failure state, when the failure transition is requested again, then the API returns 409 Conflict (409 Conflict)
- [ ] AC-ORD-10 Given an order request is retried after a timeout with the same idempotency key, when the original transaction exists, then the API returns the original transaction instead of creating a duplicate order or payment (200 OK)
- [ ] AC-ORD-11 Given a cancelled order triggers a simulated refund and inventory restock, when the cancellation is processed, then the API returns 200 OK (200 OK)

## Payment

- [ ] AC-PAY-01 Given a valid payment request, when the payment is approved, then the API returns 200 OK (200 OK)
- [ ] AC-PAY-02 Given a payment request fails, when the payment is declined, then the API returns 402 Payment Required (402 Payment Required)
- [ ] AC-PAY-03 Given a payment request cannot be processed because the order is unknown, when the request is processed, then the API returns 404 Not Found (404 Not Found)
- [ ] AC-PAY-04 Given a payment request is retried after a timeout with the same idempotency key, when the original payment exists, then the API returns the original transaction instead of charging twice (200 OK)

## End-to-end scenarios

- [ ] AC-E2E-01 Given a customer registers and then authenticates successfully, when the identity flow is completed, then the API returns 201 Created for registration and 200 OK for login (201 Created; 200 OK)
- [ ] AC-E2E-02 Given a customer views and updates a profile with valid authentication, when the profile exists, then the API returns 200 OK (200 OK)
- [ ] AC-E2E-03 Given a customer browses the catalog and an administrator manages products and categories, when the requests are valid, then the API returns 200 OK or 201 Created as appropriate (200 OK; 201 Created)
- [ ] AC-E2E-04 Given a customer searches and filters the catalog, when matches exist or do not exist, then the API returns 200 OK with matching products or an empty result set (200 OK)
- [ ] AC-E2E-05 Given two customers compete for the final unit, when only one reservation can succeed, then the successful request returns 200 OK and the conflicting request returns 409 Conflict (200 OK; 409 Conflict)
- [ ] AC-E2E-06 Given a customer adds, updates, and removes cart items, when the requests are processed, then the API returns 200 OK (200 OK)
- [ ] AC-E2E-07 Given a cart is priced with eligible discounts or none, when pricing is calculated, then the API returns 200 OK (200 OK)
- [ ] AC-E2E-08 Given a customer creates an order from a valid cart and reserved stock, when the order is created, then the API returns 201 Created (201 Created)
- [ ] AC-E2E-09 Given an order changes state during processing, when state transitions occur, then the API returns 200 OK (200 OK)
- [ ] AC-E2E-10 Given a customer makes a simulated payment, when the payment is approved or declined, then the API returns 200 OK or 402 Payment Required as appropriate (200 OK; 402 Payment Required)
- [ ] AC-E2E-11 Given payment fails and compensation is applied, when recovery is processed, then the API returns 200 OK and the order does not remain CONFIRMED (200 OK)
- [ ] AC-E2E-12 Given a customer cancels an order, when refund and inventory restock are processed, then the API returns 200 OK (200 OK)
- [ ] AC-E2E-13 Given a payment succeeds, when inventory is updated after the transaction, then the API returns 200 OK (200 OK)
- [ ] AC-E2E-14 Given the application exposes health endpoints, logs, correlation IDs, error bodies, status history, attempts, and failure diagnostics, when these observability paths are exercised, then the API returns 200 OK where applicable (200 OK)
- [ ] AC-E2E-15 Given the final-laptop scenario is retried after a timeout, when idempotency and compensation are applied, then the API returns the original transaction and prevents duplicate payment (200 OK)
