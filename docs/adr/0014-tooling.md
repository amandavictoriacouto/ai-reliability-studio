# ADR-0014: Language tooling

- Status: Proposed
- Date: 2026-10-15
- Deciders: product owner (@amandavictoriacouto)
- Related: [MVP architecture](../architecture/mvp-architecture.md), [threat model](../security/threat-model.md)

## Context

Consistent, fast developer tooling reduces friction for a solo builder.

## Decision

Python: uv, ruff, mypy (strict on domain modules), pytest. JavaScript: pnpm, ESLint, Prettier, Vitest, Playwright.

## Alternatives considered

- Poetry + Black + flake8: mature but slower and more tools.

## Consequences

- **Positive:** Fast installs and linting.
- **Negative:** Newer tools may change more often.
