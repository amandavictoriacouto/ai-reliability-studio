# GitHub Structure

How the project uses GitHub as the single source of truth for code, documentation, decisions, planning and history.

## Milestones

| Milestone | Outcome | Release |
|---|---|---|
| M0 — Product & Engineering Foundation | Approved plan, ADRs, governance, CI skeleton, scaffold | `v0.0.1` |
| M1 — Walking Skeleton | Sign-in, workspaces, product context, audit log, deployed | `v0.0.2` |
| M2 — Risk Discovery & Assessment | Catalog-grounded AI suggestions, scoring, evals | `v0.0.3` |
| M3 — Controls, Evidence & Mitigation | Control mapping, evidence store, backlog | `v0.0.4` |
| M4 — Automated Verification | GitHub posture + LLM probe verifiers | `v0.0.5` |
| M5 — Approval, Release Gate & Reporting (MVP) | Policy gate in CI, assurance report | `v0.1.0` |
| M6 — Continuous Assurance | Re-verification, expiry, drift | `v0.2.0` |

Milestone descriptions copy the outcome and exit criteria from the [roadmap](../roadmap.md).

## GitHub Project (v2): "AI Reliability Studio"

- **Status:** Backlog → Ready → In progress → In review → Done
- **Fields:** Milestone · Area · Type · Priority · Size (XS/S/M/L) · Decision needed (Yes/No)
- **Views:** Board (by status) · Roadmap (by milestone) · Decisions (filter `status:needs-decision`) · Security (filter `type:security`)
- **Automation:** new issues → Backlog; PR opened → In review; PR merged / issue closed → Done.

## Labels

Defined in [`.github/labels.yml`](../../.github/labels.yml): `type:*`, `area:*`, `priority:p0–p3`, `status:needs-decision`, `status:blocked`, `good first issue`.

## Issue templates

Feature (user story) · Bug · ADR proposal · Product decision · Risk/threat · Research. Blank issues disabled; vulnerabilities go to private reporting.

## Pull requests

Conventional Commit titles, PR template with Before/After, How, checklist and Security & AI impact section; squash merge; every PR closes at least one issue.

## Branch protection (`main`)

PR required · 1 approving review from code owner · required checks: `docs` (M0), then app CI · linear history · no force-push · no deletion. Signed commits recommended.

## Actions (by milestone)

| Workflow | Introduced | Purpose |
|---|---|---|
| `docs.yml` | M0 | Markdown lint, link check, secret scan |
| `ci-api.yml`, `ci-web.yml` | M0.6 | Lint, type-check, tests |
| `catalog.yml` | M0.7 | JSON Schema validation of `catalog/` |
| `security.yml` | M0.6 | CodeQL, dependency audit, Trivy |
| `evals.yml` | M2 | AI evals (fixtures on PR, live nightly) |
| `release.yml` | M1 | Changelog, SBOM, GitHub Release |
| `gate.yml` | M5 | Dogfooding: the Studio gates its own releases |

## Repository settings

Private vulnerability reporting · secret scanning + push protection · Dependabot alerts and updates · CodeQL (when code exists) · Discussions (optional) · description, topics (`ai-governance`, `ai-security`, `product-management`, `compliance`, `llm`, `fastapi`, `nextjs`).

## Releases

SemVer tags per milestone; release notes link milestone, demo, ADRs and changelog.
