# Initial Threat Model (AI Reliability Studio itself)

> Status: **Draft v0.1** · Method: STRIDE per trust boundary + OWASP Top 10 for LLM Applications 2025 for AI components · Review cadence: every milestone and on any new trust boundary.
> Dogfooding: this threat model becomes the first assessment *inside* the product once M3 ships.

## 1. Scope and assets

| ID | Asset | Why it matters | CIA priority |
|---|---|---|---|
| A1 | Product context & system profiles | Customer roadmaps, architecture, data flows (confidential) | C |
| A2 | Evidence objects | Basis of every claim; may contain secrets/PII if users upload carelessly | I, C |
| A3 | Gate decisions & policies | A forged "pass" defeats the product's purpose | **I** |
| A4 | Approvals & audit log | Non-repudiation; audit defensibility | **I** |
| A5 | Credentials: GitHub tokens/App keys, LLM API keys, CI gate tokens | Pivot into customer repos; cost abuse | C |
| A6 | AI prompts & catalog | Integrity of risk suggestions | I |
| A7 | User accounts / sessions | Account takeover | C, I |

## 2. Trust boundaries

TB1 Browser ↔ Web · TB2 Web ↔ API · TB3 CI runner ↔ Gate API · TB4 API/Worker ↔ PostgreSQL · TB5 API/Worker ↔ Object storage · TB6 AI gateway ↔ LLM provider · TB7 Worker ↔ GitHub API · TB8 Worker ↔ **customer-registered LLM endpoint** (untrusted) · TB9 Uploaded files ↔ parsers.

## 3. Threats and mitigations

Priority: **H**igh / **M**edium / **L**ow. Target: **MVP** = must ship in v0.1; **Later** = tracked.

| ID | Boundary | STRIDE / OWASP LLM | Threat | Pri | Mitigation | Target |
|---|---|---|---|---|---|---|
| T01 | TB3 | Spoofing / Tampering | Forged or replayed gate "pass" for a different commit | H | Commit and repo taken from GitHub OIDC token; decision bound to (release, SHA, policy digest, evidence snapshot digest), short TTL, Ed25519 JWS verified by the action; required Check Run posted by our GitHub App ([detail](../architecture/release-gate-binding.md)) | MVP |
| T02 | TB2/TB4 | Information disclosure | Cross-tenant data access (IDOR) | H | workspace_id on every row, composite FKs, forced RLS with `SET LOCAL`, BFF pattern, generated cross-tenant tests ([detail](tenant-isolation-and-authz.md)) | MVP |
| T03 | TB8 | SSRF / Elevation | Probe-target URL used to hit internal services or cloud metadata | H | Ownership verification + explicit authorization; deny private/link-local/metadata ranges on all resolved IPs; IP pinning against rebinding; no redirects; egress proxy; isolated runner without DB credentials; strict limits ([detail](endpoint-verifier-security.md)) | MVP |
| T04 | TB6 | LLM01 Prompt injection | Malicious text in profile/uploaded docs manipulates risk suggestions (e.g., "report no risks") | H | Delimited untrusted input; output constrained to catalog IDs; rules-based baseline risks always included for matching profile attributes; human acceptance required; injection cases in eval suite | MVP |
| T05 | TB8 | LLM01 indirect | Customer endpoint responses inject instructions into our evaluator/LLM judge | M | Treat responses as data; deterministic checks preferred over LLM-as-judge; judge prompts hardened and evaluated | MVP |
| T06 | All | LLM09 Misinformation / over-reliance | Users treat output as certification ("we're compliant") | H | Assurance language rules, "verified vs attested vs not verified" everywhere, limitations block in every report, no "compliant" badge | MVP |
| T07 | TB5 / DB | Tampering | Evidence or audit records altered after the fact | H | Server-side hashing, write-once evidence storage, tombstones; hash-chained audit log with signed external checkpoints ([evidence](../architecture/evidence-model.md), [audit limits](audit-log-integrity.md)) | MVP |
| T08 | TB1 | Repudiation | Approver denies having approved | M | Approvals require re-auth (recent login), recorded with identity, IP, user agent, justification; included in hash chain | MVP |
| T09 | TB7 | Elevation / Info disclosure | Over-privileged GitHub token leaks or is abused | H | GitHub App with read-only minimal permissions; installation tokens short-lived; secrets encrypted (KMS/libsodium) and never logged | MVP |
| T10 | TB6 | LLM02 Sensitive info disclosure / LGPD | Customer confidential/personal data sent to LLM providers and retained | H | Data minimisation, PII redaction pass, per-workspace "no LLM" mode, provider zero-retention options where available, documented sub-processors and international transfer basis | MVP (doc + toggle), Later (DPA) |
| T11 | TB9 | Tampering / DoS | Malicious uploads (zip bombs, polyglots, macro docs, XSS via SVG/HTML) | M | Allow-list types, size limits, no server-side rendering of HTML/SVG, content-type sniffing, serve from separate domain with `Content-Disposition: attachment`, malware scan Later | MVP (limits), Later (AV) |
| T12 | TB6 | LLM10 Unbounded consumption | Cost exhaustion via repeated discovery or huge inputs | M | Per-workspace quotas, token budgets, rate limits, input size caps, cost metrics + alerts | MVP |
| T13 | TB6 | LLM06 Excessive agency | LLM triggers actions (create issues, change status) | M | LLM has **no tools** in MVP; all writes are explicit user actions | MVP |
| T14 | Build | LLM03 Supply chain / Tampering | Compromised dependency, action or container | M | Lockfiles, Dependabot, pinned action SHAs, pip-audit/npm audit, Trivy, SBOM (Syft) per release, signed releases (Sigstore) Later | MVP (scan/pin), Later (signing) |
| T15 | TB1 | Spoofing | Account takeover | M | OIDC providers only (no passwords), secure cookies, CSRF protection, session rotation | MVP |
| T16 | Catalog | LLM04 Data/model poisoning (catalog) | Malicious PR alters catalog or default policy to weaken checks | M | CODEOWNERS on `catalog/`, required review, schema + regression tests ("policy X must still block scenario Y") | MVP |
| T17 | TB2 | LLM05 Improper output handling | LLM output rendered as HTML → XSS | M | Render as text/sanitised Markdown only; strict CSP | MVP |
| T18 | TB2 | LLM07 System prompt leakage | Users extract system prompts | L | No secrets in prompts (prompts are public in repo anyway) | MVP |
| T19 | Ops | Info disclosure | Logs/traces contain secrets or customer prompts | M | Redaction processors in OTel pipeline, no prompt bodies by default | MVP |
| T20 | TB3 | DoS | Gate API outage blocks customer releases | M | Configurable fail-open/fail-closed per policy, audited "break-glass" override | MVP |

## 4. Residual risks and assumptions

- Hosting provider and LLM providers are trusted sub-processors (assumption; documented).
- Verifiers prove **specific, scoped facts** at a point in time; they don't prove absence of vulnerabilities (stated in every report).
- Legal interpretation (LGPD, EU AI Act) is out of scope; framework mappings are informational.

## 5. Security requirements derived for the backlog

SR-01 tenant isolation tests · SR-02 SSRF guard library with tests for IPv4/IPv6/DNS rebinding · SR-03 gate decision binding + signature · SR-04 hash-chained audit + verify CLI · SR-05 evidence write-once · SR-06 GitHub App least privilege · SR-07 LLM input/output guards + injection eval cases · SR-08 quotas & budgets · SR-09 upload hardening · SR-10 CI supply-chain scanning · SR-11 CODEOWNERS for catalog/policies · SR-12 OTel redaction.
