# Third-Party License Compatibility

> Status: **Initial check** (M0, 2026-10-16) · Related: [`LICENSE`](../../LICENSE) (Apache-2.0, decision Q1), [`NOTICE`](../../NOTICE), [framework references](framework-references.md), `LICENSE-CONTENT.md` (content licensing notes)
> Scope: the **planned** stack in the [MVP architecture](../architecture/mvp-architecture.md). No application dependency is installed yet. Licenses are re-checked automatically once dependencies are pinned (§4).

## 1. Project license

AI Reliability Studio's code is licensed under the **Apache License 2.0** (decision Q1, approved 2026-10-16). Apache-2.0 is a permissive license with an explicit patent grant. It is compatible with permissive dependencies (MIT, BSD, ISC, Apache-2.0) and with LGPL libraries used as unmodified, separately installed dependencies. It is **not** compatible with including GPL-2.0-only code, and copyleft (GPL/AGPL) code must not be copied into or linked into this repository's distributed code.

## 2. Planned dependencies

Licenses for packages were read from the npm and PyPI registry metadata on 2026-10-16. Rows marked † were taken from the projects' own documentation and were not re-checked in this session.

| Component | Use | License | Compatible | Notes |
|---|---|---|---|---|
| Next.js, React | Web app | MIT | Yes | |
| TypeScript | Web app, GitHub Action | Apache-2.0 | Yes | |
| Tailwind CSS, shadcn/ui | UI | MIT | Yes | shadcn/ui components are copied into the repo; keep their MIT notice in the copied files |
| TanStack Query, openapi-typescript, next-intl | Web app | MIT | Yes | |
| Auth.js (`next-auth`, `@auth/core`) | Authentication (Q5) | ISC | Yes | Re-check the project's maintenance status at scaffold time |
| FastAPI, Pydantic, SQLAlchemy, Alembic | API | MIT | Yes | |
| Procrastinate | Job queue (ADR-0004) | MIT | Yes | Depends on psycopg |
| psycopg 3 | PostgreSQL driver | LGPL-3.0-only | Yes, as a dependency | Used unmodified and installed from PyPI, never vendored or modified. If it is ever modified, the LGPL obligations apply to that copy |
| cryptography | Signing, encryption | Apache-2.0 OR BSD-3-Clause | Yes | |
| joserfc (candidate) | JWS / JWT | BSD-3-Clause | Yes | Library choice is confirmed at scaffold time |
| rfc8785 (candidate) | JSON canonicalization | Apache-2.0 | Yes | |
| Anthropic SDK | LLM adapter | MIT | Yes | Provider terms of service apply separately to API usage |
| OpenAI SDK | LLM adapter | Apache-2.0 | Yes | Provider terms of service apply separately to API usage |
| OpenTelemetry SDKs | Observability | Apache-2.0 | Yes | |
| pytest, mypy, ruff, Schemathesis | Dev and test tools | MIT | Yes | Not distributed |
| uv | Python tooling | MIT OR Apache-2.0 | Yes | Not distributed |
| Bandit, pip-audit, testcontainers | Dev and security tools | Apache-2.0 | Yes | Not distributed |
| Playwright | E2E tests | Apache-2.0 | Yes | Not distributed |
| Vitest, ESLint, Prettier, pnpm | Dev tools | MIT | Yes | Not distributed |
| markdownlint-cli2 | Docs CI | MIT | Yes | Not distributed |
| Semgrep (Community Edition) | SAST in CI | LGPL-2.1-or-later | Yes, as a tool | Run as a CI tool only; never distributed with or linked into our code. Its rule registries have their own terms, checked when rules are chosen |
| PostgreSQL † | Database | PostgreSQL License | Yes | Separate service |
| MinIO † | Local S3-compatible storage (dev only) | AGPL-3.0 | Yes, as a separate local service | Used only as an unmodified container in local Docker Compose; never distributed, linked or offered as a network service by this project. Replace with another S3-compatible store if that changes |
| Jaeger †, OWASP ZAP †, Trivy † | Local tracing, DAST, image scanning | Apache-2.0 | Yes | Tools and separate services |
| gitleaks †, lychee † | Docs CI | MIT; Apache-2.0 OR MIT | Yes | |
| GitHub Actions used in CI (`actions/checkout`, `DavidAnson/markdownlint-cli2-action`, `lycheeverse/lychee-action`, `gitleaks/gitleaks-action`) † | CI only | Per action repository | Yes, not distributed | Pinned by full-length commit SHA. The repository's Actions settings allow only GitHub-created actions and these three third-party actions, and require SHA pinning. `gitleaks-action` requires a free license key for organization-owned repositories; this repository is user-owned, so none is needed today |

**Result:** no incompatibility found for the planned stack. Two items need care: psycopg (LGPL, keep it an unmodified dependency) and MinIO (AGPL, local development only).

## 3. Content that is not code

- Framework texts and catalogs keep their own licenses and are **not** relicensed under Apache-2.0. For example, OWASP GenAI material is CC BY-SA 4.0 and NIST publications are U.S. government works; ISO/IEC standards are copyrighted and are only referenced by identifier, never quoted at length. See [framework references](framework-references.md).
- Documentation licensing is described in `LICENSE-CONTENT.md`.

## 4. Ongoing control (planned, M0-12)

- A license check runs in CI once dependencies exist (for example `pip-licenses` and `license-checker`) with an **allow-list**: MIT, BSD-2-Clause, BSD-3-Clause, ISC, Apache-2.0, PostgreSQL, Python-2.0, and LGPL for unmodified dependencies. Anything else fails the build until reviewed.
- Dependabot pull requests that change a dependency's license are reviewed by the code owner.
- The SBOM published per release lists each component's license.
