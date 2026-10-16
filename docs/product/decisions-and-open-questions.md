# Decisions, Risks and Open Questions (for approval)

> Status of each item is tracked here and in GitHub Issues labeled `status:needs-decision`. Approved items move to the decision log below and, where architectural, to an ADR with status *Accepted*.

## 1. Decisions requiring approval

| ID | Area | Recommendation | Main alternative | Why |
|---|---|---|---|---|
| **D1** | Scope | MVP = one vertical slice "Release Readiness for AI-native features" (context → risks → controls → verification → evidence → gate). All 11 capabilities exist in thin form; Continuous Monitoring deferred to v0.2. | Broader, shallower coverage of all modules | Demonstrates the full value chain; credible for portfolio and users. |
| **D2** | Positioning | Keep "Shift-left Risk Intelligence for Product Teams"; unit of analysis = **feature/release**; wedge = **evidence-backed release gate**. | Org-level AI governance registry | Avoids head-on competition with Credo/OneTrust/Vanta (see discovery). |
| **D3** | AI design | AI suggests only from a versioned catalog; humans accept; deterministic rules baseline; no LLM tools/agency in MVP. | Free-form generative risk analysis; agentic discovery | Auditability, lower hallucination and injection risk. |
| **D4** | Assurance language | Use "verified / attested / failed / not verified" + mandatory limitations; never "compliant/secure/certified". | Compliance score % | Matches your "no absolute promises" principle. |
| **D5** | Architecture | Modular monolith + worker, Postgres (RLS) + Postgres queue, S3-compatible evidence store, hash-chained audit log (ADR-0001…008). | Microservices; Redis/Celery | Lowest ops for solo builder; strong integrity story. |
| **D6** | Stack refinements | Keep your stack; add `uv`, `ruff`, `mypy`, SQLAlchemy 2/Alembic, Pydantic v2, pnpm, openapi-typescript, Testcontainers, Schemathesis, import-linter. | — | Modern defaults, contract-first API. |
| **D7** | LLM layer | Own thin `LLMProvider` interface; adapters for Anthropic + OpenAI first. | LiteLLM / LangChain | Control of provenance and evals; small surface. |
| **D8** | Policy engine | Declarative YAML evaluated in Python, behind an interface; OPA as future option. | OPA/Rego from day 1 | PM-readable policies. |
| **D9** | Verifiers in MVP | (a) GitHub repo posture, (b) LLM endpoint probe suite, plus (c) a minimal GitHub Actions CI-results import with verified provenance (F5). | Cloud config (AWS), broader SAST/DAST imports | Directly tied to AI-feature risks; demoable without cloud accounts. |
| **D10** | Frameworks in v0.1 | Depth: OWASP LLM Top 10 2025 + NIST AI RMF. Reference-only mappings: ISO/IEC 42001 & 27001 (identifiers), LGPD articles. EU AI Act: informational only (no classification) until v0.2+. | Full EU AI Act module in MVP | EU AI Act dates are shifting (Omnibus); ISO texts are licensed. |
| **D11** | Dogfooding | The Studio assesses itself and gates its own releases from M5. | — | Strongest portfolio proof. |

### Decisions proposed by the M0 architecture review

| ID | Area | Recommendation | Detail |
|---|---|---|---|
| **D12** | Gate authentication | GitHub Actions OIDC as the primary CI identity; commit SHA and repository taken from the signed token, not the request body; scoped API tokens only as a labelled fallback | [Release gate binding](../architecture/release-gate-binding.md) |
| **D13** | Gate decisions | Decisions signed as JWS (Ed25519) over commit, policy digest and evidence snapshot digest; public JWKS for independent verification | [Release gate binding](../architecture/release-gate-binding.md) |
| **D14** | Evidence | Per-workspace envelope encryption enabling crypto-shredding; governance-mode object lock; tombstones on deletion; default retention 3 years for decision-referenced evidence | [Evidence model](../architecture/evidence-model.md) |
| **D15** | Audit log | **Approved 2026-10-16, phased:** per-workspace chains; signed checkpoints in the same database in M1 (with stated limits); Merkle tree and independent write-once storage in M3; external anchoring later | [Audit log integrity](../security/audit-log-integrity.md) |
| **D16** | Frontend ↔ backend | **Approved 2026-10-16.** Backend-for-Frontend: browser holds only an httpOnly session; Next.js server calls the API with 60 s identity-only JWTs; roles loaded from DB per request | [Tenant isolation & authZ](../security/tenant-isolation-and-authz.md) |
| **D17** | Verifier isolation | Separate verifier-runner pool with no database/storage credentials, egress proxy, fixed probe suites, ownership verification via DNS TXT or well-known file | [Endpoint verifier security](../security/endpoint-verifier-security.md) |

