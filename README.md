# AI Reliability Studio

**Shift-left Risk Intelligence for Product Teams.** An AI-native platform that helps product teams identify, assess, mitigate and **verify** security, privacy, governance and compliance risks across the lifecycle of digital and AI-native products: from discovery and requirements to development and release decisions.

> **Status:** M0 — Product & Engineering Foundation (planning). No features are implemented yet.

## Why

Risk work usually arrives late and ends in documents that *assert* rather than *demonstrate*. AI Reliability Studio connects each step of a release into one traceable chain:

```text
product context → risks → controls → automated verification → evidence → human approval → policy-based release gate
```

Every result states which requirements were verified, how, when, and by which evidence.

## What it is not

- Not a certification body, legal advisor or substitute for an auditor.
- It never claims a product is "secure" or "compliant". It reports what was **verified**, **attested**, **failed** or **not verified**, and the limits of each check. See [assurance language](docs/governance/assurance-language.md).

## Documentation

| Area | Documents |
|---|---|
| Product | [Product brief](docs/product/product-brief.md) · [Competitive discovery](docs/product/competitive-discovery.md) · [Discovery track](docs/product/discovery-track.md) · [PRD v0.1](docs/product/prd-v0.1.md) · [Decisions & open questions](docs/product/decisions-and-open-questions.md) |
| Architecture | [MVP architecture](docs/architecture/mvp-architecture.md) · [Release gate binding](docs/architecture/release-gate-binding.md) · [Evidence model](docs/architecture/evidence-model.md) · [ADRs](docs/adr/README.md) |
| Security | [Threat model](docs/security/threat-model.md) · [Endpoint verifier security](docs/security/endpoint-verifier-security.md) · [Tenant isolation & authZ](docs/security/tenant-isolation-and-authz.md) · [Audit log integrity](docs/security/audit-log-integrity.md) · [Security policy](SECURITY.md) |
| Delivery | [Roadmap](docs/roadmap.md) · [Implementation plan](docs/engineering/implementation-plan.md) · [Repository organization](docs/engineering/repository-organization.md) |
| Governance | [Assurance language](docs/governance/assurance-language.md) · [Verification methods](docs/governance/verification-methods.md) · [Framework references & licensing](docs/governance/framework-references.md) |

## Language

The product, code and documentation are in English (en-US). The frontend is internationalization-ready for future languages.

## Planned stack

Next.js · TypeScript · Tailwind CSS · shadcn/ui · Python · FastAPI · PostgreSQL · provider-agnostic LLM gateway · Pytest · Vitest · Playwright · OpenTelemetry · GitHub Actions. Rationale in the [ADRs](docs/adr/README.md).

## References

NIST AI RMF · OWASP GenAI Security Project · ISO/IEC 42001 · ISO/IEC 27001 · LGPD · EU AI Act. Used within their licensing and interpretation limits; see [framework references](docs/governance/framework-references.md).

## Contributing & security

See [CONTRIBUTING.md](CONTRIBUTING.md) and [SECURITY.md](SECURITY.md).

## License

To be defined (decision Q1 in [decisions & open questions](docs/product/decisions-and-open-questions.md)). Catalog content licensing: [LICENSE-CONTENT.md](LICENSE-CONTENT.md).
