# ADR-0007: Content-addressed, write-once evidence storage

- Status: Proposed
- Date: 2026-10-15
- Deciders: product owner (@amandavictoriacouto)
- Related: [MVP architecture](../architecture/mvp-architecture.md), [threat model](../security/threat-model.md), [detailed design](../architecture/evidence-model.md)

## Context

Evidence is the basis of every assurance claim and must be tamper-evident (T07).

## Decision

Store evidence blobs in S3-compatible object storage addressed by SHA-256, write-once (object lock where available); metadata and links live in PostgreSQL.

## Alternatives considered

- Store blobs in PostgreSQL: simpler, but bloats the database and weakens immutability.

## Consequences

- **Positive:** Integrity verifiable by hash; natural de-duplication.
- **Negative:** Requires lifecycle management for retention and deletion requests.
