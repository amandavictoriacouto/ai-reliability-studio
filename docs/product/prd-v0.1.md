# PRD v0.1: Release Readiness for AI-native Features

> Status: **Draft for approval** · Target release: v0.1 (end of M5)
> Scope principle: one end-to-end vertical slice — *from product context to a release decision backed by evidence* — rather than breadth across all modules.

## 1. Problem statement

A product team about to ship an LLM-powered feature cannot answer, in one place and with evidence: *Which risks does this feature carry? Which requirements mitigate them? Which of those are actually verified, and can we release?*

## 2. Goals and non-goals

### Goals (v0.1)

- G1. A PM can describe a feature and receive a catalog-grounded risk list in minutes.
- G2. Each accepted risk maps to concrete controls/requirements with framework references.
- G3. At least two classes of controls are verified automatically and produce stored, hashed evidence.
- G4. A policy evaluates a release and returns **pass / fail / needs-approval** in CI, with reasons and evidence links.
- G5. Everything is reconstructable from an append-only audit trail.

### Non-goals (v0.1)

 continuous runtime monitoring, Jira/Linear sync, EU AI Act risk classification wizard, multi-language UI (English only, i18n-ready per decision Q4), SSO/SAML, billing.

## 3. Personas

- **Paula, AI PM (primary)** — owns a "support-reply assistant" feature; needs launch confidence and a story for leadership.
- **Eduardo, Product Engineer** — wants requirements as checks in his pipeline, not as PDFs.
- **Sara, Security Engineer** — wants early, structured context and the power to set blocking policy.
- **Gabi, GRC/Privacy** — wants evidence per requirement, reusable for ISO 27001/42001 and LGPD records.

## 4. Core domain model (shared vocabulary)

| Entity | Definition |
|---|---|
| Workspace | Tenant boundary. All data is workspace-scoped. |
| Product | A product or system under assessment. |
| Release | A versioned change set of a product (e.g., "v2.3 – AI reply assistant"), linked to a repo and commit SHA. |
| System Profile | Structured context: AI usage (model type, provider, RAG, tools/agency), data categories (personal, sensitive per LGPD art. 5 II, children), users, deployment, jurisdictions. |
| Risk | Catalog-referenced risk instance for a release; status: suggested → accepted/rejected; rating (likelihood × impact), rationale, owner. |
| Control | Requirement from our control catalog (e.g., `ARS-LLM-01: untrusted input is isolated from system instructions`), with framework references. |
| Verification | Execution of a method (automated / manual / attestation) against a control → result `passed` / `failed` / `inconclusive` / `not_applicable`. |
| Evidence | Immutable artifact supporting a verification: type, source, collector + version, collected_at, SHA-256, storage URI, valid_until. |
| Mitigation | Backlog item to close a gap; may be exported to GitHub Issues. |
| Policy | Versioned declarative rule set evaluated for a release. |
| Gate Decision | Result of evaluating a policy for (release, commit SHA, policy version) at time T, with reasons and evidence refs; expires. |
| Approval | Human sign-off (risk acceptance, release approval, gate override) with role, justification, timestamp. |
| Audit Event | Append-only, hash-chained record of every state change. |

## 5. User stories and acceptance criteria (MVP)

### E1 — Product context

- **US1.1** As a PM, I create a product and a release and fill a guided System Profile (≤ 15 questions, plain language, help text per question).
  - AC: profile is versioned; every change creates an audit event; incomplete profiles are allowed but flagged.
- **US1.2** As a PM, I can start from a sample product ("AI support assistant") to explore the workflow.

### E2 — AI-assisted risk discovery

- **US2.1** As a PM, I click "Discover risks" and receive suggested risks, each with: catalog ID, plain-language explanation, *why it applies to my profile*, framework references, confidence.
  - AC: suggestions reference **only** existing catalog IDs (schema-validated; invalid IDs are dropped and logged).
  - AC: each suggestion stores model, provider, prompt version, input hash and output.
  - AC: nothing becomes an accepted risk without a human action.
- **US2.2** As a PM, I accept, reject (with reason) or add a risk manually.
- **US2.3** As a Security Engineer, I can adjust a risk rating and the change is attributed to me.

### E3 — Risk assessment

- **US3.1** Rate likelihood (1–5) and impact (1–5) with anchored descriptions; inherent and residual score computed.
- **US3.2** Risk acceptance requires an Approval by a role allowed by policy.

### E4 — Control mapping

