# Initial Backlog

Issues to create after the M0 documentation is approved. IDs are provisional (`M0-01` …) and become GitHub issue numbers.

## M0 — Product & Engineering Foundation

| ID | Title | Type | Area | Priority | Depends on |
|---|---|---|---|---|---|
| M0-01 | Decide D1–D11 (scope, positioning, AI design, architecture) | decision | product | p0 | — |
| M0-02 | ~~Decide Q1: code license~~ (approved 2026-10-16: Apache-2.0) | decision | product | p0 | — |
| M0-03 | Decide Q2: primary demo scenario | decision | product | p1 | — |
| M0-04 | Decide Q7–Q8: LLM providers, target market (Q4, Q5 and Q6 approved) | decision | product | p1 | — |
| M0-19 | Decide D12–D17 from the architecture review | decision | product | p0 | — |
| M0-05 | Define capacity and target date for v0.1 (Q9) | decision | product | p1 | — |
| M0-06 | Merge M0 planning documentation | docs | product | p0 | M0-01 |
| M0-07 | Accept/reject ADR-0001…0016 | adr | infra | p0 | M0-01, M0-04 |
| M0-08 | Add Apache-2.0 LICENSE and NOTICE (in PR 1) and license check in CI | chore | infra | p0 | M0-02 |
| M0-09 | Apply labels, milestones, Project board and branch protection | chore | infra | p1 | M0-06 |
| M0-10 | Enable repository security settings (private reporting, secret scanning, push protection, Dependabot) | security | infra | p0 | — |
| M0-11 | Scaffold monorepo: `apps/api` (FastAPI health), `apps/web` (Next.js shell), `infra/docker-compose.yml`, Makefile | feature | infra | p1 | M0-07 |
| M0-12 | CI for apps: lint, type-check, tests; security workflow | chore | infra | p1 | M0-11 |
| M0-13 | Catalog JSON Schemas v0 (risk, control, mapping, policy) | feature | catalog | p1 | M0-07 |
| M0-14 | Interview guide v1, screener and consent form (discovery track week 1) | research | product | p1 | — |
| M0-15 | Recruit 10–12 PMs/AI PMs and 5 security/GRC interviewees | research | product | p2 | M0-14 |
| M0-16 | Verify EU AI Act Digital Omnibus publication status and update docs | research | product | p2 | — |
| M0-17 | Hands-on trials of 3–4 competitors; update competitive discovery | research | product | p2 | — |
| M0-18 | Release `v0.0.1` (M0 close) with release notes | chore | infra | p1 | M0-06…M0-13 |

## M1 — Walking Skeleton (ready to refine)

| ID | Title | Type | Area | Priority |
|---|---|---|---|---|
| M1-01 | PostgreSQL, SQLAlchemy, Alembic, Testcontainers setup | feature | api | p0 |
| M1-02 | Auth.js sign-in with GitHub and Google (per ADR-0009, Q5) | feature | web/api | p0 |
| M1-03 | Workspaces, memberships and RBAC with permission-matrix tests | feature | api | p0 |
| M1-04 | Row-Level Security + cross-tenant isolation tests (T02) | security | api | p0 |
| M1-05 | Hash-chained audit log + verify CLI (T07, T08) | feature | api | p0 |
| M1-06 | Products and releases CRUD | feature | api/web | p1 |
| M1-07 | Versioned System Profile + guided wizard | feature | web | p1 |
| M1-08 | OpenTelemetry across web/api with redaction (T19) | feature | infra | p1 |
| M1-11 | i18n-ready web shell: en-US message catalog, lint rule against hard-coded strings | feature | web | p2 |
| M1-12 | BFF auth path: service JWT, session revocation, no API tokens in browser test | security | web/api | p0 |
| M1-13 | Signed audit checkpoints + daily verification job | security | api | p1 |
| M1-09 | Deploy pipeline + preview environments on free tiers (per ADR-0016, Q6); no paid services, deploy only with explicit approval | chore | infra | p1 |
| M1-10 | Playwright golden path: sign in → system profile | test | web | p1 |

## M2–M6 (epics, to be broken down at milestone start)

- **M2:** Risk catalog v0 · AI gateway · Rules baseline · LLM discovery + review UI · Scoring & acceptance · Eval harness + golden set · Injection evals
- **M3:** Control catalog + mappings · Control auto-proposal · Evidence store · Upload hardening · Attestations · Mitigation backlog + GitHub Issues export · Dogfood threat model
- **M4:** Verifier interface · Queue roles + Job-Lease API + isolated runner · GitHub App + posture verifier · SSRF guard · Endpoint ownership · LLM probe suite · Verification UI
- **M5:** Policy engine · Approvals & overrides · Gate API (OIDC, head-SHA binding + signature) · CI-results import with verified provenance · GitHub Action + Check Run · Assurance report · Self-gating · ZAP baseline · v0.1.0 release
- **M6:** Re-verification scheduler · Evidence expiry · Notifications · Trend dashboard
