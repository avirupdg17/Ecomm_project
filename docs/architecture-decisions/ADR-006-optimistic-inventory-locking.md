# ADR-006: Optimistic locking for stock reservation

**Status:** Accepted  
**Date:** 2026-10-19

## Context

Concurrent customers may attempt to reserve the final unit. Inventory must
allow only one reservation (AC-INV-02) and prevent overselling using optimistic
locking with `@Version` and one retry after `OptimisticLockException`
(NFR-REL-01).

## Decision

Inventory stores a per-product `@Version` value and performs reservation as an
atomic update conditioned on the expected version and available quantity. On a
version conflict, retry once; if it still conflicts or stock is insufficient,
return the defined conflict outcome.

## Consequences

**+** Competing reservations cannot both commit against the same stock
version, preventing overselling without a distributed lock.  
**+** Short transactions preserve throughput for independent products.  
**-** A conflict can surface as a reservation failure and must be handled by
the Order saga as `STOCK_UNAVAILABLE` or compensation.  
**-** Callers must tolerate one bounded retry and explicit conflict responses.

**Rejected alternative:** Pessimistic row locks or an external distributed
lock. They add contention or prohibited infrastructure and are unnecessary for
the per-product concurrency requirement.
