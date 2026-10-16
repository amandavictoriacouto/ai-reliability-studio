# Commit and Pull Request Sequence (M0)

Changes reach `main` only through pull requests from feature branches, using Conventional Commits. The four M0 pull requests are **stacked**: each one is based on the previous branch, so CI on every PR checks a link-complete subset of the documentation. They are merged in order, and only with the product owner's explicit approval.

## PR 1 — `chore/m0-repo-foundation` → `main`: "chore: repository foundation"

1. `chore: add .gitignore, .editorconfig and markdownlint config`
2. `docs: add code of conduct`
3. `ci: add issue templates, PR template, CODEOWNERS and labels`
4. `ci: add dependabot for GitHub Actions`
5. `ci: add docs workflow (markdown lint, internal links, secret scan)`
6. `docs: format existing README to pass markdown lint`
7. `chore: add Apache-2.0 license and NOTICE`

## PR 2 — `docs/m0-architecture` → PR 1: "docs(architecture): MVP architecture, ADRs, security and governance"

1. `docs(architecture): add MVP architecture`
2. `docs(adr): add ADR template, index and ADR-0001 to ADR-0016 as proposed`
3. `docs(security): add initial threat model and security policy`
4. `docs(architecture): add release gate binding and evidence model`
5. `docs(security): add endpoint verifier, tenant isolation and audit log integrity reviews`
6. `docs(governance): add assurance language, verification methods and framework references`
7. `docs(architecture): bind gate decisions to the pull request head commit`
8. `docs(architecture): add job queue roles and runner Job-Lease API`
9. `docs(architecture): define precedence of legal holds, retention and erasure`
10. `docs(governance): require verified provenance for imported CI results`
11. `docs(adr): record Q5 and Q6 in ADR-0009 and ADR-0016, still proposed`
12. `docs(governance): add third-party license compatibility check`
13. `docs(governance): reference the CI-results import story by its PRD ID`

## PR 3 — `docs/m0-product` → PR 2: "docs(product): brief, discovery, PRD and roadmap"

1. `docs(product): add product brief`
2. `docs(product): add competitive discovery and parallel discovery track`
3. `docs(product): add PRD v0.1`
4. `docs(product): add decisions log with Q4 approved`
5. `docs: add roadmap`
6. `docs: add contributing guide and content licensing notes`
7. `docs(product): align PRD with gate head-SHA binding, ownership proof and CI import`
8. `docs(product): record Q1, Q5, Q6 and the F1-F7 corrections as approved`
9. `docs: point licensing notes and contributing guide to Apache-2.0`

## PR 4 — `docs/m0-delivery` → PR 3: "docs: delivery plan, GitHub structure and README"

1. `docs(engineering): add implementation plan and repository organization`
2. `docs(pm): add GitHub structure, initial backlog and commit sequence`
3. `docs: add documentation index and changelog`
4. `docs: rewrite README with positioning and documentation index`
5. `docs(pm): list README lint fix in commit sequence`
6. `docs: update README, plans and backlog for Q1, Q5, Q6 and F1–F7`

## After the documentation PRs (each after its decision)

| Step | Content | Trigger |
|---|---|---|
| PR 5 | `docs(adr): accept ADR-…` (status updates + decision log) | D-decisions approved |
| GitHub API setup | Labels, milestones M0–M6, backlog issues, Project board, branch protection | PRs 1–4 merged, explicit approval |
| PR 6 | `feat: scaffold monorepo` (M0-11) | ADRs accepted |
| PR 8 | `ci: app CI and security workflows` (M0-12) | PR 6 |
| PR 8 | `feat(catalog): JSON Schemas v0` (M0-13) | ADRs accepted |
| PR 9 | `docs(research): interview guide v1` (M0-14) | — |
| Tag | `v0.0.1` release | M0 exit criteria met, explicit approval |
