# ADR-002: Database per service on H2

**Status:** Accepted  
**Date:** 2026-10-19

## Context

The six services must preserve ownership boundaries and run without external
runtime infrastructure. This serves the architecture's “database per service”
and “no shared persistence” constraints and the local consistency requirements
for order, inventory, and payment state (NFR-REL-06).

## Decision

Each service uses its own H2 database. A service may access only its own
schema, repositories, and entities. Cross-service data access uses REST APIs
and DTOs; no shared or external database is permitted at runtime.

## Consequences

**+** Data ownership and local transactions are explicit; services can evolve
schemas independently and remain runnable without external infrastructure.  
**+** Violations are detectable at the service boundary rather than through
hidden database coupling.  
**-** Cross-service workflows require REST calls and saga compensation instead
of joins or distributed transactions.  
**-** The same reference data may need to be represented in more than one
service.

**Rejected alternative:** One shared H2 or external database. It would couple
schemas, bypass service ownership, and violate the no-shared-persistence and
prohibited-external-database constraints.
