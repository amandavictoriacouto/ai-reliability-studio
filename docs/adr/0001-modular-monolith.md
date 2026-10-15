# ADR-0001: Modular monolith with an async worker

- Status: Proposed
- Date: 2026-10-15
- Deciders: product owner (@amandavictoriacouto)
- Related: [MVP architecture](../architecture/mvp-architecture.md), [threat model](../security/threat-model.md)

## Context

A solo builder must deliver an end-to-end product with strong integrity guarantees and low operational cost. Long-running work (AI discovery, verifiers, reports) must not block the API.

## Decision

Build one FastAPI backend with strict internal module boundaries (enforced by import-linter) and a worker process from the same codebase for long-running jobs.

## Alternatives considered

- Microservices: independent scaling, but high operational and cognitive cost for MVP.
- Serverless functions: cheap at idle, but awkward for long jobs and local development.

## Consequences

- **Positive:** One deployable, one test suite, simple local development.
- **Positive:** Clear seams allow extracting the verifier runner later.
- **Negative:** Module discipline must be enforced in CI, otherwise boundaries erode.
