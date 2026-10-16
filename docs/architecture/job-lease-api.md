# Job Queue and Job-Lease API

> Status: **Proposed** (architecture review, M0) · Related: [ADR-0004](../adr/0004-postgres-job-queue.md), [ADR-0011](../adr/0011-sandboxed-verifiers.md), [endpoint verifier security](../security/endpoint-verifier-security.md), [tenant isolation](../security/tenant-isolation-and-authz.md) §5, [threat model](../security/threat-model.md) T02, T03, T09
> Goal: background work runs with the **least privilege** that the job needs. The general worker uses the database under tenant context; the isolated verifier runner never touches the database and receives work only through short-lived, job-scoped leases.

## 1. Two kinds of executors

| Executor | Runs | Database access | How it gets work |
|---|---|---|---|
| **Worker** (same codebase as the API) | AI discovery, GitHub posture checks, report generation, retention, checkpoints | Yes, as `ars_worker` (no `BYPASSRLS`), always under `SET LOCAL app.workspace_id` | Claims jobs from the queue table with `SELECT … FOR UPDATE SKIP LOCKED` |
| **Verifier runner** (separate process type and network segment) | LLM endpoint probes, and any future verifier that contacts customer-chosen targets | **None.** No database, storage, provider or signing credentials | Calls the **Job-Lease API** on the API service (§3) |

## 2. Queue table: metadata only

The queue table (`jobs`) is a **platform table**. It holds only what scheduling needs, never customer content or secrets.

| Column | Notes |
|---|---|
| `job_id` (UUIDv7), `job_type`, `queue` | `queue` separates worker jobs from runner jobs |
| `workspace_id` | Tenant the job belongs to. Used to set the execution context, never to filter results across tenants |
| `subject_ref` | Typed reference to the domain object (e.g., `endpoint:<id>`, `release:<id>`). No payload content |
| `requested_by`, `required_permission` | Actor and permission checked at enqueue time and **re-checked** at execution time (§4) |
| `idempotency_key` | Unique per `(workspace_id, job_type, idempotency_key)`. Re-enqueueing the same work returns the existing job |
| `status`, `attempts`, `max_attempts`, `run_after` | `queued → leased → succeeded / failed / cancelled / dead` |
| `lease_id`, `lease_generation`, `lease_expires_at`, `leased_by` | Current lease. `lease_generation` is a fencing token that increases on every new lease |

**Access model (F7):**

- `ars_app` (API request path) can only **insert** jobs, through a `SECURITY DEFINER` function `enqueue_job(...)` with fixed `search_path`. The function takes `workspace_id` from `current_setting('app.workspace_id')`, so a request can only enqueue jobs for the workspace it is acting in. `ars_app` has no `SELECT`, `UPDATE` or `DELETE` on `jobs`.
- `ars_queue` (dispatcher inside the worker and the Job-Lease API) can claim, extend, complete and fail jobs. It has **no grants on tenant tables**.
- `ars_worker` (job execution) has the same tenant-table grants as `ars_app`, no `BYPASSRLS`, and no grants on `jobs`.
- RLS is enabled and forced on `jobs` with a deny-by-default policy, so the generic RLS migration test (all tables with `workspace_id` have RLS) still holds. Only the two roles above have policies.

Enqueueing stays **transactional** with the domain write: the API calls `enqueue_job` inside the same transaction, so a job exists only if the triggering change committed.

## 3. Job-Lease API (verifier runner)

Internal endpoints on the API service, reachable only from the runner network segment (not public, not from browsers).

| Endpoint | Purpose |
|---|---|
| `POST /internal/runner/leases` | Claim one job of a type the runner pool is allowed to run. Returns a lease or `204` if none |
| `POST /internal/runner/leases/{lease_id}/heartbeat` | Extend the lease (bounded, below). Response can carry `cancel` |
| `POST /internal/runner/leases/{lease_id}/results` | Submit probe results and the raw transcript. Idempotent per `(lease_id, sequence)` |
| `POST /internal/runner/leases/{lease_id}/complete` / `fail` | Finish the job with a final status |

### 3.1 Credentials

