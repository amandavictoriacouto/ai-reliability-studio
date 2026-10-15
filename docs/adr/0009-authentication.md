# ADR-0009: Authentication via OIDC (pending approval of Q5)

- Status: Proposed
- Date: 2026-10-15
- Deciders: product owner (@amandavictoriacouto)
- Related: [MVP architecture](../architecture/mvp-architecture.md), [threat model](../security/threat-model.md), [detailed design](../security/tenant-isolation-and-authz.md)

## Context

Users are product and engineering professionals; passwords add risk (T15). CI needs non-interactive access to the Gate API.

## Decision

Proposed: Auth.js in the web app with GitHub and Google OIDC; the API validates short-lived JWTs; CI uses scoped, revocable API tokens.

## Alternatives considered

- Clerk or Auth0: faster setup, paid tiers, vendor dependency.
- Keycloak: full control, heavy to operate.

## Consequences

- **Positive:** No password storage; standard protocols.
- **Negative:** Session/JWT bridging between Next.js and FastAPI must be designed carefully.
