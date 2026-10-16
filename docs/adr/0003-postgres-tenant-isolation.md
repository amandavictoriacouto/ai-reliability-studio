# ADR-0003: PostgreSQL as system of record with row-level tenant isolation

- Status: Proposed
- Date: 2026-10-15
- Updated: 2026-10-16 (explicit RLS requirements, roles, tenant context and tests; approved revision)
- Deciders: product owner (@amandavictoriacouto)
- Related: [MVP architecture](../architecture/mvp-architecture.md), [threat model](../security/threat-model.md), [detailed design](../security/tenant-isolation-and-authz.md)

## Context

Customer product context is confidential. Cross-tenant leakage is a top threat (T02).

## Decision

Use PostgreSQL; every tenant-owned row carries `workspace_id`; isolation is enforced in the repository layer and by PostgreSQL Row-Level Security as defense in depth; cross-tenant tests run in CI. These requirements are part of the decision ([detailed design](../security/tenant-isolation-and-authz.md) §4–5):

1. **Forced RLS on every tenant table.** `ENABLE` and `FORCE ROW LEVEL SECURITY`, with `USING` and `WITH CHECK` policies on `workspace_id = current_setting('app.workspace_id', true)::uuid`. An unset context matches no rows (fail closed).
2. **Database roles with least privilege.**

   | Role | Purpose | Constraints |
   |---|---|---|
   | `ars_migrator` | Owns tables; runs migrations only | Never used by the running application |
   | `ars_app` | API request path | Not owner, no `BYPASSRLS`, no `UPDATE`/`DELETE` on audit events |
   | `ars_worker` | Job execution | Same tenant grants as `ars_app`, no `BYPASSRLS` |
   | `ars_queue` | Claims and completes jobs | Queue table only, no tenant-table grants ([ADR-0004](0004-postgres-job-queue.md)) |
   | `ars_readonly_ops` | Break-glass read access | Audited use only |

3. **Transaction-scoped tenant context.** Every unit of work runs `SET LOCAL app.workspace_id` (and `app.user_id`) inside its transaction, never session-level `SET`, so pooled connections cannot leak context between requests.
4. **Composite foreign keys.** Tenant-owned references use `(workspace_id, id)`, so a row in one workspace cannot reference a row in another.
5. **Narrow exceptions.** Cross-workspace reads (for example "list my workspaces") use reviewed `SECURITY DEFINER` functions with a fixed `search_path`, filtered by `app.user_id`.
6. **Isolation tests are required in CI** (see below). A change that weakens them needs a new ADR.

## Required tests

- **Migration test:** fails if any table with a `workspace_id` column lacks enabled and forced RLS and a tenant policy.
- **Grant test:** fails if `ars_app` or `ars_worker` owns a table, has `BYPASSRLS`, or can update or delete audit events.
- **Cross-tenant API test:** generated for every operation in the OpenAPI spec; access to another workspace's resources returns 404.
- **SQL tests:** with the context unset or set to another workspace, reads return 0 rows and inserts fail.
- **Composite key test:** inserting a cross-workspace reference fails.
- **Human review:** CODEOWNERS review of migrations that touch roles, grants, policies or `SECURITY DEFINER` functions.

## Alternatives considered

- Schema-per-tenant: stronger separation, but heavier migrations and operations.
- Database-per-tenant: strongest, but disproportionate for MVP.

## Consequences

- **Positive:** Simple model, testable isolation, RLS catches application bugs.
- **Negative:** RLS requires careful transaction handling and a policy on every table. A buggy `SECURITY DEFINER` function or a misused privileged role bypasses RLS; these are reviewed and tested. Free managed PostgreSQL providers must support custom roles and forced RLS (checked under [ADR-0016](0016-hosting.md)).
