# ADR-0013: Code-first FastAPI with a generated OpenAPI contract and TypeScript client

- Status: Proposed
- Date: 2026-10-15
- Updated: 2026-10-16 (clarified as code-first; incremental tooling; approved revision)
- Deciders: product owner (@amandavictoriacouto)
- Related: [MVP architecture](../architecture/mvp-architecture.md), [threat model](../security/threat-model.md)

## Context

Python backend and TypeScript frontend must stay in sync.

## Decision

The API is **code-first**: FastAPI routes and Pydantic models are the source of truth, and the **OpenAPI document is generated** from them. The OpenAPI document is then the published contract:

1. The generated `openapi.json` is committed, and CI fails if it differs from what the code generates.
2. A TypeScript client is generated from it with `openapi-typescript`; the web app uses only this client.
3. Introduced incrementally (approved M1 simplification): client generation and the "spec is current" check in M1; a breaking-change check (OpenAPI diff) and Schemathesis property-based tests from M2, once there are endpoints worth fuzzing.
4. The same spec drives the generated cross-tenant isolation tests ([ADR-0003](0003-postgres-tenant-isolation.md)).

## Alternatives considered

- GraphQL: flexible queries, more tooling.
- tRPC: TypeScript-only.

## Consequences

- **Positive:** Single source of truth for the API.
- **Negative:** Generation step in the build. Because the contract is derived from code, API design review happens on the code PR (the spec diff makes it visible).
