# agent-skills

A shared, **project-agnostic** library of reusable [Agent Skills](https://agentskills.io) —
written once and picked up by any project. Skills stay reusable by binding to a per-project
**`project-context.yaml`** at runtime instead of hardcoding domain facts.

Compatible with the open Agent Skills spec, Claude Code / Cowork plugins, and Google ADK 2.0
tool-loading.

## Core idea: mechanism vs parameters

- **Skill = the method (mechanism).** Generic and reusable — *how* to fact-check, retrieve, review
  code, build an ontology. Never edited per project.
- **Project context = the parameters.** The per-project *values* a skill binds to (domain,
  jurisdictions, authority hierarchy, sources, compliance rules).

> Never fork a generic skill to make it domain-aware. Parameterise it via `project-context.yaml`.

The same `fact-checker` runs unchanged in social housing and KYC/AML; only the context file differs.

## Layout

```
agent-skills/
├── docs/guides/         ← shared reference guides (e.g. ADK dev guides)
└── skills/
    ├── README.md                     ← full library overview (start here)
    ├── AUTHORING-GUIDE.md            ← the standard for writing/updating skills
    ├── project-context.schema.json   ← contract each project's context must satisfy
    ├── project-context.example.yaml  ← copy into a project to start
    ├── _template/                    ← copy to start a new skill
    └── <skill-name>/                 ← one folder per skill (SKILL.md + optional references/)
```

## Using a skill

1. Copy the skill folder into your project (or reference the library).
2. Provide a `project-context.yaml` that satisfies `skills/project-context.schema.json`.
3. The host (Claude Code, ADK, etc.) loads `SKILL.md`; the skill resolves `${ctx.*}` values from
   your context file.

## Docs

- [skills/README.md](skills/README.md) — full library overview and the list of skills.
- [skills/AUTHORING-GUIDE.md](skills/AUTHORING-GUIDE.md) — how to author or update a skill.

## License

See individual skill folders for provenance; ported skills retain their upstream attribution.
