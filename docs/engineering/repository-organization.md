# GitHub Repository Organization

> Status: **Draft for approval** · Repo: `amandavictoriacouto/ai-reliability-studio` (public)

## 1. Directory layout

```text
ai-reliability-studio/
├─ README.md                  # pitch, demo GIF, architecture diagram, quickstart, limitations
├─ LICENSE                    # Apache-2.0 (decision Q1)
├─ NOTICE                     # copyright notice
├─ LICENSE-CONTENT.md         # licensing of catalog content & third-party references
├─ SECURITY.md                # vulnerability disclosure policy
├─ CONTRIBUTING.md  CODE_OF_CONDUCT.md  CHANGELOG.md
├─ .github/
│  ├─ workflows/              # ci-api.yml, ci-web.yml, catalog.yml, evals.yml, security.yml, release.yml
│  ├─ ISSUE_TEMPLATE/         # feature.yml, bug.yml, adr-proposal.yml, risk.yml, research.yml
│  ├─ pull_request_template.md
│  ├─ CODEOWNERS
│  └─ dependabot.yml
├─ apps/
│  ├─ web/                    # Next.js
│  └─ api/                    # FastAPI + worker
├─ packages/
│  └─ api-client/             # generated TypeScript client
├─ integrations/
│  └─ github-action/          # ars-gate action
├─ catalog/
│  ├─ schemas/                # JSON Schemas for all catalog files
│  ├─ risks/                  # risk catalog (YAML)
│  ├─ controls/               # control catalog (YAML)
│  ├─ mappings/               # control ↔ framework references
│  ├─ probes/                 # LLM probe suites
│  └─ policies/               # default release policies
├─ evals/                     # golden profiles, expected risks, eval reports
├─ docs/
│  ├─ product/                # brief, discovery, PRD, personas, interviews (anonymised)
│  ├─ architecture/           # overview, C4 diagrams, data model
│  ├─ adr/                    # 0001-modular-monolith.md …
│  ├─ security/               # threat model, security requirements
│  ├─ governance/             # assurance language rules, framework usage & licensing notes, AI use policy
│  └─ roadmap.md
├─ infra/
│  ├─ docker-compose.yml
│  └─ otel/
└─ scripts/
```

## 2. GitHub features

| Feature | Usage |
|---|---|
| **Milestones** | M0…M6 exactly as the roadmap; each with description = outcome + exit criteria. |
| **Issues** | Templates: Feature (user story + AC), Bug, ADR proposal, Risk/Threat, Research. Every PR closes ≥ 1 issue. |
| **Labels** | `type:` feature/bug/docs/adr/research/security/chore · `area:` web/api/ai/catalog/verifier/gate/infra · `priority:` p0–p3 · `status:` needs-decision/blocked · `good first issue` |
| **Projects (v2)** | Board "AI Reliability Studio": columns Backlog → Ready → In progress → In review → Done; fields: Milestone, Area, Size (XS–L), Decision needed (Y/N). Views: Board, Roadmap (by milestone), Decisions. |
| **Pull Requests** | Conventional Commits titles; template with Before/After, How, Tests, Security & AI impact checklist, Docs/ADR updated. Merge commits (or rebase); no squash merges, which the `main` ruleset enforces. |
| **Branch protection** | `main`: PR required, 1 review (code owner), required checks, linear history, signed commits recommended, no force-push. |
| **Actions** | CI per app (lint, type, test), catalog schema validation, AI evals (recorded fixtures on PR, live nightly), security (CodeQL, gitleaks, pip-audit, npm audit, Trivy), release (changelog + SBOM). |
| **Releases** | SemVer; release-please or changesets; notes link to milestone, demo, ADRs. |
| **ADRs** | MADR format in `docs/adr/`; status Proposed → Accepted/Rejected/Superseded; ADR PRs labeled `type:adr` and require code-owner approval. |
| **Security** | Private vulnerability reporting enabled, Dependabot alerts/updates, secret scanning + push protection, CodeQL. |
| **Discussions** (optional) | Public roadmap feedback, interview recruiting. |

## 3. Conventions

- Branches: `feat/<issue>-slug`, `fix/…`, `docs/…`, `adr/…`.
- Commits: Conventional Commits.
- Definition of Done: AC met · tests added · docs updated · security/AI checklist answered · CI green · demoable.
- Language: all repo content in English.
