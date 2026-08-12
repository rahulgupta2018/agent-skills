---
applyTo: "**"
---
# Always-on observation protocol (self-improvement)

Runs every session, passively. Capture, don't auto-apply. The user approves changes.
A committed, version-controlled copy of the same protocol held in Copilot memory at
`/memories/observation-protocol.md` (machine-local); this repo copy is the source of truth.
This repo is the **skills library** — the upstream source in the propagation chain.

## Watch for (3 signals)
1. **Corrections** — user edits/steers my output → a skill is unclear or incomplete.
2. **Gaps** — user does something manually that no skill covers → new-skill candidate.
3. **Blind spots** — this protocol or a skill misfires (wrong trigger, missing guardrail) → log it too.

## Capture (during the session)
Append terse entries to `/memories/session/observations.md` — one line each:
`[signal] <what happened> → <affected skill or "NEW: <name>"> → <suggested change>`.
No prose. Don't interrupt the task to log; capture at natural breaks.

## Promote (on review / end of session)
- Ask "anything to log?" before ending a session; flush session notes.
- Durable skill fix → propose to the user; on approval record under
  `/memories/repo/skill-updates.md` and apply to the skill.
- Cross-cutting rule (applies to many skills) → `/memories/skill-principles.md` (user scope).

## Gates (never skip)
- **Never edit a skill directly from an observation** — propose, the user approves.
- Any new/updated skill is checked against `skill-principles.md`, the `quality-governance`
  skill checklist, AND the `AUTHORING-GUIDE.md` in this repo before it's considered done.
- Vendored/adopted copies: a skill change made here (the library) must propagate downstream
  (agent-skills library → ai-software-factory vendor → project `.agents/skills/`).
