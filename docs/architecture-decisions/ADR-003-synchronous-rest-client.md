# ADR-003: Synchronous REST with RestClient

**Status:** Accepted  
**Date:** 2026-10-19

## Context

Services need immediate responses for catalog, cart, reservation, order, and
payment operations. The platform specifies synchronous REST, `RestClient`, a
1000 ms connect timeout, and a 2000 ms read timeout (NFR-REL-03), while
prohibiting Kafka and RabbitMQ at runtime.

## Decision

Use synchronous HTTP calls through Spring `RestClient`. Every client uses a
1-second connect timeout and 2-second read timeout, propagates
`X-Correlation-Id` and the applicable bearer or `SERVICE` token, and maps
non-success responses explicitly.

## Consequences

**+** Callers receive deterministic success or failure outcomes and the
implementation matches the service API boundaries.  
**+** Timeouts prevent a hanging dependency from blocking a caller
indefinitely, supporting recovery and compensation.  
**-** A dependency outage can increase request latency or fail the current
workflow.  
**-** The Order Service must persist state and compensate across calls.

**Rejected alternative:** Kafka or RabbitMQ messaging. It would add prohibited
runtime infrastructure and asynchronous delivery semantics where the required
checkout and payment APIs need an immediate result.
