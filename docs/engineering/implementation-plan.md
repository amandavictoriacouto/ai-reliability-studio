# Incremental Implementation Plan

> Status: **Draft for approval** · Each increment is a vertical slice: one or more issues → PR(s) → tests → docs/ADR → demo. No increment starts before its predecessor is merged with CI green.

## Working agreement

1. **Decide → document → build.** Product, architecture and scope decisions are proposed as issues/ADRs labeled `status:needs-decision`; the product owner (@amandavictoriacouto) approves before implementation.
2. **Small PRs** (< ~400 changed lines excluding generated code), one concern each.
3. **Test-first for domain logic** (policy engine, scoring, hash chain, SSRF guard).
4. **Security & AI checklist** in every PR (data touched, new trust boundary?, LLM usage?, threats updated?).
5. **Demo at the end of each increment** (GIF or short recording in the PR).

## M0 — Foundation (current)

| # | Increment | Output | Needs approval |
|---|---|---|---|
| 0.1 | Approve planning package (`docs/`) | Decisions log updated | ✅ |
| 0.2 | Repo exists (public). Add license, README, governance files | Repo foundation | ✅ (license) |
| 0.3 | Commit M0 docs into `docs/` + ADR-0001…016 as *Proposed* → *Accepted* | Docs PR | ✅ |
| 0.4 | Labels, milestones, issue/PR templates, Project board, branch protection | GitHub setup | — |
| 0.5 | Monorepo scaffold: `apps/api` (FastAPI hello + health), `apps/web` (Next.js shell), Docker Compose, Makefile | Runnable skeleton | — |
| 0.6 | CI: lint/type/test for both apps, catalog schema job (empty), security scans, Dependabot | Green CI | — |
| 0.7 | Catalog schemas v0 (risk, control, mapping, policy) + licensing notes | `catalog/schemas` | ✅ (catalog content policy) |
| 0.8 | Interview guide + recruitment message (English; outreach copy may be localized outside the repo) per the [discovery track](../product/discovery-track.md) | `docs/product/research/` | — |

## M1 — Walking skeleton

1.1 Postgres + SQLAlchemy + Alembic + Testcontainers · 1.2 Identity: OIDC login, workspaces, memberships, RBAC matrix tests · 1.3 RLS + cross-tenant tests · 1.4 Audit module (hash chain, verify CLI) · 1.5 Context: products, releases, versioned system profile + UI wizard · 1.6 OTel across web/api · 1.7 Deploy pipeline to chosen host + preview environments · 1.8 Playwright golden path.

## M2 — Risk discovery & assessment

2.1 Risk catalog v0 (~25 entries) with OWASP LLM 2025 + NIST AI RMF references · 2.2 AI gateway: provider protocol, 2 adapters, structured outputs, provenance table, budgets · 2.3 Rules-based baseline suggestions (no LLM) · 2.4 LLM discovery job + review UI (accept/reject/add) · 2.5 Scoring + risk acceptance approval · 2.6 Eval harness + 20 golden profiles + CI job · 2.7 Injection test cases for discovery.

## M3 — Controls, evidence & mitigation

3.1 Control catalog v0 (~30 controls) + mappings · 3.2 Auto-proposal of controls from accepted risks · 3.3 Evidence store (S3/MinIO, SHA-256, write-once, expiry) · 3.4 Upload hardening · 3.5 Attestations · 3.6 Mitigation backlog + GitHub Issues export · 3.7 Dogfood: load the Studio's own threat model.

## M4 — Automated verification

4.1 Verifier plugin interface + run lifecycle · 4.2 GitHub App (read-only) + posture verifier · 4.3 SSRF guard library + tests · 4.4 Endpoint registration + ownership challenge · 4.5 LLM probe suite v0 (≈ 15 probes) + transcript evidence · 4.6 Verification UI with verified/attested/failed/not-verified states.

## M5 — Gate & reporting (MVP v0.1.0)

5.1 Policy YAML schema + engine + regression scenarios · 5.2 Approvals & overrides (re-auth) · 5.3 Gate API with decision binding + signature · 5.4 GitHub Action + Check Run · 5.5 Release Assurance Report (web + MD/PDF) with limitations block · 5.6 Gate the Studio's own CI · 5.7 ZAP baseline + threat-model review · 5.8 Release, demo video, case study.

## M6 — Continuous assurance

6.1 Scheduler for re-verification · 6.2 Evidence expiry & stale states · 6.3 Notifications (email/Slack) · 6.4 Trend dashboard.

## Approvals needed before M0.2+

See [`docs/product/decisions-and-open-questions.md`](../product/decisions-and-open-questions.md) → decisions D1–D10 and questions Q1–Q8.
