# ADR-0016: Hosting target (pending approval of Q6)

- Status: Proposed
- Date: 2026-10-15
- Deciders: product owner (@amandavictoriacouto)
- Related: [MVP architecture](../architecture/mvp-architecture.md), [threat model](../security/threat-model.md)

## Context

The MVP needs a low-cost, low-operations public demo environment.

## Decision

Proposed: Vercel for the web app; Fly.io or Render for API and worker containers; Neon for managed PostgreSQL; Cloudflare R2 for evidence storage. Final choice in M1.

## Alternatives considered

- AWS (ECS, RDS, S3): enterprise realism, higher cost and operations.

## Consequences

- **Positive:** Cheap, fast to set up.
- **Negative:** Multiple vendors to manage; data residency to document.
