# ADR-0014: Language tooling

- Status: Proposed
- Date: 2026-10-15
- Updated: 2026-10-16 (incremental adoption; approved M1 simplification)
- Deciders: product owner (@amandavictoriacouto)
- Related: [MVP architecture](../architecture/mvp-architecture.md), [threat model](../security/threat-model.md)

## Context

Consistent, fast developer tooling reduces friction for a solo builder.

## Decision

Python: uv, ruff, mypy (strict on domain modules), pytest. JavaScript: pnpm, ESLint, Prettier, Vitest, Playwright.

Tools are adopted incrementally: each one is added when the first code it checks exists (for example Playwright with the first end-to-end flow). Security-relevant checks are never deferred past the code they protect: dependency audit, secret scanning and the license check run from the first application commit.

## Alternatives considered

- Poetry + Black + flake8: mature but slower and more tools.

## Consequences

- **Positive:** Fast installs and linting.
- **Negative:** Newer tools may change more often.
