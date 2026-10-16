# Architecture Decision Records

We record significant product-architecture decisions as ADRs (MADR-inspired format, see [template](0000-template.md)).

Lifecycle: **Proposed** (PR opened, labeled `type:adr`) → **Accepted** / **Rejected** (approved by the code owner) → **Superseded** (by a newer ADR). ADRs are never deleted.

| ADR | Title | Status |
|---|---|---|
| [ADR-0001](0001-modular-monolith.md) | Modular monolith with an async worker | Proposed |
| [ADR-0002](0002-monorepo.md) | Monorepo layout | Proposed |
| [ADR-0003](0003-postgres-tenant-isolation.md) | PostgreSQL as system of record with row-level tenant isolation | Proposed |
| [ADR-0004](0004-postgres-job-queue.md) | PostgreSQL-backed job queue | Proposed |
| [ADR-0005](0005-llm-provider-abstraction.md) | Provider-agnostic LLM gateway | Proposed |
| [ADR-0006](0006-declarative-policy-engine.md) | Declarative YAML release policies | Proposed |
| [ADR-0007](0007-content-addressed-evidence.md) | Content-addressed, write-once evidence storage | Proposed |
| [ADR-0008](0008-hash-chained-audit-log.md) | Append-only, hash-chained audit log | Proposed |
| [ADR-0009](0009-authentication.md) | Authentication with Auth.js, BFF and GitHub OIDC for CI (Q5 approved, text pending acceptance) | Proposed |
| [ADR-0010](0010-rbac.md) | Role-based access control per workspace | Proposed |
| [ADR-0011](0011-sandboxed-verifiers.md) | Sandboxed verifier execution | Proposed |
| [ADR-0012](0012-opentelemetry.md) | OpenTelemetry for observability | Proposed |
| [ADR-0013](0013-contract-first-api.md) | Contract-first REST API with generated client | Proposed |
| [ADR-0014](0014-tooling.md) | Language tooling | Proposed |
| [ADR-0015](0015-local-development.md) | Docker Compose for local development | Proposed |
| [ADR-0016](0016-hosting.md) | Hosting target: free tiers for a non-commercial project (Q6 approved, text pending acceptance) | Proposed |
