# ADR-0008: Append-only, hash-chained audit log

- Status: Accepted (2026-10-16, product owner)
- Date: 2026-10-15
- Updated: 2026-10-16 (hash formula aligned with the detailed design; per-workspace chains; phased D15 checkpoints; approved revision)
- Deciders: product owner (@amandavictoriacouto)
- Related: [MVP architecture](../architecture/mvp-architecture.md), [threat model](../security/threat-model.md), [detailed design](../security/audit-log-integrity.md)

## Context

Approvals, risk decisions and gate decisions must be non-repudiable and tamper-evident (T07, T08).

## Decision

Write every state change, in the same transaction as the change, to an append-only audit table with **one hash chain per workspace** ([detailed design](../security/audit-log-integrity.md)):

- Each event has a gap-free `seq` per workspace and a `prev_hash` (the previous event's `hash` in that workspace).
- `hash = SHA-256(JCS(event_without_hash))`, where JCS is RFC 8785 canonical JSON. The hashed event includes `workspace_id`, `seq`, `prev_hash`, `occurred_at`, `actor`, `action`, `entity_type`, `entity_id`, `payload_digest` and `request_id`. Payloads are stored as digests, not content.
- Appends lock the workspace's head row to serialize writes.
- The application role has `INSERT` and `SELECT` only; a trigger rejects `UPDATE` and `DELETE`, and `TRUNCATE` is revoked.
- A CLI (`ars audit verify`) recomputes the chain and checks it against checkpoints.
- **Checkpoints follow D15, phased** ([audit log integrity](../security/audit-log-integrity.md) §4). The chain alone is tamper-*evident* only relative to a trusted checkpoint; the guarantees at each phase, and their limits, are stated there.

## Alternatives considered

- Full event sourcing: powerful, but high complexity.
- External ledger service: extra dependency and cost.

## Consequences

- **Positive:** Tamper-evidence at low complexity; good audit story.
- **Negative:** Chain verification cost grows with volume. Until independent checkpoint storage exists (D15 phase 2), an attacker with database owner or migration-role access can rewrite the chain and its checkpoint table together; see the limits in the detailed design.
