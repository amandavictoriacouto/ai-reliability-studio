# ADR-0013: Contract-first REST API with generated client

- Status: Proposed
- Date: 2026-10-15
- Deciders: product owner (@amandavictoriacouto)
- Related: [MVP architecture](../architecture/mvp-architecture.md), [threat model](../security/threat-model.md)

## Context

Python backend and TypeScript frontend must stay in sync.

## Decision

FastAPI publishes OpenAPI; a TypeScript client is generated (openapi-typescript); CI checks for breaking changes and runs Schemathesis.

## Alternatives considered

- GraphQL: flexible queries, more tooling.
- tRPC: TypeScript-only.

## Consequences

- **Positive:** Single source of truth for the API.
- **Negative:** Generation step in the build.
