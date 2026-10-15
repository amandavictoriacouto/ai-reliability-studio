# ADR-0003: PostgreSQL as system of record with row-level tenant isolation

- Status: Proposed
- Date: 2026-10-15
- Deciders: product owner (@amandavictoriacouto)
- Related: [MVP architecture](../architecture/mvp-architecture.md), [threat model](../security/threat-model.md), [detailed design](../security/tenant-isolation-and-authz.md)

## Context

Customer product context is confidential. Cross-tenant leakage is a top threat (T02).

## Decision

Use PostgreSQL; every tenant-owned row carries `workspace_id`; isolation is enforced in the repository layer and by PostgreSQL Row-Level Security as defense in depth; cross-tenant tests run in CI.

## Alternatives considered

- Schema-per-tenant: stronger separation, but heavier migrations and operations.
- Database-per-tenant: strongest, but disproportionate for MVP.

## Consequences

- **Positive:** Simple model, testable isolation, RLS catches application bugs.
- **Negative:** RLS requires careful session configuration and connection handling.
