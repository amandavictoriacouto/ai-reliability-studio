# Verification Methods: Automated Tests, Human Review and Documentary Evidence

> Status: **Proposed** (architecture review, M0) · Related: [assurance language](assurance-language.md), [evidence model](../architecture/evidence-model.md)
> This document applies at two levels: (A) inside the product, where each control is verified by a declared method; and (B) to how **we** validate the Studio's own security and quality claims (dogfooding).

## 1. The three methods

| Method | Definition | What it can prove | What it cannot prove | Resulting status | Evidence |
|---|---|---|---|---|---|
| **Automated test** | A machine-executed, versioned, repeatable check with explicit pass criteria | That a specific condition held for a specific target at a specific time (or commit) | Anything outside the check's coverage; absence of other defects | `verified` / `failed` | Machine output (hashed), check ID and version, target, commit/time |
| **Human review** | A named, qualified person evaluates an artifact against a **written checklist** and records the outcome | Judgement-based properties (design adequacy, mapping correctness, policy fitness) | Consistency over time; it covers what the reviewer examined, not more | `verified` (method: human_review) / `failed` | Review record: reviewer, role, checklist version, artifact hash, findings, date |
| **Documentary evidence** | A document or statement exists and is attested by a named person | That an organisation **claims** a practice or state (policy, DPA, training) | That the practice is actually followed | `attested`, **never** `verified` | Document (hashed) + attestation (who, role, statement, date) |

**Rules:**

1. Each control declares its **accepted methods** and the **minimum** method a policy may require (e.g., "commit-bound automated test required").
2. LLM-based evaluations (evals, red-team probes) are automated tests with **probabilistic** results. Reports show the measured rate and sample size. A passed eval never implies immunity.
3. AI-generated suggestions are never verification of anything. They become facts only through a human action.
4. Reports show counts **per method** and **per status**, so "12 verified" always reads as, for example, "9 by automated test, 3 by human review".

## 2. Product level (A): default method per control family (MVP catalog)

| Control family (examples) | Default method | Alternative | Notes |
|---|---|---|---|
| Repository posture (branch protection, required reviews, secret scanning, Dependabot) | Automated (GitHub posture verifier) | — | `time`-bound |
| LLM input handling: prompt injection, system-prompt leakage, PII echo, output bounds | Automated (probe suite) | Human review of red-team report | `commit`-bound when the preview reports its SHA |
| Dependency vulnerabilities, secrets in code | Automated (**GitHub Actions CI-results import**, PRD US7.3) | — | `commit`-bound. Counts as `verified` only with **verified provenance**: the upload is authenticated with a GitHub Actions OIDC token from a bound repository, the API records `repository_id`, `run_id`, `run_attempt`, `workflow_ref` and the resolved `head_sha`, and the GitHub App confirms the run belongs to that repository and commit. Results uploaded without verified provenance (e.g., fallback token) are labelled `client-asserted` and stay `not_verified` unless a policy explicitly accepts them |
| Architecture & data-flow adequacy (e.g., untrusted input isolation design) | Human review (security checklist) | — | `release`-bound |
| Human oversight design (NIST AI RMF MANAGE / EU AI Act Art. 14, informational) | Human review | Documentary | — |
| Privacy: legal basis, RIPD/DPIA, data retention configuration | Documentary (attested) | Automated where configuration is inspectable (later) | LGPD articles referenced |
| Vendor / sub-processor agreements | Documentary (attested) | — | — |
| Incident response readiness | Documentary + human review (tabletop record) | — | — |

## 3. Studio level (B): how we validate our own claims

| Claim / requirement | Automated tests | Human review | Documentary evidence |
|---|---|---|---|
| Tenant isolation (SR-01) | Generated cross-tenant API tests; RLS coverage test; role-grant test | Migrations touching roles, grants and `SECURITY DEFINER` functions | — |
| SSRF protection (SR-02) | Unit tests on URL/IP validation; integration tests on rebinding, redirects and size limits; egress smoke test per environment | Network/egress configuration per environment | — |
| Gate binding & signatures (SR-03) | Replay, tamper, expiry, wrong-repo and unknown-key tests | Key-management configuration | Key rotation runbook |
| Audit log integrity (SR-04) | Tamper and truncation tests; daily chain verification job | DB grants, KMS key policy | — |
| Evidence immutability (SR-05) | Overwrite/delete-denied tests; re-hash job | Bucket policy per environment | Retention policy |
| GitHub App least privilege (SR-06) | Manifest permission check in CI | App permission changes | — |
| LLM safety of our own AI features (SR-07) | Eval suite: schema validity, catalog-only IDs, injection cases (rates + sample size) | Prompt changes and eval result review per PR | Prompt/version changelog |
| Quotas and budgets (SR-08) | Rate-limit and budget tests | — | — |
| Upload hardening (SR-09) | Type/size/sniffing tests | — | — |
| Supply chain (SR-10) | Dependency audit, CodeQL, gitleaks, Trivy, SBOM generation | Dependabot PR review | SBOM published per release |
| Catalog & policy integrity (SR-11) | Schema validation; policy regression scenarios | Code-owner review of every `catalog/` change; SME review of framework mappings | Source references with `verified_on` dates |
| Telemetry redaction (SR-12) | Tests asserting that no prompt bodies or secrets appear in exported spans | — | — |
| Assurance language | CI lint that blocks forbidden terms ("compliant", "certified", "secure" as a claim) in UI strings and report templates | Copy review of new report sections | [Assurance language](assurance-language.md) |
| Accessibility (WCAG 2.2 AA, core flows) | axe checks in Playwright | Manual keyboard and screen-reader pass per milestone | — |
| Threat model currency | — | Review at each milestone and on any new trust boundary | Updated threat model in repo |
| Privacy compliance of the Studio (LGPD) | — | Data-inventory review | Records of processing, privacy notice, sub-processor list |
| Framework licensing (no ISO text) | Heuristic CI check for long quoted passages in `catalog/` | Code-owner review | [Framework references](framework-references.md) |
