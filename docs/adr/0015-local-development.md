# ADR-0015: Docker Compose for local development

- Status: Proposed
- Date: 2026-10-15
- Updated: 2026-10-16 (incremental services; approved M1 simplification)
- Deciders: product owner (@amandavictoriacouto)
- Related: [MVP architecture](../architecture/mvp-architecture.md), [threat model](../security/threat-model.md)

## Context

Developers need PostgreSQL, object storage and an OpenTelemetry stack locally.

## Decision

Provide `infra/docker-compose.yml`, adding services only when a milestone needs them (approved M1 simplification):

- **M1:** PostgreSQL only.
- **M3:** MinIO (or another S3-compatible store with object lock and a compatible license) for the evidence store.
- **Optional profile** (`--profile observability`): OpenTelemetry Collector and Jaeger, used when working on tracing.

Docker Compose is also the fallback environment for capabilities that cannot run safely on free hosting tiers ([ADR-0016](0016-hosting.md)).

## Alternatives considered

- Cloud development environments: convenient but cost and network dependent.

## Consequences

- **Positive:** Reproducible and offline-friendly.
- **Negative:** Requires Docker locally.
