| Repository | Description | Owns / depends on |
|---|---|---|
| ecom-identity-service | Registration, login, JWT, profiles, roles | Owns user identity; depended on by all|
| ecom-catalog-service | Products, categories, prices, search | Owns product and price truth|
| ecom-inventory-service | Stock, reservations, adjustments | Owns stock truth; called by Order|
| ecom-cart-service | Carts, discount rules, checkout summary | Depends on Catalog|
| ecom-order-service | Checkout orchestration, order states, cancellation | Depends on Cart, Inventory, Payment|
| ecom-payment-service | Simulated payments, idempotency, refunds | Owns payment truth|
| ecom-platform | Shared docs, contracts, ADRs, integration workflow | Depended on by all for context|
| ecom-shared-contracts | Versioned JSON schemas of all public DTOs | Changed whenever a contract changes|
| ecom-notification-service | Order and payment notifications (email/SMS simulation) | Depends on Order events|
| ecom-loyalty-service | Loyalty points balance, earn and redeem rules | Would be called by Cart for point discounts|
| ecom-search-indexer | Builds a search index from Catalog changes | Depends on Catalog|
| ecom-reporting-etl | Nightly extraction of orders and payments for reports | Reads Order and Payment APIs|
| ecom-admin-portal-bff | Backend-for-frontend for the administration portal | Depends on Catalog, Inventory, Order|
| ecom-mobile-bff | Backend-for-frontend for the mobile app | Depends on Catalog, Cart, Order|
| ecom-api-gateway-config | Routes, rate limits and auth policies of the gateway | Depends on every public API|
| ecom-test-data-generator | Synthetic customers, products and orders for test environments | Depends on contracts|
| ecom-perf-tests | Load test scripts and thresholds | Depends on public APIs|
| ecom-infra-iac | Infrastructure as code for cloud environments (not used in the workshop) | Platform team|
| ecom-observability-config | Dashboards, alert rules, log parsers | Platform team|
| ecom-returns-service | Return requests and return approvals | Depends on Order, Payment|
| ecom-shipping-adapter | Integration with carrier APIs (simulated) | Depends on Order|
| ecom-tax-calculator | Tax rules by region | Called by Cart for tax lines|
| ecom-fraud-screening | Risk scoring of orders before payment | Would be called by Order|
| ecom-price-feed-importer | Imports supplier price files into Catalog | Writes to Catalog|
| ecom-legacy-order-sync | Synchronises orders with the legacy ERP | Reads Order|
| ecom-feature-flags | Feature flag definitions and SDK config | Used by all services|
| ecom-customer-support-tools | Internal tools for support agents | Depends on Identity, Order, Payment|
| ecom-recommendations | Product recommendation model and API | Depends on Catalog, Order history|
| ecom-warehouse-adapter | Warehouse management system integration | Depends on Inventory|
| ecom-docs-portal | Developer documentation site generated from all repositories | Reads docs of all|
| ecom-gift-cards | Gift card issuance and redemption | Would be called by Payment|
| ecom-subscriptions | Recurring order subscriptions | Depends on Order, Payment|
