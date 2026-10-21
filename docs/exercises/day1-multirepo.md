# Repositories that change

- ecom-cart-service — New discount type for redeeming loyalty points at checkout.
- ecom-loyalty-service — Provides the points balance and redemption API.
- ecom-shared-contracts — Adds the new loyalty-point redemption fields.
- ecom-platform — Updates shared documentation and ADRs.

# Contracts to read

- ecom-order-service — Its checkout orchestration contract must account for the new discount flow.
- ecom-test-data-generator — Its contract-dependent test data must support loyalty points and redemption.

# Repositories to ignore

- ecom-identity-service — User identity is unchanged by loyalty-point redemption.
- ecom-catalog-service — Product and price truth is unchanged.
- ecom-inventory-service — Stock and reservation behaviour is unchanged.
- ecom-payment-service — Payment truth is unchanged.
- ecom-notification-service — Order and payment notifications are unchanged.
- ecom-search-indexer — Catalog search indexing is unrelated.
- ecom-reporting-etl — Existing order and payment extraction is unrelated.
- ecom-admin-portal-bff — Administration portal behaviour is unchanged.
- ecom-mobile-bff — Mobile backend behaviour is unchanged.
- ecom-api-gateway-config — Existing routes, rate limits and auth policies are unchanged.
- ecom-perf-tests — No performance-test changes are required for this business change.
- ecom-infra-iac — Infrastructure as code is not used in the workshop.
- ecom-observability-config — Existing dashboards, alerts and log parsers are unchanged.
- ecom-returns-service — Returns are unrelated to loyalty-point redemption.
- ecom-shipping-adapter — Shipping integration is unrelated.
- ecom-tax-calculator — Existing tax rules are unchanged.
- ecom-fraud-screening — Fraud scoring is unrelated to the discount change.
- ecom-price-feed-importer — Supplier price imports are unrelated.
- ecom-legacy-order-sync — Legacy ERP order synchronisation is unchanged.
- ecom-feature-flags — No feature-flag change is required.
- ecom-customer-support-tools — Support tooling is unchanged.
- ecom-recommendations — Recommendations are unrelated to checkout discounts.
- ecom-warehouse-adapter — Warehouse integration is unrelated.
- ecom-docs-portal — Documentation generation is not part of the requested change.
- ecom-gift-cards — Gift-card issuance and redemption are unrelated.
- ecom-subscriptions — Recurring subscriptions are unrelated.
