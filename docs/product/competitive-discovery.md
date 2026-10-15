# Competitive Discovery

> Status: **Draft** · Research date: 2026-10-15
> **Evidence** = a claim backed by a cited public source (vendor page, press, standard body). Vendor pages are *self-reported marketing*; none of these products was hands-on tested.
> **Hypothesis** = our interpretation, to be validated. Every hypothesis has a validation method and a kill criterion.

## 1. Market map

| Category | Examples | Primary buyer / user | Unit of analysis |
|---|---|---|---|
| A. AI governance platforms | Credo AI, OneTrust AI Governance, Trustible, Holistic AI | GRC, legal, Responsible AI office | AI use case / model inventory (org level) |
| B. Compliance automation | Vanta, Drata, Secureframe | Security & compliance leads | Organisation / framework (SOC 2, ISO) |
| C. Secure-by-design & threat modeling | Security Compass SD Elements, IriusRisk | Security architects, AppSec, devs | Application / architecture |
| D. AI security testing & runtime | Promptfoo, Giskard, Lakera (Check Point), Protect AI (Palo Alto Networks) | AppSec, ML engineers | Model / LLM app endpoint |
| E. Product tooling (adjacent) | Jira, Linear, Productboard | PMs, engineers | Feature / issue |

## 2. Evidence (sourced)

### A. AI governance platforms

