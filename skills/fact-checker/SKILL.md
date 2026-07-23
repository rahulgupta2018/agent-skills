---
name: fact-checker
description: >
  Verifies specific factual claims using evidence-based analysis and rates each claim on a
  fixed scale with sourced reasoning. Activates when a user asks to "fact check", "verify",
  "is this true", debunk a viral claim, check a statistic, or assess source credibility. Owns
  claim verification and verdict reporting. Does not own multi-source research synthesis,
  producing a cited domain answer, or copy-editing.
license: MIT
metadata:
  author: awesome-llm-apps (adapted for this library)
  version: "1.1.0"
  last_updated: 2026-07-02
  category: research
---

# Fact Checker

## Overview

Takes one or more discrete claims and returns a verdict per claim, backed by the strongest
available evidence, an explicit confidence level, and source-quality notes. Optimised for
correctness and legibility of reasoning, not persuasion.

**Freedom level: MEDIUM** — the verdict scale and output template are fixed; how you gather
and weigh evidence is up to judgement.

**Project binding.** At task start, load `.agents/project-context.yaml`. Use
`${ctx.authority_hierarchy}` for source ranking, `${ctx.sources}` as the trusted registry,
and `${ctx.guardrails.pii_handling}` / `${ctx.escalation_policy}` for handling. If the file or
a key is absent, fall back to the generic defaults documented below.

## When to Activate

Activate when:
- A user asks to verify/debunk a specific statement, statistic, or viral claim.
- A claim's truth value is in question and needs an evidence-based verdict.

**Do not activate** (adjacent skills own this):
- `grounded-answer-with-citations` — owns answering a domain question with sources (no verdict).
- `technical-writer` — owns drafting/editing prose, not adjudicating truth.
- `strategy-advisor` — owns judgement calls on strategy, not factual verification.

## Verification Procedure

1. **Isolate the claim.** Extract the exact factual assertion; separate it from opinion and
   from any implicit sub-claims. If the "claim" is an opinion or value judgement, say so and
   stop — it isn't fact-checkable.
2. **Define the deciding evidence.** State what would prove and what would disprove it, and
   which source types would be authoritative for *this* claim.
3. **Gather and weigh evidence.** Prefer primary data; check dates; note context. For
   time-sensitive claims, verify against current data — do not rely on training knowledge.
4. **Assign a verdict** from the fixed scale below, with a confidence level.
5. **Give correct information** if the claim is false/misleading, and the context needed to
   interpret it properly.

## Verdict Scale (fixed)

- ✅ **TRUE** — accurate and supported by reliable evidence
- ⚠️ **MOSTLY TRUE** — accurate but missing important context or minor errors
- 🔶 **MIXED** — contains both true and false elements
- ❌ **MOSTLY FALSE** — misleading or largely inaccurate
- 🚫 **FALSE** — demonstrably wrong
- ❓ **UNVERIFIABLE** — cannot be confirmed/denied with available evidence (state what *would* settle it)

## Source-Quality Hierarchy

Use **`${ctx.authority_hierarchy}`** from the project context when present, and prefer sources
in **`${ctx.sources}`**. Weight evidence by tier; flag when a verdict rests only on lower tiers.

**Generic fallback (no project context):** peer-reviewed studies → official statistics →
fact-checked reporting from reputable outlets → qualified expert statements → general news
(corroborate) → social media/blogs (verify independently).

## Output Template

```markdown
## Claim
[exact statement]

## Verdict: [RATING]  ·  Confidence: [High/Medium/Low]

## Analysis
[why this rating — reasoning tied to evidence]

**Evidence:**
- [key supporting/refuting item]

**Context:**
- [nuance, why it matters]

## Correct information
[accurate version, if the claim is false/misleading]

## Sources
[1] [source with tier note]
```

## Guidelines

1. One verdict per discrete claim; split compound claims.
2. Every verdict names the evidence and the source tier it rests on.
3. Time-sensitive/statistical claims are checked against current data, not memory.
4. Opinions and value judgements are identified as non-factual, not rated.

## Gotchas

1. **UNVERIFIABLE as a cop-out**: reserve it for genuinely undecidable claims; always state
   what evidence would resolve it rather than stopping at "can't tell".
2. **Settled vs contested**: don't "both-sides" scientifically settled matters; conversely,
   don't present a genuinely open question as settled.
3. **Stale facts**: leadership, prices, records, and "current" claims change — verify freshly
   rather than asserting from training data.
4. **Statistic without base rate**: a number can be technically true yet misleading; check
   denominator, timeframe, and comparison group before rating TRUE.

## Integration

- `grounded-answer-with-citations` — reuse its citation discipline when sourcing evidence.
- `strategy-advisor` — hand off once facts are established and a judgement call is needed.

## References

- Best practices: https://agentskills.io/skill-creation/best-practices
- Spec: https://agentskills.io/specification
