# ADR-0012: OpenTelemetry for observability

- Status: Proposed
- Date: 2026-10-15
- Deciders: product owner (@amandavictoriacouto)
- Related: [MVP architecture](../architecture/mvp-architecture.md), [threat model](../security/threat-model.md)

## Context

AI calls, jobs and gate latency must be observable without vendor lock-in.

## Decision

Instrument web, API and worker with OpenTelemetry, using GenAI semantic conventions for LLM spans and redaction processors for sensitive content.

## Alternatives considered

- Vendor SDKs: quicker dashboards, lock-in.

## Consequences

- **Positive:** Vendor-neutral, portfolio-relevant AI observability.
- **Negative:** Requires a collector and a backend (Jaeger locally).
