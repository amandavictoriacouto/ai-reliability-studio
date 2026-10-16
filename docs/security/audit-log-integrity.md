# Audit Log Integrity: Guarantees and Limits

> Status: **Accepted** with ADR-0008 and D15 (phased), 2026-10-16 · Related: [ADR-0008](../adr/0008-hash-chained-audit-log.md), [threat model](threat-model.md) T07, T08
> Purpose: state precisely what the hash-chained audit log guarantees, what it does **not**, and which additional controls close the gaps.

## 1. Design recap

- **One chain per workspace.** Each event has `seq` (gap-free per workspace), `prev_hash` and `hash = SHA-256(JCS(event_without_hash))`. JCS is RFC 8785 canonical JSON.
- **Hashed fields:** `workspace_id`, `seq`, `prev_hash`, `occurred_at`, `actor` (pseudonymous ID + type: user, API token, system job), `action`, `entity_type`, `entity_id`, `payload_digest` (SHA-256 of the canonical before/after diff) and `request_id`.
- **Atomic writes.** Events are written **in the same database transaction** as the domain change. The workspace head row is locked (`SELECT … FOR UPDATE`) to serialize appends.
- **Database privileges.** The app role has `INSERT` + `SELECT` only. A trigger rejects `UPDATE`/`DELETE`, and `TRUNCATE` is revoked. Migrations run under a separate role.

## 2. What it guarantees

| Guarantee | Condition |
|---|---|
| **G1 Tamper-evidence of history:** any modification, deletion, insertion or reordering of an event before the latest trusted checkpoint is detected. | The verifier holds a trusted checkpoint (§4). In phase 1 that means a checkpoint the attacker could not also delete, such as one in an export kept outside the platform |
| **G2 Completeness relative to committed changes:** every committed domain change has its audit event. | Changes go through the application's domain services (same transaction) |
| **G3 Attribution:** each event records which authenticated actor caused it. | Authentication is not compromised |
| **G4 Portable verification:** a workspace can export and verify its own chain without any other tenant's data. | — |

## 3. What it does NOT guarantee (limits)

| # | Limit | Why | Mitigation |
|---|---|---|---|
| L1 | **Full chain rewrite** by someone with database superuser or migration-role access can recompute every hash | A hash chain is self-referential | Signed checkpoints (§4). In phase 1 they live in the same database, so a database-level attacker can also delete them (§4.1); independence comes with phases 2 and 3. |
| L2 | **Tail truncation** (deleting the newest events) is invisible inside the chain | There is no successor to break | Checkpoints bound the loss to the interval since the last checkpoint (one hour in phase 1); `seq` gaps are detected against checkpoints. Deleting the checkpoints too is only detectable from phase 2 or from exports held outside the platform. |
| L3 | **Events never written** (bug, direct SQL bypassing services, compromised application) | The chain only protects what was appended | Domain writes only through services; DB role for ad-hoc access is read-only; periodic reconciliation compares entity tables with audit events. |
| L4 | **Truthfulness of content:** a compromised application can append false events | Integrity ≠ truth | Least privilege, authentication controls, anomaly alerts; sensitive actions require recent auth. |
| L5 | **Timestamps are self-asserted** | Server clock | NTP-synced servers; checkpoints carry trusted time (RFC 3161, later). |
| L6 | **Write contention** from serialization per workspace | Locking the head row | Per-workspace chains; this is acceptable at MVP volumes. |
| L7 | **Signing key compromise** allows forged checkpoints | — | Key in the platform secret store (a KMS where the host offers one for free), never exported; rotation with `kid`; external anchoring in phase 3. |

## 4. Checkpoints (anchoring), phased (D15)

D15 was approved on 2026-10-16 as a **phased** implementation. Each phase adds independence from the system being audited. The guarantees of a phase must never be described more strongly than stated here.

| Phase | What is built | Where checkpoints live | What it adds |
|---|---|---|---|
| **1 (M1)** | Every hour and on demand, a job signs `{workspace_heads, created_at, prev_checkpoint_hash}` with an **Ed25519 key held in the platform secret store** (not in the database, not readable by the app, worker or queue roles). A daily job verifies every chain against the latest checkpoints. | An append-only checkpoint table **in the same PostgreSQL database**, plus a copy in every workspace audit export | Detection of tampering by anyone who can change audit rows but cannot also produce a valid checkpoint signature; tail truncation detectable back to the last checkpoint |
| **2 (M3)** | Merkle tree over workspace heads with per-workspace inclusion proofs (exports no longer show other tenants' heads). Each checkpoint is copied to **separate write-once object storage** with different credentials. | Database + independent write-once storage | Detection that does not depend on the database being intact |
| **3 (v0.2 or later)** | The checkpoint root is anchored externally: an RFC 3161 timestamp authority or a public transparency repository. | Database + write-once storage + external anchor | Detection without trusting the operator; trusted time |

### 4.1 Phase 1 limits (M1)

Phase 1 checkpoints are stored in the same database as the chain. Therefore:

- They are **not independent or tamper-proof storage**. An attacker with database owner, superuser or migration-role access can delete or rewrite the chain **and** the checkpoint table together.
- Phase 1 **does not guarantee detection of complete history deletion.** If the chain, the checkpoints and the head rows of a workspace are all removed, nothing inside the system proves they existed. Detection then depends on copies held outside the database: audit exports already downloaded by the workspace, backups, or (from phase 2) the write-once store.
- Rewriting history with *valid-looking* checkpoints requires the signing key as well. Without it, rewritten checkpoints fail signature verification, but **deleted** checkpoints leave no signature to check.
- Timestamps remain self-asserted (L5) until phase 3.

### 4.2 Trust assumptions (phase 1)

1. The checkpoint signing key stays in the platform secret store and is only available to the checkpoint job. It is not in the database, logs or repository, and it is different from the gate-decision key.
2. Database owner, superuser and migration-role credentials are not available to the running application and are used only for migrations.
3. The daily verification job runs and its alerts are monitored.
4. Workspace owners who need independent evidence download audit exports, which include the signed checkpoints, and keep them outside the platform.

### 4.3 Residual risks (phase 1)

| Risk | Impact | Treatment |
|---|---|---|
| Database-level attacker deletes chain and checkpoints together | Loss of audit history without in-system detection | Accepted for M1 (no customer data in the demo); closed by phase 2 |
| Signing key compromised | Forged checkpoints over a rewritten chain | Key rotation with `kid`; key separation; closed further by phase 3 anchoring |
| Verification job silently stops | Tampering detected late | Job health is monitored; a missing run raises an alert |
| Exports describe phase 1 as stronger than it is | Over-claiming | Assurance-language rules: exports and UI state the current phase and its limits |

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
- Checkpoint signatures are verified (inclusion proofs from phase 2).
- A rewritten chain with an unchanged checkpoint fails verification; deleting chain **and** checkpoints is detected only against an external export (this test documents the phase 1 limit).
- The export round-trip verifies.

**Human review:** database role grants and the access policy of the checkpoint signing key (secret store in phase 1), at each milestone.
