# ADR-0008: Append-only, hash-chained audit log

- Status: Proposed
- Date: 2026-10-15
- Deciders: product owner (@amandavictoriacouto)
- Related: [MVP architecture](../architecture/mvp-architecture.md), [threat model](../security/threat-model.md), [detailed design](../security/audit-log-integrity.md)

## Context

Approvals, risk decisions and gate decisions must be non-repudiable and tamper-evident (T07, T08).

## Decision

Write every state change to an append-only table where `hash = sha256(prev_hash || canonical_event)`; the application role has no UPDATE/DELETE on it; a CLI verifies the chain.

## Alternatives considered

- Full event sourcing: powerful, but high complexity.
- External ledger service: extra dependency and cost.

## Consequences

- **Positive:** Tamper-evidence at low complexity; good audit story.
- **Negative:** Chain verification cost grows with volume; checkpointing may be needed.
