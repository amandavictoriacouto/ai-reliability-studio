# ADR-0009: Authentication with Auth.js, BFF and GitHub OIDC for CI

- Status: Proposed
- Date: 2026-10-15
- Updated: 2026-10-16 (Q5 approved; decision text rewritten, review finding F2; D16 approved; GitHub-only login in M1)
- Deciders: product owner (@amandavictoriacouto)
- Related: [MVP architecture](../architecture/mvp-architecture.md), [threat model](../security/threat-model.md), [detailed design](../security/tenant-isolation-and-authz.md)

## Context

Users are product and engineering professionals; passwords add risk (T15). CI needs non-interactive access to the Gate API. Product owner decision **Q5 was approved on 2026-10-16**: Auth.js with a Backend-for-Frontend (BFF) pattern, keeping secure sessions, token protection, RBAC and multi-tenant isolation. This ADR records the resulting design and stays *Proposed* until its final text is accepted.

## Decision

1. **Login:** Auth.js in the Next.js app. **M1 ships GitHub sign-in only** (approved simplification, 2026-10-16); Google OIDC is added later without design changes. No passwords are stored.
2. **Sessions:** server-side database sessions referenced by an `httpOnly`, `Secure`, `SameSite=Lax` cookie. Sessions can be revoked immediately. Session rotation on login and privilege change; idle and absolute timeouts.
3. **BFF:** the browser never holds API tokens. Only the Next.js server calls the API, minting a per-request **identity-only** JWT (ES256, `aud=ars-api`, `sub`, `sid`, 60 s lifetime) with a key held only by the web server. The API is not reachable from browsers (no CORS).
4. **Authorization:** the API loads membership and role from the database on every request (RBAC, [ADR-0010](0010-rbac.md)) and enforces tenant isolation with `SET LOCAL` and forced RLS ([ADR-0003](0003-postgres-tenant-isolation.md)). Frontend checks are UX only.
5. **Token protection:** CSRF tokens and `Origin` checks on state-changing routes; no tokens in browser storage; signing keys in a secret manager with rotation; recent-authentication step-up (≤ 10 min) for approvals, overrides, endpoint authorization and deletions.
6. **CI:** the Gate API accepts **GitHub Actions OIDC** tokens bound to a repository by a workspace admin ([release gate binding](../architecture/release-gate-binding.md)). Scoped, hashed, revocable API tokens (≤ 90 days) are a labelled fallback for other CI.
7. **Runners:** the verifier runner uses its own pool credential and job-scoped tokens ([Job-Lease API](../architecture/job-lease-api.md)), never user tokens.

## Alternatives considered

- Clerk or Auth0: faster setup, paid tiers, vendor dependency.
- Keycloak: full control, heavy to operate.
- Browser-held API access tokens (SPA pattern): simpler, but exposes tokens to XSS.

## Consequences

- **Positive:** No password storage; standard protocols; no API tokens in the browser; role changes take effect on the next request; no paid identity service (fits Q6).
- **Negative:** The web server's signing key becomes a high-value secret. GitHub OAuth has no standard `max_age`/`prompt=login`, so step-up may require a full re-login. Every API call passes through the Next.js server, adding a hop.
