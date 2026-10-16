# AI Reliability Studio

**Shift-left Risk Intelligence for Product Teams.** An AI-native platform that helps product teams identify, assess, mitigate and **verify** security, privacy, governance and compliance risks across the lifecycle of digital and AI-native products: from discovery and requirements to development and release decisions.

> **Status:** M0 — Product & Engineering Foundation (planning). No product features are implemented yet.
>
> This is a non-commercial, educational, open-source **AI Product Engineering portfolio project**, built in public. It shows how product, architecture, security and governance decisions are made, recorded and verified.

## Why

Risk work usually arrives late and ends in documents that *assert* rather than *demonstrate*. AI Reliability Studio connects each step of a release into one traceable chain:

```text
product context → risks → controls → automated verification → evidence → human approval → policy-based release gate
```

Every result states which requirements were verified, how, when, and by which evidence.

## What it is not

- Not a certification body, legal advisor or substitute for an auditor.
- It never claims a product is "secure" or "compliant". It reports what was **verified**, **attested**, **failed** or **not verified**, and the limits of each check. See [assurance language](docs/governance/assurance-language.md).

## Where the project stands

| Category | What it contains | Where |
|---|---|---|
| **Implemented and running** | Repository foundation: docs CI (Markdown lint, internal link check, secret scan) required on `main`; a branch ruleset; secret scanning with push protection; private vulnerability reporting; Dependabot; SHA-pinned actions | [`.github/`](.github/), [Security policy](SECURITY.md) |
| **Approved decisions** | License (Apache-2.0), English-only v0.1, authentication approach (Auth.js with BFF), free-tier hosting, phased audit checkpoints, review corrections | [Decision log](docs/product/decisions-and-open-questions.md#5-decision-log) |
| **Proposed architecture** | 16 ADRs, all *Proposed* until explicitly accepted | [ADRs](docs/adr/README.md) |
| **Tested security controls** | Only the repository controls above. Every product control in the security documents is a **design**, with its planned tests; none is implemented or tested yet | [Verification methods](docs/governance/verification-methods.md) |
| **Known limitations** | Documented per design, for example the audit log checkpoint limits and the release gate's limits | [Audit log integrity](docs/security/audit-log-integrity.md), [Release gate binding](docs/architecture/release-gate-binding.md) |
| **Future capabilities** | Milestones M1–M6 and the MVP scope | [Roadmap](docs/roadmap.md), [PRD v0.1](docs/product/prd-v0.1.md) |

## Documentation

| Area | Documents |
|---|---|
| Product | [Product brief](docs/product/product-brief.md) · [Competitive discovery](docs/product/competitive-discovery.md) · [Discovery track](docs/product/discovery-track.md) · [PRD v0.1](docs/product/prd-v0.1.md) · [Decisions & open questions](docs/product/decisions-and-open-questions.md) |
| Architecture | [MVP architecture](docs/architecture/mvp-architecture.md) · [Release gate binding](docs/architecture/release-gate-binding.md) · [Evidence model](docs/architecture/evidence-model.md) · [Job queue & Job-Lease API](docs/architecture/job-lease-api.md) · [ADRs](docs/adr/README.md) |
| Security | [Threat model](docs/security/threat-model.md) · [Endpoint verifier security](docs/security/endpoint-verifier-security.md) · [Tenant isolation & authZ](docs/security/tenant-isolation-and-authz.md) · [Audit log integrity](docs/security/audit-log-integrity.md) · [Security policy](SECURITY.md) |
| Delivery | [Roadmap](docs/roadmap.md) · [Implementation plan](docs/engineering/implementation-plan.md) · [Repository organization](docs/engineering/repository-organization.md) |
| Governance | [Assurance language](docs/governance/assurance-language.md) · [Verification methods](docs/governance/verification-methods.md) · [Framework references & licensing](docs/governance/framework-references.md) · [Third-party licenses](docs/governance/third-party-licenses.md) |

## Language

The product, code and documentation are in English (en-US). The frontend is internationalization-ready for future languages.

## Planned stack

Next.js · TypeScript · Tailwind CSS · shadcn/ui · Python · FastAPI · PostgreSQL · provider-agnostic LLM gateway · Pytest · Vitest · Playwright · OpenTelemetry · GitHub Actions. Rationale in the [ADRs](docs/adr/README.md).

## References

NIST AI RMF · OWASP GenAI Security Project · ISO/IEC 42001 · ISO/IEC 27001 · LGPD · EU AI Act. Used within their licensing and interpretation limits; see [framework references](docs/governance/framework-references.md).

## Contributing & security

See [CONTRIBUTING.md](CONTRIBUTING.md) and [SECURITY.md](SECURITY.md).

## License

[Apache License 2.0](LICENSE) (decision Q1). Copyright notice: [NOTICE](NOTICE). Catalog and third-party content licensing: [LICENSE-CONTENT.md](LICENSE-CONTENT.md) and [third-party license compatibility](docs/governance/third-party-licenses.md).

Non-commercial, educational, open-source portfolio project (decision Q6).
