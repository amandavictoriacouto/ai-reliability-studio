# Contributing

Thanks for your interest. The project is in its foundation phase; the workflow below applies to all changes.

## Workflow

1. **Issue first.** Every change starts from an issue (feature, bug, ADR proposal, risk/threat or research template).
2. **Decide before building.** Product, architecture and scope changes are proposed as an issue labeled `status:needs-decision` or as an ADR (`docs/adr/`) and approved by the code owner before implementation.
3. **Branch:** `feat/<issue>-slug`, `fix/…`, `docs/…`, `adr/…`.
4. **Commits:** [Conventional Commits](https://www.conventionalcommits.org/) (`feat:`, `fix:`, `docs:`, `chore:`, `ci:`, `test:`, `refactor:`).
5. **Pull request:** small and single-purpose, using the PR template; merged using a merge commit after review and green CI; preserve commit history for stacked M0 pull requests.

## Definition of Done

- Acceptance criteria met · tests added or updated · documentation updated · security & AI impact checklist answered · CI green · change is demoable.

## Content rules

- All repository content is written in **English**.
- Never commit secrets, credentials, personal data or customer data.
- Never reproduce copyrighted standard text (e.g., ISO/IEC). See [framework references](docs/governance/framework-references.md).

## License

The project is licensed under the [Apache License 2.0](LICENSE). By submitting a contribution, you agree that it is licensed under the same terms (Apache-2.0, section 5). Do not submit code you cannot license this way, and do not copy code under incompatible licenses (for example GPL or AGPL) into the repository. New dependencies must pass the [third-party license check](docs/governance/third-party-licenses.md).
