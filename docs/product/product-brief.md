# Product Brief: AI Reliability Studio

> Status: **Draft for approval** · Milestone: M0 — Product & Engineering Foundation · Owner: @amandavictoriacouto
> Items marked **[DECISION]** need explicit approval. Items marked **[HYPOTHESIS]** are unvalidated.

## 1. One-liner

AI Reliability Studio helps product teams ship digital and AI-native features with **verified** security, privacy, governance and compliance requirements, by turning risk work into requirements, automated checks, auditable evidence and policy-based release decisions.

## 2. Problem

Product teams decide *what* ships and *when*, but risk work usually happens elsewhere and late:

- **Risk enters late.** Security, privacy and AI-governance reviews typically happen near release, when changing scope is expensive. **[HYPOTHESIS H2]**
- **Product people lack a usable language for risk.** GRC tooling speaks in frameworks and clauses; PMs speak in features, users and outcomes. **[HYPOTHESIS H1]**
- **Assurance is asserted, not demonstrated.** Many reviews end in a document or a checkbox ("reviewed by security") with no link between a requirement, how it was verified, and the evidence. **[HYPOTHESIS H4]**
- **AI features add new failure modes** (prompt injection, sensitive data disclosure, excessive agency, misinformation) that classic AppSec checklists do not cover. *Evidence:* OWASP Top 10 for LLM Applications 2025 ([OWASP GenAI](https://genai.owasp.org/2024/11/17/owasp-reveals-updated-2025-top-10-risks-for-llms-announces-new-llm-project-sponsorship-program-and-inaugural-sponsors/)).
- **Regulation is real but moving.** The EU AI Act high-risk obligations were postponed by the Digital Omnibus political agreement (Annex III to 2 Dec 2027, Annex I to 2 Aug 2028, per [Gibson Dunn](https://www.gibsondunn.com/eu-ai-act-omnibus-agreement-postponed-high-risk-deadlines-and-other-key-changes/); publication status must be re-verified before we rely on it). Teams need to design for obligations whose details still shift.

## 3. Target users

| Segment | Role | Primary job-to-be-done |
|---|---|---|
| **Primary** | Product Manager / Product Owner / AI PM | "Before I commit to a launch, show me which risks this feature carries, what we must do about them, and whether it's actually done." |
| Secondary | Product / Software Engineer | "Tell me exactly which requirements apply to my change and let my pipeline prove them automatically." |
| Secondary | Security Engineer | "Get involved at design time, with structured context, and stop being the last-minute gate." |
| Secondary | GRC / Privacy / Compliance | "Get traceable evidence per requirement that I can reuse for audits, without chasing teams." |

**Ideal first customer [HYPOTHESIS]:** a 50–500 person software company (Brazil or EU-exposed) shipping its first LLM-powered features, with a small or no dedicated security team and an upcoming ISO/IEC 27001 / 42001 or enterprise-customer security review.

## 4. Positioning

**Shift-left Risk Intelligence for Product Teams.**

- **For** product teams building digital and AI-native products
- **who** must ship fast while proving risks are managed,
- **AI Reliability Studio** is a risk-to-release workspace
- **that** connects product context → risks → controls → automated verification → evidence → release decision,
- **unlike** GRC/AI-governance platforms (organisation-level, built for compliance teams) and AI security testing tools (technical, built for AppSec),
- **we** operate at the level of a *feature or release*, in the PM's vocabulary, with engineering-grade verification.

## 5. Product principles

1. **Verified over asserted.** Every result states: requirement → verification method → result → evidence → date → who/what verified it.
2. **Honest assurance language.** We never say "secure" or "compliant". We say "N of M mapped requirements verified; K attested; J not verified", with scope and limitations.
3. **AI suggests, humans decide.** AI output is always a *suggestion* with provenance (model, prompt version, inputs). Acceptance, risk ratings and approvals are human acts recorded in the audit trail.
4. **Product language first, rigour underneath.** PM-facing views use feature/outcome language; every item drills down to framework references and technical detail.
5. **Compliance as code.** Risk catalog, control catalog, mappings and release policies are versioned files, reviewed via pull requests.
6. **Dogfooding.** AI Reliability Studio is the first product assessed in AI Reliability Studio. Its own release gate guards its own releases.

## 6. Scope evolution (from the vision to the MVP)

| Capability (vision) | MVP (v0.1) | Later |
|---|---|---|
| Product Context Management | ✅ Products, features/releases, AI system profile, data categories | Imports from Jira/Linear/docs |
| AI-Assisted Risk Discovery | ✅ Catalog-grounded suggestions, human accept/reject | Agentic discovery over repos/docs |
| Risk Assessment | ✅ Likelihood × impact, rationale, residual risk | Quantitative models, portfolio views |
| Control Mapping | ✅ Own control catalog mapped to OWASP LLM 2025, NIST AI RMF, ISO 42001/27001 *references*, LGPD articles | EU AI Act classification assistant, more frameworks |
| Automated Control Verification | ✅ 2 verifier types: GitHub repo posture + LLM endpoint probe suite; minimal GitHub Actions CI-results import (dependency and secret scans) with verified provenance | IaC, cloud, broader SAST/DAST imports |
| Evidence Management | ✅ Automated + uploaded + attestations, SHA-256, expiry | Evidence reuse across frameworks, auditor portal |
| Mitigation Backlog | ✅ Mitigation items, export to GitHub Issues | Two-way sync Jira/Linear |
| Human Review & Approval | ✅ Role-based approval of risks and releases | Multi-step workflows, delegation |
| Policy-Based Release Gates | ✅ Declarative policy + GitHub Action/Check | Other CI systems, runtime gates |
| Audit Trail & Reporting | ✅ Append-only, hash-chained log; Release Assurance Report | Customisable reports, Trust Center |
| Continuous Monitoring | ⏳ Scheduled re-verification + evidence expiry (v0.2) | Runtime telemetry, drift detection |

## 7. Success metrics

**North-star [DECISION]:** *Releases decided with verified evidence* — number of release decisions where every blocking requirement has a verification result linked to evidence.

| Type | Metric | MVP target (initial, to recalibrate) |
|---|---|---|
| Activation | Time from sign-up to first Release Assurance Report | < 30 min with sample product |
| AI quality | Precision of AI-suggested risks (accepted ÷ suggested) | ≥ 60% on eval set; recall ≥ 80% on golden set |
| Assurance | % gate decisions fully traceable to evidence | 100% (hard requirement) |
| Shift-left | % risks identified before "in development" status | Tracked from day 1, baseline unknown |
| Reliability | Gate API p95 latency / availability | < 500 ms / 99.5% (demo environment) |
| Portfolio | Public repo with ADRs, CI green, live demo, case study | v0.1 release |

## 8. Non-goals (explicit)

- Not a certification body, legal advisor or replacement for an auditor.
- Not a full GRC suite (no vendor risk management, policy authoring suite, or HR controls in MVP).
- Not a runtime AI firewall/guardrail product.
- No reproduction of copyrighted standard text (ISO/IEC): we reference identifiers and write our own summaries.

## 9. Key constraints

- Solo builder with AI pair-engineering; capacity is the binding constraint → narrow vertical slice first.
- Public repository: no secrets, no customer data, no licensed standard text in the repo.
- English (en-US) only for product UI, code and documentation in v0.1; the frontend is internationalization-ready for pt-BR later (decision Q4, approved).