| Credential | Held by | Scope | Lifetime |
|---|---|---|---|
| **Runner pool credential** | Runner process, from the platform secret manager | `lease:claim` for the allowed `job_type`s only. Cannot read or write any domain data | Rotated at least every 90 days; revocable instantly |
| **Job-scoped token** (returned with the lease) | One runner process, for one job | `aud=ars-runner-api`, `sub=lease_id`, claims `job_id`, `workspace_id`, `lease_generation`, `scope=results:submit` | Equal to the lease expiry (≤ 15 min), never refreshed beyond the job's wall-clock limit |
| **Endpoint test credential** (if the customer supplied one) | Delivered inside the lease response only | One target, one job | In memory for the job; never logged, never returned in results |

The job-scoped token is signed by the API with a key used for nothing else. The runner cannot use it to claim other jobs, read data, or act for another workspace.

### 3.2 Authorization at lease time

Before a lease is issued, the API re-checks, inside a transaction with `SET LOCAL app.workspace_id = job.workspace_id`:

1. The endpoint is still **verified** and **authorized** (not expired, not revoked) per [endpoint verifier security](../security/endpoint-verifier-security.md) §2.
2. `requested_by` still has `required_permission` in the workspace.
3. Kill switches (global, workspace, endpoint) are off, and per-workspace and per-endpoint concurrency and daily limits allow the run.

If any check fails, the job is cancelled with a reason and an audit event. The lease payload contains only what the job needs: normalized target, pinned IP policy, probe suite version, limits, and the optional test credential.

### 3.3 Lease expiry, fencing and idempotency

- Lease duration is the job's wall-clock limit plus a margin (≤ 15 min). Heartbeats extend it in steps of ≤ 60 s, but never past the hard wall-clock limit.
- An expired lease returns the job to `queued` with `attempts + 1`, until `max_attempts` (default 3), then `dead`.
- Every write from the runner carries `lease_generation`. The API rejects writes whose generation is not the current one, so a stale runner that wakes up after its lease expired cannot overwrite the new attempt's results.
- Result submissions are idempotent per `(lease_id, sequence)`. The API computes evidence hashes itself and ignores any client-supplied hash, consistent with the [evidence model](evidence-model.md).

### 3.4 Tenant isolation

- The API derives `workspace_id` from the lease record, never from the request body. Any body field that names a workspace, job or endpoint must match the lease, or the request is rejected.
- Results are stored by the API under `SET LOCAL app.workspace_id`, so RLS applies exactly as for user requests.

### 3.5 Execution limits

Limits are enforced on **both sides**: the runner enforces timeouts, rate, response size and request count locally; the API enforces maximum results per lease, maximum transcript size, and rejects submissions after the lease's wall-clock deadline. Platform maximums are in [endpoint verifier security](../security/endpoint-verifier-security.md) §5.

## 4. Worker execution (database jobs)

1. The dispatcher (`ars_queue`) claims a job with `SKIP LOCKED` and sets a lease, using the same fencing and expiry rules as §3.3.
2. Execution opens a **new transaction** as `ars_worker` and runs `SET LOCAL app.workspace_id = job.workspace_id`, `SET LOCAL app.user_id = job.requested_by`.
3. It re-checks `required_permission` for `requested_by` before doing any work. A revoked membership cancels the job.
4. Platform jobs that are not tenant-scoped (retention, audit checkpoints) use their own dedicated roles with minimum grants, as listed in [tenant isolation](../security/tenant-isolation-and-authz.md) §5.

## 5. Tests

All automated:

- `ars_app` cannot select, update or delete jobs, and cannot enqueue a job for another workspace.
- `ars_queue` cannot read any tenant table.
- A job-scoped token cannot submit results for another job, workspace or lease generation.
- A stale lease (expired, then re-leased) cannot write results.
- Lease is refused after endpoint revocation, permission removal or kill switch.
- Duplicate result submission is idempotent; duplicate enqueue returns the existing job.
- The runner image contains no database or storage credentials (container policy check).

## 6. Status

This is a **planned** design. Nothing in this document is implemented yet.
