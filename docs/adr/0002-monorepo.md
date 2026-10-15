# ADR-0002: Monorepo layout

- Status: Proposed
- Date: 2026-10-15
- Deciders: product owner (@amandavictoriacouto)
- Related: [MVP architecture](../architecture/mvp-architecture.md), [threat model](../security/threat-model.md)

## Context

API, web app, catalog content, GitHub Action and documentation evolve together; reviewers and portfolio readers benefit from one place.

## Decision

Use a single repository with `apps/`, `packages/`, `integrations/`, `catalog/`, `evals/`, `docs/` and `infra/`.

## Alternatives considered

- Polyrepo per component: cleaner ownership, but cross-cutting changes need coordinated PRs.

## Consequences

- **Positive:** Atomic PRs across API, UI, catalog and docs.
- **Negative:** CI must use path filters to stay fast.
