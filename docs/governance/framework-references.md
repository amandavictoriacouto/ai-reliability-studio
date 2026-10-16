# Framework References, Licensing and Interpretation Limits

| Reference | How we use it | Licensing / interpretation notes |
|---|---|---|
| **NIST AI RMF 1.0** and Playbook | Function/category/subcategory identifiers in mappings; paraphrased guidance | US Government publication; generally not subject to US copyright. Verify on the NIST site before quoting at length; always attribute. |
| **OWASP Top 10 for LLM Applications 2025** (OWASP GenAI Security Project) | Risk identifiers (LLM01–LLM10) and short attributed descriptions | OWASP content is published under a Creative Commons license (typically CC BY-SA 4.0); verify per document. Adapted text carries attribution and share-alike. |
| **ISO/IEC 42001:2023**, **ISO/IEC 27001:2022** | Clause and Annex A control **identifiers** plus original one-line summaries | Copyrighted, paid standards. **No reproduction of standard text.** Organisations with a licence may attach their own notes privately. |
| **LGPD** (Lei nº 13.709/2018) | Article references and short quotes | Statute; mappings are informational and do not constitute legal advice. |
| **EU AI Act** (Regulation (EU) 2024/1689) | Article references; obligation dates stored as data with source and date checked | Dates affected by the Digital Omnibus (provisional agreement May 2026); re-verify publication in the Official Journal before relying on them. Not legal advice. |

## Rules for contributors

1. Every mapping cites the reference identifier and version.
2. Dates and obligations are data in `catalog/`, with `source_url` and `verified_on` fields.
3. Changes to `catalog/` and `catalog/policies/` require code-owner review.

## Relation to the code license

The repository's code is licensed under Apache-2.0 (decision Q1). Framework material keeps its own license and is never relicensed. Dependency licenses are tracked in [third-party license compatibility](third-party-licenses.md).
