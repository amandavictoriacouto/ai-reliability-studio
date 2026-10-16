# ADR-0004: PostgreSQL-backed job queue

- Status: Proposed
- Date: 2026-10-15
- Updated: 2026-10-16 (queue roles and Job-Lease API, review findings F4 and F7)
- Deciders: product owner (@amandavictoriacouto)
- Related: [MVP architecture](../architecture/mvp-architecture.md), [threat model](../security/threat-model.md), [detailed design](../architecture/job-lease-api.md)

## Context

The worker needs a durable queue for discovery, verification and scheduled jobs.

## Decision

Use a PostgreSQL-backed queue (`SELECT … FOR UPDATE SKIP LOCKED`, e.g. Procrastinate) behind a small queue interface, with these constraints from the architecture review ([job queue and Job-Lease API](../architecture/job-lease-api.md)):

- The queue table holds **metadata only** (job type, `workspace_id`, typed subject reference, requester, idempotency key, lease fields). No customer content or secrets.
- It is a platform table with forced RLS and deny-by-default. The request path can only enqueue through a `SECURITY DEFINER` function bound to its current workspace. A dedicated `ars_queue` role claims and completes jobs and has no access to tenant tables.
- Job execution runs as `ars_worker` (no `BYPASSRLS`) under `SET LOCAL app.workspace_id` from the job, and re-checks the requester's permission before doing work.
- Leases have an expiry, a fencing generation and bounded retries. Enqueue is idempotent.
- The **verifier runner never polls the database.** It obtains work through the internal Job-Lease API with short-lived, job-scoped credentials.

## Alternatives considered

- Celery + Redis: mature, but adds infrastructure.
- Managed cloud queues: vendor lock-in for the demo.

## Consequences

- **Positive:** No extra infrastructure; transactional enqueue with domain writes.
- **Negative:** Throughput ceiling; revisit if job volume grows. The Job-Lease API adds an internal endpoint surface that must be tested as security-sensitive code.
