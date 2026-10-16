# Tenant Isolation, RLS and Frontend–Backend Authorization

> Status: **Proposed** (architecture review, M0) · Related: [ADR-0003](../adr/0003-postgres-tenant-isolation.md), [ADR-0009](../adr/0009-authentication.md), [ADR-0010](../adr/0010-rbac.md), [threat model](threat-model.md) T02, T15, T17

## 1. Review findings on the initial architecture

| # | Finding | Change proposed |
|---|---|---|
| F1 | The original draft let the API validate a JWT, but did not say who mints it or where it lives. If the browser holds API tokens, an XSS bug could steal them. | **Backend-for-Frontend (BFF):** the browser only holds an httpOnly session cookie. Only the Next.js server calls the API (§3). |
| F2 | It was not specified whether roles travel in the token. Roles in a token stay valid until the token expires, even after a role is removed. | Tokens carry **identity only**. The API loads membership and role from the database on every request. |
| F3 | RLS with connection pooling can leak tenant context between requests if `SET` is used at session level. | Use transaction-scoped `SET LOCAL` only. The application role is not the table owner and has no `BYPASSRLS`. `FORCE ROW LEVEL SECURITY` is on (§4). |
| F4 | Workers and cross-workspace operations (e.g., "list my workspaces", retention jobs) were not covered. | Explicit, narrow paths for these, including the queue table's own roles (§5). |
| F5 | Cross-tenant references (a risk in workspace A pointing to a control in workspace B) were possible via foreign keys. | **Composite foreign keys** `(workspace_id, id)` (§4). |
| F6 | Next.js data caching could serve one tenant's data to another. | No shared caching of authenticated data (§6). |

## 2. Tenancy model

- **Workspace = tenant.** A user can belong to several workspaces with different roles.
- Every tenant-owned table has `workspace_id uuid NOT NULL`. Global tables (catalog versions, framework references) are read-only to the app role.
- Workspace-scoped API routes have the form `/v1/workspaces/{workspace_id}/…`. The workspace is always explicit, never inferred from a "current workspace" cookie.

## 3. Authentication and the frontend ↔ backend path

```mermaid
sequenceDiagram
  participant B as Browser
  participant N as Next.js server (BFF)
  participant A as FastAPI
  participant D as PostgreSQL
  B->>N: request + session cookie (httpOnly, Secure, SameSite=Lax)
  N->>N: validate Auth.js session
  N->>A: request + short-lived service JWT (sub=user_id, sid, aud=ars-api, exp=60s, ES256)
  A->>A: verify JWT signature (web's public key), aud, exp, sid not revoked
  A->>D: BEGIN; SET LOCAL app.user_id, app.workspace_id; membership/role check
  A->>D: domain query (RLS enforced)
  A-->>N: response
  N-->>B: rendered page / JSON
```

- **Login:** Auth.js with GitHub and Google OIDC (Q5 approved 2026-10-16; [ADR-0009](../adr/0009-authentication.md) remains Proposed until its text is accepted). Sessions are stored server-side (database sessions), so a session can be revoked immediately.
- **Service JWT:** minted per request by the Next.js server with an ES256 key that only the web server holds, valid for 60 seconds. The API trusts only this key, plus GitHub OIDC for the gate endpoint ([release gate binding](../architecture/release-gate-binding.md)).
- **Network:** the API is not reachable from browsers. CORS is disabled, and the only public API endpoints are `/gate/*` and `/.well-known/jwks.json`.
- **CSRF:** SameSite=Lax cookies, CSRF tokens on state-changing route handlers and server actions, and `Origin` header checks.
- **Frontend authorization is UX only.** Hiding a button is not a control. Every check is enforced in the API.
- **Step-up for sensitive actions** (approvals, overrides, endpoint authorization, deletions): the session must have authenticated within the last 10 minutes. GitHub OAuth has no standard `prompt=login` or `max_age`, so the step-up may require a full re-login. This is an open UX detail to settle when ADR-0009 is accepted.

## 4. Database isolation (RLS)

```sql
ALTER TABLE risks ENABLE ROW LEVEL SECURITY;
ALTER TABLE risks FORCE ROW LEVEL SECURITY;
CREATE POLICY tenant_isolation ON risks
  USING (workspace_id = current_setting('app.workspace_id', true)::uuid)
  WITH CHECK (workspace_id = current_setting('app.workspace_id', true)::uuid);
```

- **Roles:** `ars_app` (no ownership, no `BYPASSRLS`), `ars_worker` (same tenant grants as `ars_app`, used by job execution), `ars_queue` (queue table only), `ars_migrator` (owner, used only in migrations), `ars_readonly_ops` (break-glass, audited).
- **Context:** `SET LOCAL` inside each transaction. If `app.workspace_id` is unset, `current_setting(..., true)` returns NULL, so **no rows match**. The system fails closed.
- **Composite foreign keys:** for example, `risks(workspace_id, control_id) → controls(workspace_id, id)`. This makes cross-tenant references impossible at the schema level.
- **Defense in depth:** the repository layer also requires a workspace context object, so a missing filter is a type error in code. RLS catches what the code misses.

