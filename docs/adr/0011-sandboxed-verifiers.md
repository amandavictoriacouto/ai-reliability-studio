# ADR-0011: Sandboxed verifier execution

- Status: Proposed
- Date: 2026-10-15
- Deciders: product owner (@amandavictoriacouto)
- Related: [MVP architecture](../architecture/mvp-architecture.md), [threat model](../security/threat-model.md), [detailed design](../security/endpoint-verifier-security.md)

## Context

Verifiers call external systems, including customer-registered LLM endpoints, creating SSRF and abuse risk (T03).

## Decision

Verifiers run in the worker behind a plugin interface with egress allow-listing, private-range blocking after DNS resolution, timeouts, no shared credentials, and mandatory endpoint ownership verification.

## Alternatives considered

- Run verifiers only inside customer CI: safer for us, worse onboarding.

## Consequences

- **Positive:** Safe-by-default managed verification.
- **Negative:** Network controls add implementation and testing effort.
