# ADR-0016: Hosting target: free tiers for a non-commercial project

- Status: Proposed
- Date: 2026-10-15
- Updated: 2026-10-16 (Q6 approved; free-tier direction)
- Deciders: product owner (@amandavictoriacouto)
- Related: [MVP architecture](../architecture/mvp-architecture.md), [threat model](../security/threat-model.md)

## Context

The MVP needs a public demo environment. Product owner decision **Q6 was approved on 2026-10-16**: AI Reliability Studio is a **non-commercial, educational, open-source portfolio project**. Hosting must prioritize free tiers, and no paid infrastructure or paid service may be provisioned or enabled without explicit authorization. The security architecture must be preserved even if some capabilities only run locally. This ADR stays *Proposed* until its final text is accepted; provider choices are confirmed in M1.

## Decision

1. **Web (Next.js):** Vercel **Hobby** plan, preferred, subject to its terms of use and limits. Hobby is intended for personal, non-commercial use; the project must stay within that scope, and the choice is re-checked before any commercial use.
2. **API, worker, PostgreSQL, object storage, verifier runner:** free tiers only, chosen in M1 against a written checklist: no payment method required or no automatic paid upgrade; support for forced RLS and separate database roles; S3-compatible storage with object lock or an equivalent write-once control; network isolation and egress control for the runner; data region documented.
3. **Local-first fallback:** any capability that cannot be hosted for free **without weakening a security control** runs only in the local Docker Compose environment ([ADR-0015](0015-local-development.md)) and is shown in the public demo with recorded evidence instead. The most likely case is the isolated verifier runner with its egress proxy. A control is never removed to fit a free tier.
4. **Cost guardrails:** no paid plan, add-on or usage-based billing is enabled. Where a provider requires a payment method, spending limits are set to zero and the case goes to the product owner first. Demo data is synthetic.
5. **Secrets** live in each provider's secret store; nothing is committed to the repository.

## Alternatives considered

- Paid PaaS (e.g., Fly.io or Render paid plans, managed Postgres paid tiers): simpler, but conflicts with Q6.
- AWS (ECS, RDS, S3): enterprise realism, higher cost and operations.
- Local-only, no public demo: zero cost, weaker portfolio value.

## Consequences

- **Positive:** Zero cost; the security design is unchanged; free-tier limits are documented rather than hidden.
- **Negative:** Free tiers have sleep, quota and retention limits, so the public demo may be slow or partial. Several vendors to manage. Some controls (runner network isolation, object lock) may only be demonstrable locally. Provider terms can change and must be re-checked at each milestone.
