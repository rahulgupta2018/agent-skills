---
name: grounded-answer-with-citations
description: >
  Produces answers grounded in a curated knowledge base, with inline citations to the exact source
  and an authority-ranked confidence level. Activates when a user asks a domain/regulatory/policy
  question and expects a defensible, sourced answer rather than free-form generation. Owns answer
  synthesis and citation/confidence presentation. Does not own retrieval (ontology-guided-retrieval)
  or document-vs-rule comparison (policy-gap-analysis).
license: MIT
metadata:
  author: Social Housing AI (generalised for this library)
  version: "1.1.0"
  last_updated: 2026-07-02
  category: knowledge
---

# Grounded Answer with Citations

## Overview

Turns retrieved, curated context into a defensible answer where every domain/regulatory claim
carries an inline citation to the exact source plus an explicit **authority level** and
**confidence signal**, with "as at" dates and jurisdiction. The promise: the answer is only as
trustworthy as its sources, and the user can see them.

**Freedom level: LOW** — a grounding-critical procedure; follow it exactly.

**Project binding.** Load `.agents/project-context.yaml`. Rank sources by
`${ctx.authority_hierarchy}`, scope to `${ctx.jurisdictions}`, map authority→label via
`${ctx.confidence_tiers}`, apply `${ctx.guardrails}` and `${ctx.escalation_policy}`. Absent a
context file, use the generic defaults below.

## When to Activate

Activate when:
- A user asks a factual domain/regulatory/policy question and expects a sourced answer.
- The agent has retrieved context and must synthesise a final, cited response.
- A briefing, explainer, or "second pair of eyes" answer is requested.

**Do not activate** (adjacent skills own this):
- `ontology-guided-retrieval` — owns *finding and ranking* the source material.
- `policy-gap-analysis` — owns comparing a document against the rules.
- `fact-checker` — owns verifying a standalone claim (no grounding corpus).

## Core Concepts

- **Authority hierarchy** from `${ctx.authority_hierarchy}` decides precedence on conflict.
  Generic default (highest→lowest): primary law → secondary law/rules → regulator standards →
  official guidance → adjudications/determinations → best practice → general model knowledge.
  *(Example — UK social housing: statute → SI → RSH standard → GOV.UK guidance → Ombudsman
  determination → best practice.)*
- **Grounding contract**: no domain/regulatory assertion without a retrieved source. Model-only
  content is labelled as such at the lowest confidence.
- **Temporal + jurisdiction validity**: cite the version in force for the relevant date and the
  correct jurisdiction.

## Workflow

1. **Check inputs.** Confirm retrieved context exists (from `ontology-guided-retrieval`). If
   nothing relevant was found, say so — never fabricate.
2. **Screen guardrails.** Apply `${ctx.guardrails}` (default: detect PII and stop). Do not proceed
   on prohibited data.
3. **Rank sources** by authority, then in-force recency, then jurisdiction match.
4. **Draft** using only supported statements; attach a citation per claim: source, clause/section,
   "as at" date, jurisdiction.
5. **Assign confidence** (map via `${ctx.confidence_tiers}`): High = top-tier in-force,
   jurisdiction-matched; Medium = guidance/adjudication/best practice; Low = model knowledge or
   stale/uncertain sourcing.
6. **Escalate** on high stakes per `${ctx.escalation_policy}` (note: support, not professional
   advice; suggest expert review / the primary source).
7. **State gaps** explicitly — what's uncovered, and any assumed jurisdiction/date.

## Examples

```
Q: "What are the damp-and-mould investigation timescales?"  (context: UK social housing, England)
A: "Investigate within 10 working days of awareness [Hazards in Social Housing (Prescribed
    Requirements) (England) Regs 2025, in force 2025-10-27 — England]. Confidence: High.
    Support, not legal advice; verify against the SI."
```
```
Q: "Typical board approval time for a new policy?"
A: "No curated authority governs this; operational and org-specific. Confidence: Low (uncited)."
```

## Guidelines

1. Every domain/regulatory sentence has a citation or an explicit low-confidence label.
2. Authority level and confidence tier are shown, never hidden.
3. "As at" date and jurisdiction accompany each legal/regulatory citation.
4. Guardrail hit ⇒ stop and defer.
5. High-stakes answers include an escalation note.

## Gotchas

1. **Consolidated vs raw sources**: the free/raw version may not be the current consolidated text;
   prefer the curated/point-in-time source and flag if only raw text is available.
2. **Jurisdiction bleed**: never present one jurisdiction's rule as universal.
3. **Confidence inflation**: don't raise confidence because the user sounds authoritative or
   anxious — base it only on source authority.
4. **Silent staleness**: a past in-force date isn't proof of "still current" — check for later
   amendment/repeal.

## Integration

- `ontology-guided-retrieval` — supplies the ranked, provenance-rich context this skill cites.
- `policy-gap-analysis` — reuses this citation/confidence format per finding.
- `ontology-builder-assistant` — defines the authority/temporal/jurisdiction model relied on.

## References

- Reads `.agents/project-context.yaml` (authority hierarchy, confidence tiers, guardrails).
- Best practices: https://agentskills.io/skill-creation/best-practices
