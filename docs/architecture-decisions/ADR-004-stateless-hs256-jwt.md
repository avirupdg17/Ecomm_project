# ADR-004: Stateless HS256 JWT and service tokens

**Status:** Accepted  
**Date:** 2026-10-19

## Context

All services must validate bearer authorization while preserving service
boundaries. The platform requires stateless HS256 JWT authentication signed
with `JWT_SECRET`, CUSTOMER/ADMIN/SERVICE roles (NFR-SEC-01), and signed
SERVICE-subject calls for reservation and payment writes (NFR-SEC-12).

## Decision

Use stateless HS256 JWTs signed with the shared `JWT_SECRET`, loaded from
environment or GitHub Actions secrets. Propagate the caller bearer token for
user-context calls; use short-lived, scoped `SERVICE` tokens for background,
retry, reconciliation, and compensation work.

## Consequences

**+** Every service can validate tokens locally without a session store, and
background work has an explicit non-user identity.  
**+** Role and ownership checks remain enforceable at each API boundary.  
**-** Secret distribution and rotation must be coordinated across services.  
**-** Stateless tokens require short lifetimes and careful revocation policy.

**Rejected alternative:** A shared server-side session or fabricated user
tokens. Sessions add prohibited shared runtime state, while fabricated user
identity would weaken authorization and auditability.
