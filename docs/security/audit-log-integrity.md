# Audit Log Integrity: Guarantees and Limits

> Status: **Proposed** (architecture review, M0) · Related: [ADR-0008](../adr/0008-hash-chained-audit-log.md), [threat model](threat-model.md) T07, T08
> Purpose: state precisely what the hash-chained audit log guarantees, what it does **not**, and which additional controls close the gaps.

## 1. Design recap

- **One chain per workspace.** Each event has `seq` (gap-free per workspace), `prev_hash` and `hash = SHA-256(JCS(event_without_hash))`. JCS is RFC 8785 canonical JSON.
- **Hashed fields:** `workspace_id`, `seq`, `prev_hash`, `occurred_at`, `actor` (pseudonymous ID + type: user, API token, system job), `action`, `entity_type`, `entity_id`, `payload_digest` (SHA-256 of the canonical before/after diff) and `request_id`.
- **Atomic writes.** Events are written **in the same database transaction** as the domain change. The workspace head row is locked (`SELECT … FOR UPDATE`) to serialize appends.
- **Database privileges.** The app role has `INSERT` + `SELECT` only. A trigger rejects `UPDATE`/`DELETE`, and `TRUNCATE` is revoked. Migrations run under a separate role.

## 2. What it guarantees

| Guarantee | Condition |
|---|---|
| **G1 Tamper-evidence of history:** any modification, deletion, insertion or reordering of an event before the latest trusted checkpoint is detected. | The verifier holds a trusted checkpoint (§4) |
| **G2 Completeness relative to committed changes:** every committed domain change has its audit event. | Changes go through the application's domain services (same transaction) |
| **G3 Attribution:** each event records which authenticated actor caused it. | Authentication is not compromised |
| **G4 Portable verification:** a workspace can export and verify its own chain without any other tenant's data. | — |

## 3. What it does NOT guarantee (limits)

| # | Limit | Why | Mitigation |
|---|---|---|---|
| L1 | **Full chain rewrite** by someone with database superuser or migration-role access can recompute every hash | A hash chain is self-referential | External, signed checkpoints (§4). A rewrite then mismatches published checkpoints. |
| L2 | **Tail truncation** (deleting the newest events) is invisible inside the chain | There is no successor to break | Checkpoints bound the loss to the interval since the last checkpoint; `seq` gaps are detected against checkpoints. |
| L3 | **Events never written** (bug, direct SQL bypassing services, compromised application) | The chain only protects what was appended | Domain writes only through services; DB role for ad-hoc access is read-only; periodic reconciliation compares entity tables with audit events. |
| L4 | **Truthfulness of content:** a compromised application can append false events | Integrity ≠ truth | Least privilege, authentication controls, anomaly alerts; sensitive actions require recent auth. |
| L5 | **Timestamps are self-asserted** | Server clock | NTP-synced servers; checkpoints carry trusted time (RFC 3161, later). |
| L6 | **Write contention** from serialization per workspace | Locking the head row | Per-workspace chains; this is acceptable at MVP volumes. |
| L7 | **Signing key compromise** allows forged checkpoints | — | Key in KMS, never exported; rotation with `kid`; checkpoints also anchored externally. |

## 4. Checkpoints (anchoring)

- Every **15 minutes**, and on demand, a checkpoint job builds a **Merkle root** over the current head hash of each workspace and signs `{root, workspace_heads, created_at, prev_checkpoint_hash}` with an **Ed25519 key held in KMS**. This key is not the gate-decision key, and database credentials do not grant access to it.
- Checkpoints are written to a **separate write-once bucket** (different credentials; object lock). They are also returned in the workspace's audit export.
- **Later (proposed, D15):** publish the global root to an external timestamping service (RFC 3161 TSA) or to a public transparency repository. This lets third parties detect rewrites without trusting us.

## 5. Verification

`ars audit verify --workspace <id> [--from-checkpoint <id>]` performs these checks:

1. Recomputes every hash and checks `prev_hash` linkage and `seq` continuity.
2. Checks the head hashes against signed checkpoints (signature and Merkle inclusion proof).
3. Reports the first divergence, if any.

A scheduled job runs verification daily for all workspaces and alerts on failure.

## 6. Privacy within an immutable log

- Events store **pseudonymous actor IDs**, not names or emails. The identity mapping lives in a separate, mutable table.
- Payloads store **digests**, not content. Free text such as justifications is stored in the entity tables, with only its hash in the event.
- A valid LGPD erasure request, where no legal hold or retention obligation applies ([evidence model](../architecture/evidence-model.md) §5.4), deletes the identity mapping and the free-text fields. The chain stays verifiable, and the events become pseudonymous.

## 7. Tests

**Automated:**

- Modify, delete, reorder and insert events, each of which must be detected.
- Tail truncation must be detected given a later checkpoint and undetected without one. This test documents L2.
- An UPDATE/DELETE by the app role is rejected.
- A domain change rolled back means no event is written; a committed change means exactly one event.
- Checkpoint signatures and inclusion proofs are verified.
- The export round-trip verifies.

**Human review:** database role grants and the KMS key policy, at each milestone.