- **US4.1** Accepted risks auto-propose controls from catalog mappings; user can add/remove with justification.
- **US4.2** Each control shows framework references (OWASP LLM 2025 ID, NIST AI RMF subcategory, ISO/IEC 27001/42001 clause/control *identifier only*, LGPD article) and its verification methods.

### E5 — Verification & evidence

- **US5.1** Automated verifier **GitHub repository posture**: branch protection on default branch, required reviews, secret scanning / Dependabot alerts enabled, CI workflow present (read-only GitHub App or fine-grained token).
- **US5.2** Automated verifier **LLM endpoint probe suite**: runs a fixed, versioned set of probes (direct/indirect prompt injection, system-prompt leakage, PII echo, unbounded output) against a user-registered endpoint in a sandboxed worker; result with per-probe transcripts as evidence.
  - AC: endpoint must be verified as owned (token/header challenge) before probing; network egress restricted (see threat model).
- **US5.3** Manual evidence upload (PDF, PNG, MD, JSON ≤ 10 MB) and **attestations** (signed human statement). Attestations are always displayed as "attested", never "verified".
- **US5.4** Evidence has a validity window; expired evidence makes the verification "stale".

### E6 — Mitigation backlog

- **US6.1** Failed/unverified controls generate mitigation items with owner and due date; export to GitHub Issues with a back-link.

### E7 — Review, approval & release gate

- **US7.1** Workspace admins define a policy (YAML), e.g. "block if any accepted risk with residual ≥ 15 has an unverified control; require Security approval for residual ≥ 10".
- **US7.2** A GitHub Action calls the Gate API with release + commit SHA and gets pass / fail / needs-approval, a summary, and a link to details; posted as a Check Run.
  - AC: decision bound to commit SHA, policy version and evidence snapshot; expires after configurable TTL; replay for another SHA fails.
- **US7.3** Authorized override with mandatory justification, recorded as an Approval.

### E8 — Audit trail & reporting

- **US8.1** Every state change produces an audit event (actor, action, entity, before/after hash, timestamp, prev-hash).
- **US8.2** **Release Assurance Report** (web + Markdown/PDF export): scope, profile, risks, controls, verification status counts (verified / attested / failed / not verified), evidence index with hashes, approvals, gate decision, limitations statement.

## 6. AI requirements

- Provider-agnostic interface; at least 2 providers configurable; deterministic fallback (rules-only suggestions) if LLM unavailable.
- Structured output validated against Pydantic schemas; temperature low; max tokens bounded.
- Grounding: model receives the System Profile + relevant catalog entries; it cannot invent controls.
- Prompts versioned in repo; changes trigger the eval suite in CI.
- **Eval set:** ≥ 20 synthetic system profiles with expert-labelled expected risks; report precision/recall per prompt version.
- Data handling: profile content sent to the LLM is minimised; configurable "no LLM" mode per workspace; provider data-retention settings documented.

## 7. Non-functional requirements

| Area | Requirement |
|---|---|
| Security | OWASP ASVS L2 as the baseline target for the app itself; tenant isolation tests; secrets in a secret manager only. |
| Privacy | LGPD-aligned: data inventory, purpose, retention, deletion on workspace removal; no personal data required to use the demo. |
| Integrity | Evidence immutable (content-addressed by SHA-256); audit log hash-chained and verifiable via endpoint/CLI. |
| Observability | OpenTelemetry traces/metrics/logs, including LLM calls (latency, tokens, cost, prompt version). |
| Performance | Gate API p95 < 500 ms (decision from precomputed state); discovery < 60 s. |
| Accessibility | WCAG 2.2 AA for core flows. |
| Language | en-US only; all user-facing strings externalized in message catalogs (i18n-ready); regulatory terms and identifiers kept in their official form. |
| Quality | Backend ≥ 80% line coverage on domain modules; E2E for the golden path. |

## 8. Release criteria for v0.1

- Golden path demo works end-to-end on the sample product and on AI Reliability Studio itself (dogfooding).
- Threat-model mitigations marked "MVP" are implemented and tested.
- Gate runs in the project's own CI and blocks a release on purpose in a recorded demo.
- Public docs: README, architecture, ADRs, limitations statement.

## 9. Open questions (see [decisions-and-open-questions.md](decisions-and-open-questions.md))

Q-P1 primary demo scenario; Q-P2 frameworks depth in v0.1; Q-P4 open-source license and business model. (Q-P3 UI language: resolved, English only.)
