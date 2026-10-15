# Evidence Model: Integrity, Retention, Access and Deletion

> Status: **Proposed** (architecture review, M0) · Related: [ADR-0007](../adr/0007-content-addressed-evidence.md), [release gate binding](release-gate-binding.md), [audit log integrity](../security/audit-log-integrity.md), [threat model](../security/threat-model.md) T07, T10, T11

## 1. Evidence types

| Type | Produced by | Typical binding | Example |
|---|---|---|---|
| `automated_result` | Verifier / CI integration | `commit` or `time` | Probe transcript, repo posture snapshot, test report |
| `review_record` | Human reviewer in the app | `release` | Completed design-review checklist with reviewer identity |
| `document` | Upload | `release` or `time` | DPIA/RIPD, policy PDF, pen-test report |
| `attestation` | Named person in the app | `release` | "Data retention is configured to 30 days" signed by the owner |

Each evidence record has `id` (UUIDv7), `workspace_id`, `type`, `source` (verifier ID and version, or uploader), `collected_at`, `sha256`, `size`, `media_type`, `binding` (+ `commit_sha` when commit-bound), `valid_until`, `storage_key` and `status`.

## 2. Integrity

| Mechanism | Detail |
|---|---|
| **Server-side hashing** | SHA-256 is computed by the API while streaming the upload or verifier output. A hash sent by the client is ignored. |
| **Content addressing within a tenant** | `storage_key = <workspace_id>/<sha256>`. Content is de-duplicated **only inside a workspace**; it is never shared across tenants, to avoid cross-tenant inference. |
| **Write-once storage** | The application role can `PUT` new keys and `GET`, but has **no overwrite or delete permission** on the evidence bucket. Object lock (governance mode) is used where the provider supports it. Deletion goes through a separate, audited deletion service (§5). |
| **Immutable metadata** | Evidence rows are insert-only. A correction creates a new evidence record that `supersedes` the previous one. |
| **Audit linkage** | Creating, linking, superseding or deleting evidence emits an audit event that contains the `sha256`, never the content. |
| **Periodic re-verification** | A daily job re-hashes a random sample (plus everything referenced by decisions in the last 7 days) and alerts on any mismatch. |
| **Decision manifests** | Each gate decision stores the list of `(evidence_id, sha256)` it used ([release gate binding](release-gate-binding.md) §3). |

## 3. Confidentiality and encryption

- TLS in transit. Provider-side encryption at rest.
- **Per-workspace envelope encryption (proposed, D14):** objects are encrypted with a per-workspace data key (DEK), which is wrapped by a KMS key. This enables **crypto-shredding**: destroying the DEK makes the whole workspace's evidence unreadable, even in backups.
- **Secret and PII screening on ingest:** uploads and verifier outputs pass a secret scanner (gitleaks rules) and a PII detector. When something is detected, the evidence is quarantined and the uploader must confirm or redact it before it is linked.
- **Upload hardening:** allow-listed types (PDF, PNG, JPEG, Markdown, plain text, JSON, CSV), at most 10 MB, magic-byte sniffing, no server-side rendering of HTML/SVG, and downloads served from a separate domain with `Content-Disposition: attachment`.

## 4. Access

| Action | Roles | Notes |
|---|---|---|
| See evidence metadata | All workspace members | Hash, type, status, dates |
| Download content | `owner`, `admin`, `security`, `grc`, `product`, `engineer` | Through a **pre-signed URL valid for 5 minutes**, issued after the authorization check; every download is an audit event |
| Upload / attest | `owner`, `admin`, `security`, `grc`, `product`, `engineer` | — |
| Delete content (§5) | `owner`, `admin` | Recent authentication + justification |
| External auditor (later) | Time-boxed, read-only role scoped to selected releases | — |

The bucket is private. No public URLs. Pre-signed URLs are not logged in full.

## 5. Retention and deletion

### 5.1 Lifecycle states

`quarantined → active → stale (valid_until passed) → superseded → deleted (tombstone)`. `legal_hold` can be applied to any state and blocks deletion.

### 5.2 Retention defaults (to be confirmed, D14)

| Data | Default | Configurable |
|---|---|---|
| Evidence referenced by a gate decision | Life of the workspace, minimum 3 years after the decision | Workspace admins may set 1–7 years |
| Unreferenced evidence | 1 year after `collected_at` | 90 days – 7 years |
| Quarantined, never confirmed | 7 days | No |
| Probe transcripts | 1 year | 90 days – 3 years |

A scheduled job applies retention. Deletion produces a tombstone.

### 5.3 Deletion paths

| Trigger | What happens |
|---|---|
| **Accidental upload** of secrets or personal data | An admin deletes the content with a justification. The object is removed, and the row becomes a **tombstone** that keeps `id`, `sha256`, `size`, `type`, deleter, reason and time. Decisions that referenced it show "evidence redacted". The audit chain stays verifiable, because it only ever contained hashes. |
| **Retention expiry** | The object is deleted automatically, a tombstone is kept and an audit event is emitted. |
| **Data subject request (LGPD Art. 18)** | Content containing that person's data is deleted or redacted (a new redacted version supersedes the old one). Identity data in audit events is pseudonymized ([audit log integrity](../security/audit-log-integrity.md) §6). |
| **Workspace deletion** | 30-day grace period (recoverable), then crypto-shredding of the workspace DEK, object deletion, database rows deleted except tombstones, and the workspace's audit chain is exported to the owner before final removal. |
| **Backups** | Backups expire after at most 35 days. Crypto-shredding makes evidence in older backups unreadable. |

**Tension, made explicit:** write-once storage and legal deletion requirements conflict. We resolve this by using object lock in **governance** mode, not compliance mode. Only the audited deletion service can delete, and every deletion leaves a tombstone. This preserves tamper-*evidence* (any deletion is visible) without promising tamper-*proof* retention.

## 6. Tests

All automated: hash computed server-side (client hash ignored); overwrite attempt by the app role fails; deletion without the deletion service fails; a tombstone keeps the hash while the content is unavailable; a cross-tenant key access attempt fails; a pre-signed URL expires; a quarantine blocks linking; retention job behavior; re-verification detects a modified object (fault injection).

Human review: bucket policy and key-management configuration, per environment, at each milestone.
