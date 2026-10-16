# ADR-0006: Declarative YAML release policies

- Status: Proposed
- Date: 2026-10-15
- Deciders: product owner (@amandavictoriacouto)
- Related: [MVP architecture](../architecture/mvp-architecture.md), [threat model](../security/threat-model.md), [detailed design](../architecture/release-gate-binding.md)

## Context

Release gates must be understandable by product managers and auditable as code.

## Decision

Policies are versioned YAML documents validated by JSON Schema and evaluated by a small Python engine behind a `PolicyEngine` interface.

## Alternatives considered

- OPA/Rego: powerful and standard, but a new language for product users.
- Cedar: strong authorization model, less suited to release rules.

## Consequences

- **Positive:** PM-readable, reviewable in PRs, easy to test with scenarios.
- **Negative:** Limited expressiveness; OPA may be added as an alternative engine.
