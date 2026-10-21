# ADR-005: Idempotent order and payment creation

**Status:** Accepted  
**Date:** 2026-10-19

## Context

Timeouts can leave a client unsure whether creation completed. Order and
payment retries must return the original logical transaction rather than
creating a duplicate (NFR-REL-02, AC-ORD-10, AC-PAY-04). The architecture
also requires idempotency for saga calls and compensations.

## Decision

Order and Payment creation endpoints require an `Idempotency-Key`. The service
stores the key with a canonical request hash and original response. Reuse with
the same hash returns that response; reuse with a different hash is rejected
as a conflicting request.

## Consequences

**+** Retries after timeouts cannot create duplicate orders or charges.  
**+** A request hash detects accidental or malicious key reuse with different
payloads.  
**-** Each service must retain idempotency records and define their lifecycle.  
**-** Canonical request serialization must remain stable for hash comparisons.

**Rejected alternative:** Retrying POST creation without an idempotency key or
using only a client-generated order/payment ID. The former can duplicate a
logical transaction; the latter does not reliably detect changed payloads or
an incomplete first response.
