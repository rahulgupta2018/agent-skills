---
name: policy-gap-analysis
description: >
  Compares an organisation's document (policy, procedure, strategy) against the current authoritative
  rules for its topic, and reports gaps, non-compliances, and recommended changes — each with a
  citation and authority level. Activates when a user uploads/references a policy and asks to check
  it against regulation, prepare for inspection/audit, or find where it falls short. Owns
  document-vs-rule comparison. Does not own general Q&A (grounded-answer-with-citations) or retrieval
  (ontology-guided-retrieval).
license: MIT
metadata:
  author: Social Housing AI (generalised for this library)
  version: "1.1.0"
  last_updated: 2026-07-02
  category: knowledge
---

# Policy Gap Analysis

## Overview

Takes a customer document and checks it against the in-force authoritative position for its topic,
producing a structured gap report: compliant / missing / outdated / weaker / ambiguous — each
finding cited and confidence-rated. Recurring, audit-visible, and high-value across regulated
domains (housing compliance, KYC/AML, data protection, etc.).

**Freedom level: LOW** — findings must be evidence-backed and cited; follow the procedure.

**Project binding.** Load `.agents/project-context.yaml`. Compare against `${ctx.authority_hierarchy}`
sources scoped to `${ctx.jurisdictions}`; apply `${ctx.guardrails}` and `${ctx.escalation_policy}`.
Absent a context file, elicit the applicable rules from the user.

## When to Activate

Activate when:
- A user uploads/pastes a policy/procedure/strategy/template and asks to check it.
- A user asks "where does our X fall short of current regulation?" or prepares for inspection/audit.
- A regulatory change prompts a "does this affect our policy?" review.

**Do not activate** (adjacent skills own this):
- `grounded-answer-with-citations` — owns answering a standalone question (no document diff).
- `ontology-guided-retrieval` — owns fetching the authoritative position (this skill calls it).
- `ontology-builder-assistant` — owns ontology design.

## Core Concepts

- **Requirement extraction**: turn regulation into discrete, checkable requirements.
- **Coverage mapping**: match each requirement to the document text that satisfies it (or none).
- **Gap types**: *missing*, *outdated* (references superseded rules), *weaker than required*,
  *ambiguous*.
- **Jurisdiction + temporal fit**: check against the rules in force for the org's jurisdiction and
  the relevant date.

## Workflow

1. **Screen guardrails.** Apply `${ctx.guardrails}` (default: detect PII and stop).
2. **Scope** — topic(s), the org's jurisdiction (`${ctx.jurisdictions}`), applicable standards.
3. **Fetch the authoritative position** via `ontology-guided-retrieval` per topic (authority,
   jurisdiction, in-force dates).
4. **Extract requirements** as a checklist.
5. **Map document → requirements** — locate the clause that addresses each; record exact text or
   "not found".
6. **Classify** each finding (compliant/missing/outdated/weaker/ambiguous).
7. **Recommend a change** per gap, citing the governing rule (title, clause, "as at" date,
   jurisdiction) + authority; reuse the format from `grounded-answer-with-citations`.
8. **Summarise** — prioritised gap list (severity × authority), overall readiness signal, and an
   escalation note per `${ctx.escalation_policy}`.

## Example

```
Input: "Damp & Mould Policy v3" (UK social housing, England HA) — check against Awaab's Law.
Output:
 • MISSING: no 10-working-day investigation timescale → add it
   [Hazards in Social Housing (Prescribed Requirements)(England) Regs 2025, in force 2025-10-27 —
    England]. Authority: SI. Confidence: High. Severity: High.
 • OUTDATED: cites pre-2024 consumer standards → update to the four current standards.
 Overall: Not inspection-ready — 2 high-severity gaps. Support only; take legal advice.
```

## Guidelines

1. Every gap finding cites the governing rule and shows authority + confidence.
2. Findings are classified and severity-ranked.
3. Comparison is against in-force, jurisdiction-matched rules.
4. Guardrail hit ⇒ stop and defer.
5. Output includes an overall readiness signal and an escalation note.

## Gotchas

1. **False "compliant"**: the document mentions a topic but doesn't meet the requirement — check
   substance, not keyword presence.
2. **Wrong jurisdiction baseline**: comparing against a generic baseline instead of the org's
   jurisdiction.
3. **Stale rule**: comparing against a superseded version — confirm in force as at the review date.
4. **Over-reach**: recommending changes with no cited requirement — keep findings tied to rules.

## Integration

- `ontology-guided-retrieval` — fetches the current authoritative position per topic.
- `grounded-answer-with-citations` — supplies the citation/confidence format per finding.
- `ontology-builder-assistant` — models the requirement/authority/temporal structure used.

## References

- Reads `.agents/project-context.yaml` (authority hierarchy, jurisdictions, guardrails, escalation).
- Best practices: https://agentskills.io/skill-creation/best-practices
