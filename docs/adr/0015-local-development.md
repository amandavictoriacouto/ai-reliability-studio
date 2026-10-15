# ADR-0015: Docker Compose for local development

- Status: Proposed
- Date: 2026-10-15
- Deciders: product owner (@amandavictoriacouto)
- Related: [MVP architecture](../architecture/mvp-architecture.md), [threat model](../security/threat-model.md)

## Context

Developers need PostgreSQL, object storage and an OpenTelemetry stack locally.

## Decision

Provide `infra/docker-compose.yml` with PostgreSQL, MinIO, OpenTelemetry Collector and Jaeger.

## Alternatives considered

- Cloud development environments: convenient but cost and network dependent.

## Consequences

- **Positive:** Reproducible and offline-friendly.
- **Negative:** Requires Docker locally.
