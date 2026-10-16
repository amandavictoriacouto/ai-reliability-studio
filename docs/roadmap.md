# Development Roadmap

> Status: **Draft for approval** · Milestones are outcome-based; durations are not committed because capacity (solo builder, part-time) is unknown. Each milestone ends with a GitHub Release, a demo recording and a short write-up (portfolio + content).

| Milestone | Outcome (what is true at the end) | Main deliverables | Exit criteria | Release |
|---|---|---|---|---|
| **M0 — Product & Engineering Foundation** | We know what we build, why, and how; the repo is ready for safe incremental work. | Brief, discovery, PRD, architecture + ADRs, threat model, roadmap, repo scaffold, CI skeleton, contribution docs, interview guide | All M0 docs approved and merged; CI green on empty scaffold; Project board populated for M1 | `v0.0.1` (docs) |
| **M1 — Walking Skeleton** | A user can sign in, create a workspace, product and release, fill a system profile; every change is audited; deployed. | Auth (Auth.js + BFF, Q5), workspaces/RBAC, context module, audit log with hash chain, OTel baseline, Docker Compose, free-tier deploy pipeline (Q6; deploys only with explicit approval) | E2E golden path (sign in → profile) green; tenant isolation tests; demo URL live on free tiers, or a documented local-only demo where a free tier would weaken a control | `v0.0.2` |
| **M2 — Risk Discovery & Assessment** | A PM gets catalog-grounded risk suggestions and rates them. | Risk catalog v0 (OWASP LLM 2025 + NIST AI RMF refs), AI gateway (2 providers), discovery job, accept/reject, scoring, eval harness + golden set | Eval precision ≥ 60%, recall ≥ 80% on golden set; injection cases pass; provenance stored | `v0.0.3` |
| **M3 — Controls, Evidence & Mitigation** | Accepted risks map to controls; evidence can be uploaded/attested; gaps become backlog items. | Control catalog + mappings (ISO refs as identifiers only, LGPD articles), evidence store (hash, expiry), attestations, mitigation backlog, GitHub Issues export | Evidence immutability tests; dogfooding: Studio's own threat model loaded | `v0.0.4` |
| **M4 — Automated Verification** | Two verifier types produce evidence automatically. | Verifier plugin interface, GitHub posture verifier (GitHub App), LLM probe suite verifier with SSRF guard + ownership check | SSRF test suite green; verifications reproducible; evidence linked | `v0.0.5` |
| **M5 — Approval, Release Gate & Reporting (MVP)** | A release can be evaluated in CI with evidence-backed pass/fail/needs-approval. | Policy engine (YAML), approvals/overrides, Gate API, GitHub Action + Check Run, Release Assurance Report (web/MD/PDF), ZAP baseline | Studio's own CI is gated by Studio; recorded demo of a blocked then fixed release; MVP threat mitigations done | **`v0.1.0` (MVP)** |
| **M6 — Continuous Assurance** | Assurance stays current after release. | Scheduled re-verification, evidence expiry alerts, drift notifications, dashboard trends | Stale evidence surfaces automatically; notification tests | `v0.2.0` |
| **Later (candidates)** | — | EU AI Act classification helper, LGPD RIPD/DPIA helper, Jira/Linear sync, more verifiers (IaC, cloud), pt-BR UI (frontend is i18n-ready per Q4), SSO, Trust Center | Prioritised after MVP validation interviews | — |

## Parallel discovery track (runs from M0 to M5)

Full plan, decision points and research privacy rules: [discovery track](product/discovery-track.md).

- M0: interview guide + recruiting 10–12 PMs/AI PMs, 5 security/GRC.
- M1–M2: interviews; validate H1–H3, H7.
- M3: mock report test for H4.
- M5: gate usability test for H5; decision memo: continue / refocus.

## Portfolio & content track (leverage)

Each milestone yields: 1 GitHub Release note, 1 ADR highlight, 1 LinkedIn post (decision + trade-off), 1 short video (demo), and feeds a long-form case study at v0.1.0.
