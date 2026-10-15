# ADR-0004: PostgreSQL-backed job queue

- Status: Proposed
- Date: 2026-10-15
- Deciders: product owner (@amandavictoriacouto)
- Related: [MVP architecture](../architecture/mvp-architecture.md), [threat model](../security/threat-model.md)

## Context

The worker needs a durable queue for discovery, verification and scheduled jobs.

## Decision

Use a PostgreSQL-backed queue (`SELECT … FOR UPDATE SKIP LOCKED`, e.g. Procrastinate) behind a small queue interface.

## Alternatives considered

- Celery + Redis: mature, but adds infrastructure.
- Managed cloud queues: vendor lock-in for the demo.

## Consequences

- **Positive:** No extra infrastructure; transactional enqueue with domain writes.
- **Negative:** Throughput ceiling; revisit if job volume grows.
