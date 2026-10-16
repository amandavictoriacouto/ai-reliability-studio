# Parallel User Discovery Track

> Status: **Proposed** (M0) · Related: [competitive discovery](competitive-discovery.md) hypotheses H1–H7, [roadmap](../roadmap.md)
> Principle: **discovery informs product-shaped bets; it does not block no-regret engineering.**

## 1. What engineering can do without waiting

| Work | Why it is no-regret | Milestone |
|---|---|---|
| Repository, CI, security scanning, scaffold | Needed for any product | M0 |
| Identity, workspaces, RBAC, tenant isolation (RLS) | Any B2B SaaS needs them | M1 |
| Hash-chained audit log, evidence store | The core trust primitives, whatever the user segment | M1–M3 |
| Observability, deployment pipeline | Operational baseline | M1 |
| AI gateway with provenance and eval harness | Needed for any AI feature | M2 |

## 2. What waits for a discovery checkpoint

| Decision point | Before starting | Hypotheses | Evidence needed | If invalidated |
|---|---|---|---|---|
| **DP1** | M2 UX (risk discovery flow, PM-facing vocabulary) | H1, H2, H3 | ≥ 8 problem interviews (≥ 5 PMs/AI PMs) | Shift the primary persona to security/GRC with a PM-facing view; keep the core engine |
| **DP2** | M4 verifier selection, M5 gate UX | H4, H5 | Mock report test (5 security/GRC), gate output test (5 engineers) | Change the verifier priorities; make the gate advisory-only by default |
| **DP3** | Public v0.1 portfolio positioning and demo documentation | H6, H7 | ≥ 5 concept-validation conversations; target-user signals | Re-position the educational demo around the strongest validated workflow |

Checkpoints are 30-minute reviews with the product owner. The outcome is recorded in [decisions and open questions](decisions-and-open-questions.md).

## 3. Cadence and methods

| Weeks (relative to M0 approval) | Activity | Output |
|---|---|---|
| 1 | Interview guide v1, screener, consent form (LGPD), recruiting post | `research/interview-guide.md` |
| 2–5 | 2–3 problem interviews per week (30 min). Ask for artifact walkthroughs (last 3 launches: when did security, privacy or AI review happen?) | Anonymized notes (private), weekly synthesis |
| 5 | **DP1** synthesis | Persona and JTBD update, decision recorded |
| 6–9 | Concept tests: mock "asserted vs evidence-linked" reports (H4); gate output mock in a PR (H5) | Test results |
| 9 | **DP2** | Verifier and gate decisions |
| 10+ | Concept-validation conversations, voluntary demo feedback (using synthetic or authorised test data) | Research feedback synthesis, **DP3** |

**Recruiting channels:** Brazilian and international product communities and events, LinkedIn outreach to AI PMs, security/GRC communities. Target mix: 60% PM/PO/AI PM, 25% engineering/security, 15% GRC/privacy. Prefer companies shipping LLM features in the last 12 months.

## 4. Research operations and privacy

- Participants give informed consent (purpose, recording, retention, withdrawal) under LGPD. Recordings are optional and deleted after synthesis, within 90 days at most.
- **Raw notes never go into the public repository.** Only anonymized syntheses are published in `docs/product/research/`, with no names, companies or identifying details unless participants explicitly agree.
- Each synthesis maps findings to hypotheses as: supports, contradicts or inconclusive, with counts. No cherry-picked quotes without context.

## 5. Metrics for the track

Interviews completed per week · % of hypotheses with a clear verdict at each decision point · number of voluntary demo reviewers · number of insights that changed a backlog item (shows that discovery actually influences delivery).

## 6. Interview guide outline (v1)

1. Context: role, team, AI features shipped recently.
2. Last launch story: who raised risks, when, what was blocked, what changed.
3. Current tools and artifacts for security, privacy and AI governance.
4. Accountability: who signs off and what evidence they rely on.
5. Pain ranking (top 5 problems around launches).
6. Reaction to concept (shown only at the end, to avoid bias): a feature-level risk-to-release workflow.
7. Wrap-up: referrals, openness to a pilot.