## 2. Framework licensing & interpretation boundaries (proposed rules)

| Reference | Proposed handling | Note |
|---|---|---|
| NIST AI RMF 1.0 + Playbook | Reference IDs and paraphrase; quote with attribution | US Government work, generally public domain — *verify on NIST site before quoting at length*. |
| OWASP GenAI (LLM Top 10 2025) | Reference IDs, short attribution, link | OWASP content is typically CC BY-SA 4.0 — *verify*; any derived text in `catalog/` must carry attribution and share-alike → separate `LICENSE-CONTENT.md`. |
| ISO/IEC 42001, 27001 | **Only clause/control identifiers and our own one-line summaries**; no reproduced text | Copyrighted, paid standards. Users with a licence can attach their own notes. |
| LGPD (Lei 13.709/2018) | Article references and quotes allowed (statute) | Mappings are informational, not legal advice. |
| EU AI Act (Reg. 2024/1689) | Article references; dates flagged as "subject to Omnibus" | Re-verify Omnibus publication status before shipping dates. |

## 3. Key risks

| ID | Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|---|
| R1 | Scope creep across 11 capabilities stalls delivery | High | High | D1 vertical slice; strict milestone exit criteria. |
| R2 | Users read outputs as certification → legal/reputational exposure | Med | High | D4 language rules; disclaimers; ToS. |
| R3 | LLM suggestions are wrong or manipulated | Med | High | D3; evals; injection tests; human acceptance. |
| R4 | Probe verifier becomes an SSRF/abuse vector | Med | High | Ownership check, egress controls (threat model T03). |
| R5 | Licensing violation (ISO text in public repo) | Low | High | Section 2 rules; CODEOWNERS on `catalog/`. |
| R6 | Regulatory dates/obligations change mid-build | High | Med | Dates as data in catalog, sourced and versioned. |
| R7 | Solo capacity / burnout | High | High | Small increments; content reuse from build; cut M6 if needed. |
| R8 | Incumbents ship similar PM-facing features | Med | Med | Differentiate on workflow + traceability; move fast to public demo. |
| R9 | No validated demand (it remains a portfolio piece only) | Med | Med | Parallel interview track with kill criteria. |
| R10 | LLM/hosting costs for a public demo | Med | Low | Free tiers only (Q6); no paid service without explicit authorization; quotas, cached demo data, "no LLM" mode; local-only fallback for capabilities that cannot run safely for free. |

## 4. Open questions for the product owner

| ID | Question | Options (recommended marked ★) |
|---|---|---|
| ~~Q1~~ | ~~Repository license?~~ | **Approved 2026-10-16:** Apache-2.0. See decision log. |
| **Q2** | Primary demo scenario? | ★ A) "AI customer-support reply assistant" (RAG + PII) · B) AI hiring screener (EU AI Act high-risk flavour) · C) Internal coding agent |
| ~~Q3~~ | ~~When to make the repo public?~~ | **Resolved:** the repository is already public (created 2026-10-15). |
| ~~Q4~~ | ~~UI language for v0.1?~~ | **Approved 2026-10-15:** English (en-US) only; i18n-ready frontend. See decision log. |
| ~~Q5~~ | ~~Auth approach (ADR-0009)?~~ | **Approved 2026-10-16:** Auth.js with BFF. See decision log. |
| ~~Q6~~ | ~~Hosting (ADR-0016)?~~ | **Approved 2026-10-16:** free tiers for a non-commercial project; Vercel Hobby preferred for the web app. See decision log. |
| **Q7** | LLM providers to support first? | ★ A) Anthropic + OpenAI · B) One provider + local (Ollama) · C) Other |
| **Q8** | Target market emphasis for discovery interviews? | ★ A) Brazil first (LGPD), EU-exposed second · B) Global from start |
| **Q9** | Weekly capacity (hours) and any target date (e.g., an event or job-search window) for v0.1? | Needed to turn milestones into dates. |

## 5. Decision log

