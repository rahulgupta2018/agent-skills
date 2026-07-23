# Agent Skill Authoring Guide — Standard for the generic skills library

*Last updated: 2026-07-02 · Applies to every skill in this library*

This is the **standard** for the **generic, cross-project Agent Skills library**. Skills here are written once, project-agnostic, and **picked up by any project** (social housing today, KYC/AML tomorrow). They stay reusable by binding to a per-project **`project-context.yaml`** at runtime rather than hardcoding domain specifics (see §11). It is compatible with the open [Agent Skills spec](https://agentskills.io), Claude Code / Cowork plugins, and Google ADK 2.0 tool-loading.

Core principle — **mechanism vs parameters**: a skill encodes the reusable *method*; the project context supplies the *values* (domain, jurisdiction, authority hierarchy, sources, compliance rules). Never fork a generic skill to make it domain-aware; parameterise it.

It was distilled from three reviewed references: the *Agent Skills for Context Engineering* template (structure, ownership boundaries, <500-line rule), the *Redis Agent Skills* pack (richer frontmatter), and *awesome-llm-apps / awesome_agent_skills* (spec-conformant examples), plus [agentskills.io best practices](https://agentskills.io/skill-creation/best-practices).

---

## 1. What a skill is

A skill is a **packaged, discoverable capability**: a `SKILL.md` (instructions) plus optional `references/` (deep material loaded on demand) and `scripts/` (helper code). The agent reads the frontmatter to decide *when* to activate, then loads the body only when relevant. Skills keep the context window lean by moving detail out of the system prompt and into files fetched just-in-time.

## 2. Directory layout (mandatory)

```
.agents/skills/
  <skill-name>/
    SKILL.md            # required — instructions + frontmatter
    references/         # optional — deep docs, loaded on demand
      <topic>.md
    scripts/            # optional — runnable helpers (Python, etc.)
      <script>.py
```

- Folder name = the skill `name` = **kebab-case**, and must match the frontmatter `name` exactly.
- Keep `SKILL.md` **under ~500 lines**. Anything longer, or reference-only, goes in `references/`.

## 3. Frontmatter (mandatory schema)

```yaml
---
name: policy-gap-analysis                 # required · kebab-case · == folder name
description: >                            # required · third person · triggers + ownership
  One-paragraph description written for discovery. State what the skill DOES,
  the concrete triggers that should activate it, and where its boundary ends.
license: MIT                              # optional but recommended
metadata:                                 # library metadata (standardised keys)
  author: <author or attribution>
  version: "0.1.0"                        # semver; bump on meaningful change
  last_updated: 2026-07-02
  category: knowledge                      # research | coding | knowledge | writing | planning | agent | data | design
---
```

Rules for `description` (this field is injected into the system prompt — it decides discovery):
- **Third person, present tense.** "Compares a policy against current regulation…" — never "I can…".
- Lead with the capability, then list **explicit triggers** (keywords, task types), then the **ownership boundary**.
- Keep it dense and specific; vague descriptions cause wrong-skill activation.

### `metadata` keys — generic library vs project overlay

Because this is a **generic, cross-project library**, skill files carry only library metadata; project-specific fields are applied when a project *adopts* a skill — never baked into the shared file.

- **In the skill file (every skill):** `author`, `version`, `last_updated`, and **`category`** (`research | coding | knowledge | writing | planning | agent | data | design`). Do **not** put `layer`/`priority` here.
- **Project overlay (not in the skill file):** `layer` (a project's architecture layer) and `priority` (`V1 | FF | Later`) live in that project's `.agents/project-context.yaml` under `skills[]`. This keeps the shared skill byte-identical across projects.

## 4. Body sections (standard order)

Use these headings. Omit a section only if it genuinely has no content.

1. **Overview** — 2–4 sentences: what this skill does and the outcome it produces.
2. **When to Activate** — direct triggers and indirect signals. **Must** include a **"Do not activate"** block naming adjacent skills that own nearby work (prevents broad skills stealing activation).
3. **Core Concepts** — only the mental models the agent doesn't already have. Challenge every line: "does the agent really need this?" Don't explain what a capable model already knows.
4. **Workflow / Detailed Topics** — the executable steps, decision tables, or ordered procedure. Match specificity to fragility (see §5).
5. **Practical Guidance** — patterns, anti-patterns, decision frameworks.
6. **Examples** — concrete input → output pairs, including edge cases.
7. **Guidelines** — numbered, verifiable rules.
8. **Gotchas** — numbered, experience-derived failure modes. Highest-signal content; keep each specific and non-overlapping.
9. **Integration** — related skills (plain text, not links) and how they hand off.
10. **References** — links to this skill's own `references/*.md`, related skills, and external sources.

## 5. Freedom level (calibrate your instructions)

State how prescriptive the skill is, because it changes how the agent follows it:

- **High freedom** — many valid approaches; give principles, let the agent choose.
- **Medium freedom** — a preferred pattern exists; some variation is fine.
- **Low freedom** — the operation is fragile; the exact sequence must be followed (e.g. legal sign-off gating, PII stripping).

Compliance-critical skills in this project (grounding, citations, PII, sign-off) are **low freedom** — write them as strict procedures.

## 6. Compliance behaviour — parameterised, not hardcoded

Compliance-sensitive skills must enforce rules, but the **specifics come from the project**, not the skill. Write the skill to read them from `project-context.yaml` and fall back to a sensible generic default when absent:

1. **Ground and cite.** Never assert a domain/regulatory fact without a source and an authority level taken from `${ctx.authority_hierarchy}`. Uncited/model-only content is flagged at low confidence.
2. **Respect authority precedence + temporal/jurisdiction validity.** Prefer higher-authority, in-force, jurisdiction-matched sources (`${ctx.jurisdictions}`); surface "as at" dates.
3. **Honour data guardrails.** Apply `${ctx.guardrails.pii_handling}` (default: detect and stop). Never process prohibited data.
4. **Stay a support tool.** Apply `${ctx.escalation_policy}` on high-stakes outputs and append its disclaimer; the skill never makes the final decision.

A skill written this way behaves correctly in social housing *and* KYC/AML with no edits — only the context file changes.

## 7. Naming conventions

- Verb-led or capability-led kebab-case: `policy-gap-analysis`, `ontology-guided-retrieval`, `grounded-answer-with-citations`.
- One capability per skill. If a skill spans two clearly separable jobs, split it and cross-link via Integration.
- Prefer narrow, well-bounded skills over broad ones — narrow skills activate more accurately.

## 8. Versioning & lifecycle

- Semantic version in `metadata.version`; bump and update `last_updated` on any behaviour-changing edit.
- Deprecate rather than delete: note deprecation at the top of `SKILL.md` and point to the replacement.

## 9. Authoring checklist (run before committing a skill)

- [ ] Folder name == `name`, kebab-case.
- [ ] `description` is third-person, has explicit triggers and an ownership boundary.
- [ ] Frontmatter has `license` + `metadata` (author, version, last_updated, **category**) — no `layer`/`priority`.
- [ ] Domain specifics are read from `project-context.yaml` (§11), not hardcoded; generic fallback documented.
- [ ] Body ≤ ~500 lines; deep material moved to `references/` with explicit "load when X" pointers.
- [ ] "Do not activate" block names the adjacent skills.
- [ ] Freedom level is appropriate; compliance behaviour (§6) is parameterised.
- [ ] Gotchas are specific and non-overlapping.
- [ ] Integration lists real sibling skills.

## 10. Starting a new skill

Copy `_template/` to `<skill-name>/`, fill in the frontmatter and sections, delete unused placeholders, and run the checklist. See the seed skills `grounded-answer-with-citations`, `ontology-guided-retrieval`, and `policy-gap-analysis` for worked examples of this standard.

## 11. Project context binding (mechanism vs parameters)

Generic skills stay reusable by binding to a per-project context file at runtime.

**The file.** Each project has `.agents/project-context.yaml`, validated against `project-context.schema.json` (in this library). It holds the project's `domain`, `jurisdictions`, `authority_hierarchy`, `sources`, `glossary`, `compliance_rules`, `guardrails`, `escalation_policy`, `confidence_tiers`, `tenancy`, `tech_bindings`, and a `skills[]` manifest (which skills the project adopts + `layer`/`priority`/`overrides`). Start from `project-context.example.yaml`.

**Reference convention.** In a skill body, refer to context values with `${ctx.<path>}` (documentation notation), e.g. `${ctx.authority_hierarchy}`, `${ctx.jurisdictions}`, `${ctx.sources.primary}`. This signals the value is project-supplied.

**Resolution + fallback (state this in the skill).** The agent loads `.agents/project-context.yaml` at the start of a task; a skill resolves each `${ctx.*}` from it. **If the file or key is missing, the skill uses its documented generic default** — so every skill still works standalone. Per-skill `overrides` in the manifest win over shared context values.

**Precedence.** per-skill `overrides` → project `project-context.yaml` → skill's generic default.

**Harness wiring.** Point the project's `AGENTS.md` / `CLAUDE.md` at `project-context.yaml` as the source of truth so context loads before skills bind (progressive disclosure — the skill says *when* to read deeper files).

Net effect: the same `fact-checker` (or any skill) runs unchanged across social-housing and KYC/AML; only `project-context.yaml` differs.