- **Credo AI** describes an AI registry (use cases, models, vendors), shadow-AI discovery, dynamic risk scoring, audit trails, "policy packs", a risks-and-controls library, an AI assistant for intake and risk/control mapping, and pre-built support for EU AI Act, NIST AI RMF, ISO 42001 and SOC 2. Source: [credo.ai/solutions/risk-management](https://www.credo.ai/solutions/risk-management).
- **OneTrust AI Governance** and **Trustible** are compared head-to-head by analysts and review sites, and Gartner Peer Insights lists an "AI Governance Platforms" market. Sources: [Gartner Peer Insights comparison](https://www.gartner.com/reviews/market/ai-governance-platforms/compare/product/onetrust-ai-governance-vs-trustible-responsible-ai-governance), [The CTO Club](https://thectoclub.com/tools/trustible-vs-onetrust/), [Holistic AI alternatives (Gartner)](https://external.pi.gpi.aws.gartner.com/reviews/product/holistic-ai-1391980002/alternatives).

### B. Compliance automation

- **Vanta ISO 42001** describes hourly automated control tests, integrations (cloud, code, identity, GenAI tools such as OpenAI) for automatic evidence collection, pre-mapped AI controls and risk scenarios, evidence reuse across EU AI Act and NIST AI RMF, an AI agent that flags evidence gaps, and auditor access. Source: [vanta.com/products/iso-42001](https://vanta.com/products/iso-42001).
- Vanta reports holding ISO 42001 certification itself. Source: [Vanta](https://vanta.com/resources/vanta-earns-iso-42001-certification).

### C. Secure-by-design & threat modeling

- **SD Elements** (Security Compass) says it generates actionable security requirements *before code is written*, offers threat-modeling diagrams, portfolio dashboards, compliance content for 80+ regulations/guidelines, and just-in-time training. Source: [Security Compass press release](https://www.securitycompass.com/company-news/security-compass-releases-new-developer-centric-threat-modeling-capabilities-in-sd-elements-in-support-of-new-secure-software-development-guidelines/).
- **IriusRisk "Jeff"** turns text, images, user stories, docs, source code or SBOMs into an architecture diagram and threat model; notably, *threats and countermeasures come from a rules engine, not from the AI*; it matches components via RAG and offers compliance views. Sources: [iriusrisk.com/ai-threat-modeling](https://iriusrisk.com/ai-threat-modeling), [SecurityBrief](https://securitybrief.co.uk/story/iriusrisk-unveils-jeff-ai-tool-for-threat-modelling-images).

### D. AI security testing & runtime

- **Promptfoo** documents red-team plugins and presets aligned to OWASP LLM Top 10, OWASP API Top 10 and OWASP Top 10 for Agentic Applications. Sources: [OWASP LLM Top 10 docs](https://www.promptfoo.dev/docs/red-team/owasp-llm-top-10/), [plugins](https://www.promptfoo.dev/docs/red-team/plugins/).
- **Giskard** is an open-source testing framework for LLM and ML models (vulnerability scans, RAG evaluation). Source: [Giskard docs](https://docs.giskard.ai/en/latest/getting_started/quickstart/quickstart_llm.html).
- **Consolidation:** Palo Alto Networks announced a definitive agreement to acquire Protect AI (Apr 2025) — [Globes](https://en.globes.co.il/en/article-1001509080); Check Point announced the acquisition of Lakera (Sep 2025), whose products cover pre-deployment assessment (Red) and runtime protection (Guard) — [CyberScoop](https://cyberscoop.com/check-point-lakera-acquistion-ai-security/).

### Reference landscape

- OWASP published the **Top 10 for LLM Applications 2025**. Source: [genai.owasp.org](https://genai.owasp.org/2024/11/17/owasp-reveals-updated-2025-top-10-risks-for-llms-announces-new-llm-project-sponsorship-program-and-inaugural-sponsors/).
- **EU AI Act Digital Omnibus**: provisional political agreement 6 May 2026, Annex III high-risk obligations to 2 Dec 2027 and Annex I to 2 Aug 2028, effective only once published in the Official Journal. Source: [Gibson Dunn](https://www.gibsondunn.com/eu-ai-act-omnibus-agreement-postponed-high-risk-deadlines-and-other-key-changes/). *Action: confirm publication status before using dates in the product.*

## 3. Comparative view (evidence-based where cited, otherwise marked)

| Dimension | A. AI governance | B. Compliance automation | C. Secure design | D. AI security testing | **AI Reliability Studio (target)** |
|---|---|---|---|---|---|
| Primary user | GRC / legal | Compliance / security | Security / devs | AppSec / ML eng | **PM / PO / AI PM** |
| Unit | AI use case (org) | Framework (org) | Application | Endpoint / model | **Feature / release** |
| Automated verification | Partial (claimed) | ✅ strong (claimed) | Requirements, not verification* | ✅ strong (technical) | ✅ narrow but evidence-linked |
| AI in workflow | Intake & mapping | Gap flagging | Diagram → threat model | Attack generation | Risk discovery, grounded in catalog |
| Release gate | Approvals | — * | — * | CI eval possible | ✅ **policy-based gate in CI** |
| Product vocabulary | Low* | Low* | Medium* | Low* | **High (design goal)** |

\* = our assessment from public pages, not verified evidence.

## 4. Hypotheses (to validate)

| ID | Hypothesis | Validation method | Kill / pivot criterion |
|---|---|---|---|
| H1 | PMs/AI PMs feel accountable for AI-feature risk but lack a tool and vocabulary to act on it. | 10–12 interviews (PMs, AI PMs) in BR + EU-exposed companies; Mulheres de Produto / Product Camp network. | < 4 of 10 report this as a top-5 pain. |
| H2 | Security/privacy review happens late and causes release delays or scope cuts. | Interviews + ask for last 3 launches' timelines. | Most teams report reviews at discovery already. |
| H3 | Categories A/B are perceived as "for compliance people" and are not used by PMs day to day. | Interviews with PMs at companies that own Vanta/Drata/OneTrust. | PMs already use them for feature-level decisions. |
| H4 | "Evidence-linked verification" is a stronger trust signal for buyers than reports/questionnaires. | Show 2 mock reports (asserted vs evidence-linked) to 5 security/GRC people. | No preference or preference for the simpler one. |
| H5 | A CI release gate is acceptable to engineering teams if it explains *why* and allows audited overrides. | Prototype gate output reviewed by 5 engineers. | Majority would disable it. |
| H6 | The white space is the *workflow* (feature-level, PM-owned), not the testing technology, which incumbents are consolidating. | Supported indirectly by acquisitions (evidence D). Validate via willingness-to-pay conversations. | Buyers only value testing depth. |
| H7 | Brazil (LGPD + ANPD AI guidance) and EU exposure create a reachable beachhead. | Interviews + regulatory scan. | No urgency in BR market within 12–18 months. |

## 5. Implications for MVP

1. **Don't compete on testing depth.** Integrate open tooling (e.g., Promptfoo-style probe suites) as verifiers and compete on the requirement → evidence → decision chain. *(Follows from H6, evidence D.)*
2. **Borrow the IriusRisk pattern:** AI proposes, a deterministic catalog/rules layer is the source of truth. This reduces hallucinated controls and is easier to defend in an audit.
3. **Make the release gate the wedge.** It is a concrete moment where PM, engineering and security meet, and it produces a demonstrable artifact for the portfolio.
4. **Expect incumbents to move down-market.** Our defensibility must be UX for product teams + traceability, not framework coverage.

## 6. Research backlog

- Hands-on trials (free tiers / demos) of Vanta, Credo AI, IriusRisk Community, Promptfoo; capture screenshots and workflows.
- Pricing pages snapshot (not collected in this pass).
- Brazilian landscape (local GRC/LGPD tools, ANPD AI guidance status).
- Interview guide v1 (see [roadmap](../roadmap.md), M0).
