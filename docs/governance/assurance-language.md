# Assurance Language

AI Reliability Studio must never overstate what it knows. These rules apply to the UI, reports, documentation, marketing and AI-generated text.

## Status vocabulary

| Status | Meaning | Required data |
|---|---|---|
| **Verified** | An automated or manual verification method was executed and passed. | Method, version, timestamp, evidence (hash), executor |
| **Attested** | A named person stated that the requirement is met; not independently verified. | Person, role, statement, timestamp |
| **Failed** | A verification was executed and did not pass. | Method, result details, evidence |
| **Not verified** | No valid verification or attestation exists (including expired evidence). | — |
| **Not applicable** | Requirement excluded from scope with justification. | Justification, approver |

Each status also records its **method** (automated test, human review or documentary evidence); see [verification methods](verification-methods.md).

## Forbidden claims

"Secure", "compliant", "certified", "guaranteed", "100% safe", "audit-proof", or any compliance score presented as a certification.

## Required in every report

1. Scope (product, release, commit, catalog and policy versions).
2. Counts per status.
3. Evidence index with hashes and validity.
4. **Limitations statement:** verification proves specific, scoped facts at a point in time; it does not prove the absence of vulnerabilities or legal compliance. Framework mappings are informational, not legal advice.

## AI-generated content

Always labeled as a suggestion, with model, provider and prompt version available on demand. Suggestions become facts only through a recorded human action.
