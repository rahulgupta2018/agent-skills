# Agent Skills — Generic, Cross-Project Library

A shared library of reusable Agent Skills. Skills are written **once, project-agnostic**, and
**picked up by any project** (social housing today, KYC/AML tomorrow). They stay reusable by
binding to a per-project **`project-context.yaml`** at runtime instead of hardcoding domain facts.

Compatible with the open [Agent Skills spec](https://agentskills.io), Claude Code / Cowork
plugins, and Google ADK 2.0 tool-loading.

---

## Core idea: mechanism vs parameters

- **Skill = the method (mechanism).** Generic and reusable. *How* to fact-check, retrieve, do gap
  analysis. Never edited per project.
- **Project context = the parameters.** Per-project *values* the skill binds to — domain,
  jurisdictions, authority hierarchy, sources, compliance rules.

> **Never fork a generic skill to make it domain-aware. Parameterise it via `project-context.yaml`.**

The same `fact-checker` runs unchanged in social housing and KYC/AML; only the context file differs.

---

## What's in this library

```
agent-skills/
├── docs/
│   └── guides/                    ← shared reference guides (e.g. ADK dev guides)
└── skills/
    ├── README.md                 ← you are here
    ├── AUTHORING-GUIDE.md        ← the standard for writing/updating skills
    ├── project-context.schema.json   ← contract that each project's context file must satisfy
    ├── project-context.example.yaml  ← copy this into a project to start
    ├── _template/                ← copy this to start a new skill
    │   └── SKILL.md
    └── <skill-name>/             ← one folder per skill
        ├── SKILL.md              ← required: instructions + frontmatter
        ├── references/           ← optional: deep docs, loaded on demand
        ├── rules/                ← optional: rule catalogues, loaded on demand
        └── scripts/              ← optional: runnable helpers
```

### Skill catalogue (by `metadata.category`)

| Category | Skills |
|---|---|
| knowledge | `grounded-answer-with-citations`, `ontology-guided-retrieval`, `policy-gap-analysis`, `ontology-builder-assistant` |
| agent | `memory-systems`, `multi-agent-patterns`, `self-improving-agent-skills`, `context-fundamentals`, `context-degradation`, `context-compression`, `context-optimization`, `filesystem-context`, `tool-design` |
| coding | `python-expert`, `fullstack-developer`, `tdd-red-green-refactor`, `typed-service-contracts` |
| planning | `project-planner`, `sprint-planner`, `strategy-advisor` |
| writing | `technical-writer` |
| design | `ux-designer` |
| data | `visualization-expert` |
| research | `fact-checker` |

Each skill's `SKILL.md` declares what it owns and a **"Do not activate"** block naming adjacent
skills, so activation stays precise as the library grows.

### Collections (bundled skill sets)

Some skills belong to a **cohesive bundle** rather than standing alone as generic skills. Their
skill folders live **flat** alongside the catalogue (so they are auto-discovered as top-level
skills) but are adopted **as a set**, not cherry-picked individually. Shared, non-skill assets for
the bundle live in a holder folder plus `docs/`.

| Collection | Skills (top-level `adk-*`) | Shared assets |
|---|---|---|
| ADK development | `adk-setup`, `adk-style`, `adk-git`, `adk-architecture`, `adk-agent-builder`, `adk-debug`, `adk-review`, `adk-sample-creator`, `adk-unit-design`, `adk-unit-guide` | `adk-agent/` holds the shared `AGENTS.md`, `CONTRIBUTING.md`, and `contributing/samples/`; reference guides live in `docs/guides/`. Meta-skills for **building on / contributing to the ADK framework** — only relevant to ADK-based projects. |

Notes on collections:
- They are **not** part of the generic catalogue above and do **not** follow the library
  frontmatter standard (no `metadata.category` etc.) — they are kept as shipped by their source.
- Adopt a collection by copying **all** its `adk-*` skill folders into the project's
  `.agents/skills/` (they land flat there) together with the bundle's shared files
  (`adk-agent/AGENTS.md`, `contributing/`, and the `docs/guides/`), and wire the bundle's
  `AGENTS.md` guidance into the project.
- Use a collection only when the project works in that ecosystem (e.g. adopt the ADK skills when the
  project is built on ADK — which the Social Housing Knowledge AI project is, per ADR-05).

---

## How to adopt skills into a project

### Step 1 — Create the project skills folder

Adopt skills into the project's agent folder (do not edit the library copies):

```
<project-root>/.agents/
├── project-context.yaml     ← the project's parameters (see Step 3)
└── skills/
    ├── fact-checker/         ← copied from this library
    ├── grounded-answer-with-citations/
    └── ...                   ← only the skills this project needs
```

Copy in **only the skills the project needs** — pick from the catalogue above. Copy the whole skill
folder (its `SKILL.md` plus any `references/`, `rules/`, `scripts/`).

> Prefer copying over symlinks so a project pins a known-good version. Record the version you copied
> (see `metadata.version` in each `SKILL.md`) in the project's `skills[]` manifest.

### Step 2 — Install any skill dependencies

Some skills bundle scripts with dependencies. Install them in the project environment if you'll run
those scripts:

- `ontology-builder-assistant/scripts/owl_to_graphrag_schema.py` → `rdflib`, `neo4j-graphrag`
- `memory-systems/scripts/memory_store.py` → `numpy`
- `multi-agent-patterns/scripts/coordination.py` → standard library only
- `self-improving-agent-skills/` → see its `README.md` (FastAPI + ADK backend, Next.js frontend)

### Step 3 — Create `project-context.yaml`

Copy `project-context.example.yaml` to `<project-root>/.agents/project-context.yaml` and fill it in.
Validate it against `project-context.schema.json`. Minimum useful fields:

```yaml
project: { name: "...", code: "..." }
domain: "..."                       # one-line domain statement
jurisdictions: [ ... ]
authority_hierarchy: [ ... ]        # highest → lowest
sources: { primary: [...], licensed: [...] }
compliance_rules: [ ... ]           # grounding, PII, escalation, audit
guardrails: { pii_handling: detect_and_stop }
escalation_policy: { triggers: [...], disclaimer: "..." }
confidence_tiers: { high: "...", medium: "...", low: "..." }
tech_bindings: { graph_store: "...", llm: "...", cache: "..." }
skills:                             # manifest of adopted skills + overlay
  - { name: fact-checker, enabled: true, priority: FF }
  - { name: grounded-answer-with-citations, enabled: true, layer: "...", priority: V1 }
meta: { version: "0.1.0", last_updated: "YYYY-MM-DD", owner: "..." }
```

Validate:

```bash
python3 -c "import json,yaml,jsonschema; \
jsonschema.Draft202012Validator(json.load(open('project-context.schema.json'))).validate(\
yaml.safe_load(open('.agents/project-context.yaml')))" && echo VALID
```

### Step 4 — Wire the harness

Point the project's `AGENTS.md` / `CLAUDE.md` at the context file so it loads **before** skills bind:

```markdown
## Project context
The source of truth for project parameters is `.agents/project-context.yaml`.
Load it at the start of every task; skills resolve `${ctx.*}` references from it.
```

That's it — skills now behave in a project-aware way with no edits to the skills themselves.

---

## How skills read project context

Skills reference project values with the `${ctx.<path>}` convention, e.g.
`${ctx.authority_hierarchy}`, `${ctx.jurisdictions}`, `${ctx.sources.primary}`,
`${ctx.guardrails.pii_handling}`, `${ctx.escalation_policy}`, `${ctx.tech_bindings.cache}`.

**Resolution + fallback:** the agent loads `.agents/project-context.yaml`; a skill resolves each
`${ctx.*}` from it. **If the file or a key is missing, the skill uses its documented generic
default**, so every skill still works standalone.

**Precedence:** per-skill `overrides` (in the manifest) → project `project-context.yaml` → skill's
generic default.

---

## Where things live (summary)

| File | Location | Purpose |
|---|---|---|
| Skill definitions | `agent-skills/skills/<name>/SKILL.md` (library) | Reusable methods — never edited per project |
| Copied skills | `<project>/.agents/skills/<name>/` | The versions a project actually uses |
| Project parameters | `<project>/.agents/project-context.yaml` | Domain, jurisdictions, authority, sources, rules |
| Context contract | `agent-skills/skills/project-context.schema.json` | Schema every project context is validated against |
| Harness wiring | `<project>/AGENTS.md` or `CLAUDE.md` | Points the agent at the context file |

---

## Adding or updating a skill

1. Copy `_template/` to `<skill-name>/` and follow **`AUTHORING-GUIDE.md`** (frontmatter with
   `category`, third-person description, "Do not activate", gotchas, integration, `${ctx.*}` binding
   with a generic fallback).
2. Keep `SKILL.md` under ~500 lines; move depth to `references/` with explicit "Load when …" pointers.
3. Bump `metadata.version` and `last_updated` on any behaviour-changing edit; deprecate rather than delete.
4. Run the authoring checklist in `AUTHORING-GUIDE.md` before committing.

**Do not** put project-only fields (`layer`, `priority`) in a library `SKILL.md` — those belong in a
project's `project-context.yaml` under `skills[]`.

---

## Conventions at a glance

- Folder name == `metadata.name`, kebab-case.
- Library metadata: `author`, `version`, `last_updated`, `category`.
- Compliance behaviour is parameterised (`${ctx.*}`), never hardcoded.
- Reference docs open with `# Title` and a `> **Load when: …**` pointer.

See `AUTHORING-GUIDE.md` for the full standard and `project-context.example.yaml` for a ready-to-copy
starting point.

## New Skills in the repo
1. skills/agent-discovery 
2. skills/quality-governance