| Date | ID | Decision | Approved by | Reference |
|---|---|---|---|---|
| 2026-10-15 | Q3 | Repository is public from M0 (build in public). | Repository owner (repo created public) | — |
| 2026-10-15 | Q4 | **Language and internationalization.** (1) English (en-US) is the default and only supported language in MVP v0.1. (2) All UI components, navigation, forms, notifications, validation messages and reports use English. (3) Source code, comments, tests, documentation, Issues, PRs, commit messages and ADRs are written in English. (4) Product documentation (brief, discovery, PRD, roadmap) is maintained in English. (5) The frontend is internationalization-ready so pt-BR and other languages can be added without major refactoring. (6) No multilingual UI or translation services in the MVP. (7) Regulatory frameworks keep their official terminology, identifiers and references. | Product owner | [MVP architecture §11](../architecture/mvp-architecture.md#11-internationalization-readiness) |
| 2026-10-16 | Q1 | **License.** The repository's code is licensed under the **Apache License 2.0**, with the official license text in `LICENSE` and the copyright notice in `NOTICE`. Third-party license compatibility of the planned stack is checked and tracked. Framework material keeps its own license. | Product owner | [`LICENSE`](../../LICENSE), [third-party licenses](../governance/third-party-licenses.md) |
| 2026-10-16 | Q5 | **Authentication.** Auth.js with a Backend-for-Frontend (BFF) pattern. Secure server-side sessions, token protection (no API tokens in the browser), RBAC and multi-tenant isolation are maintained. ADR-0009 is updated and stays *Proposed* until its final text is accepted (accepted 2026-10-16). | Product owner | [ADR-0009](../adr/0009-authentication.md), [tenant isolation](../security/tenant-isolation-and-authz.md) |
| 2026-10-16 | Q6 | **Hosting and cost.** AI Reliability Studio is a non-commercial, educational, open-source portfolio project. Vercel Hobby is preferred for the frontend, subject to its terms and limits. Free tiers are prioritized for FastAPI, PostgreSQL, object storage and isolated runners. The security architecture is preserved even if some capabilities remain local-only. **No paid infrastructure or paid service is provisioned or enabled without explicit authorization.** ADR-0016 is updated and stays *Proposed*. | Product owner | [ADR-0016](../adr/0016-hosting.md) |
| 2026-10-16 | F1–F7 | Corrections from the M0 architecture and product review approved: head-SHA gate binding (F1), ADR-0009 text (F2), PRD ownership and gate stories (F3), Job-Lease API for the isolated runner (F4), CI-results import with verified provenance (F5), precedence of legal holds, retention and erasure (F6), queue-table roles and tenant-scoped workers (F7). The documents are updated; implementation is planned for M1 and later. | Product owner | [Job-Lease API](../architecture/job-lease-api.md), [release gate binding](../architecture/release-gate-binding.md), [evidence model §5.4](../architecture/evidence-model.md#54-precedence-legal-holds-retention-obligations-and-erasure-requests) |
| 2026-10-16 | D15 | **Audit checkpoints, phased.** Phase 1 (M1): hourly Ed25519-signed checkpoints, key in the platform secret store, stored in the same database and in audit exports. Phase 1 is **not** independent or tamper-proof storage and does **not** guarantee detection of complete history deletion; trust assumptions and residual risks are documented. Phase 2 (M3): Merkle proofs and independent write-once storage. Phase 3 (later): external anchoring. | Product owner | [Audit log integrity §4](../security/audit-log-integrity.md#4-checkpoints-anchoring-phased-d15) |
| 2026-10-16 | D16 | **BFF authentication** as described in ADR-0009 (Auth.js server-side sessions, identity-only service JWTs, no API tokens in the browser, roles loaded per request). | Product owner | [ADR-0009](../adr/0009-authentication.md) |
| 2026-10-16 | M1 scope | **M1 simplifications:** GitHub-only login initially (Google later); PostgreSQL-only local infrastructure (other services added when needed); remaining tooling adopted incrementally. Essential security controls are not removed. Also approved: ADR-0003, ADR-0008 and ADR-0013 text revisions. | Product owner | [ADR-0003](../adr/0003-postgres-tenant-isolation.md), [ADR-0008](../adr/0008-hash-chained-audit-log.md), [ADR-0013](../adr/0013-contract-first-api.md), [ADR-0015](../adr/0015-local-development.md) |
| 2026-10-16 | ADRs | **M1 architecture accepted.** ADR-0001 (modular monolith), ADR-0002 (monorepo), ADR-0003 (PostgreSQL and RLS), ADR-0008 (hash-chained audit log), ADR-0009 (authentication with BFF), ADR-0010 (RBAC), ADR-0013 (code-first FastAPI with generated OpenAPI contract), ADR-0014 (tooling) and ADR-0015 (local development) are *Accepted* in their reviewed text. D15 (phased, with its M1 limits and trust assumptions), D16 and the M1 simplifications are confirmed. All other ADRs stay *Proposed* until separately approved. | Product owner | [ADR index](../adr/README.md) |
| 2026-10-16 | Deferrals | D7/Q7 (paid LLM providers) deferred to M2; D14 (evidence encryption and key management) deferred to the M3 design review. No paid services or API usage without explicit approval. | Product owner | — |
| *pending* | D1–D6, D8–D13, D17, Q2, Q8, Q9; D7/Q7 (M2); D14 (M3); acceptance of ADR-0004, 0005, 0006, 0007, 0011, 0012, 0016 (still *Proposed*) | Awaiting review | — | — |