## 5. Non-request paths

| Path | Approach |
|---|---|
| "List my workspaces", invitations | `SECURITY DEFINER` functions with fixed `search_path`, filtered by `app.user_id`. They are reviewed as security-sensitive code. |
| Job queue table | Platform table holding **metadata only** (no content, no secrets), with forced RLS and deny-by-default. `ars_app` can only enqueue through a `SECURITY DEFINER` function bound to its current `app.workspace_id`. A dedicated `ars_queue` role claims and completes jobs and has no grants on tenant tables ([job queue and Job-Lease API](../architecture/job-lease-api.md)). |
| Workers (discovery, verification, reports) | The job carries `workspace_id` and `requested_by`. Execution runs as `ars_worker` (no `BYPASSRLS`) in a new transaction with `SET LOCAL app.workspace_id` and `app.user_id` from the job, and re-checks the requester's permission before doing work. No bypass role. |
| Platform jobs (retention, checkpoints) | A dedicated role per job type with minimum grants. Every run is audited. |
| Endpoint verifier runner | No database access at all. Work arrives through the Job-Lease API with job-scoped, short-lived tokens; the API derives the workspace from the lease, never from the request ([endpoint verifier security](endpoint-verifier-security.md)). |
| LLM calls | Prompts include only data from the current workspace. There is no cross-tenant retrieval or embedding index. Provenance records `workspace_id`. |
| Object storage | Keys are prefixed with `workspace_id`. Pre-signed URLs are issued only after the API authorization check. |

## 6. Caching and telemetry

- Authenticated data is fetched with `cache: 'no-store'`. Tenant pages use no ISR or static generation. The CDN does not cache responses that carry `Set-Cookie` or `Authorization`.
- Telemetry carries `workspace_id` as an attribute, never content. Dashboards that show tenant data are internal only.

## 7. Authorization (RBAC) matrix (MVP)

| Permission | owner | admin | security | grc | product | engineer | viewer |
|---|---|---|---|---|---|---|---|
| Manage members & roles | ✅ | ✅ | — | — | — | — | — |
| Edit product context / profile | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | — |
| Run AI discovery | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | — |
| Accept/reject risks | ✅ | ✅ | ✅ | ✅ | ✅ | — | — |
| Accept residual risk (approval) | ✅ | ✅ | ✅ | ✅ | per policy | — | — |
| Upload evidence / attest | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | — |
| Register/authorize endpoints | ✅ | ✅ | ✅ | — | — | register only | — |
| Edit/activate policies | ✅ | ✅ | ✅ | — | — | — | — |
| Approve release / override gate | ✅ | — | ✅ | per policy | per policy | — | — |
| Delete evidence content | ✅ | ✅ | — | — | — | — | — |
| Read everything (metadata) | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |

The matrix lives in code as data and is tested exhaustively (role × permission).

## 8. Validation

| Control | Method |
|---|---|
| Cross-tenant isolation | **Automated:** for every API operation in the OpenAPI spec, a generated test uses two workspaces and asserts that access to the other workspace's resources returns 404. Plus direct SQL tests with RLS context unset or wrong (expect 0 rows and failing inserts). |
| RLS enabled and forced on every tenant table | **Automated:** a migration test queries `pg_class` / `pg_policies` and fails if any table with `workspace_id` lacks RLS. |
| Queue roles: `ars_app` enqueue-only for its own workspace, `ars_queue` without tenant grants | **Automated** grant tests ([job queue and Job-Lease API](../architecture/job-lease-api.md) §5). |
| No `BYPASSRLS` or ownership for `ars_app` | **Automated** role-grant test + **human review** of migrations that touch roles or grants. |
| RBAC matrix | **Automated:** exhaustive matrix test. |
| BFF: no API token in browser | **Automated:** Playwright checks that no `Authorization` headers or tokens appear in browser storage or network calls to the API host. |
| `SECURITY DEFINER` functions | **Human review** (CODEOWNERS on `apps/api/**/migrations`). |

## 9. Residual risks

- A bug in a `SECURITY DEFINER` function or the use of a privileged role bypasses RLS. These are rare, reviewed and audited.
- If the web server's signing key is compromised, an attacker can impersonate any user to the API. Mitigations: key held in a secret manager, 60 s token lifetime, rotation, and an alert on anomalous `sid` usage.
- Noisy-neighbour performance effects in shared tables are an availability issue, not a confidentiality one.
