---
name: memory-systems
description: >
  Designs persistent memory for agent systems — cross-session retention, entity tracking,
  temporal validity, graph/vector retrieval, consolidation, and framework selection. Activates
  when an agent must remember across sessions, maintain entity identity, reason over accumulated
  knowledge, or choose a memory framework. Owns persistent semantic memory. Does not own
  multi-agent orchestration or the skill self-improvement loop.
license: MIT
metadata:
  author: Agent Skills for Context Engineering (adapted for this library)
  version: "4.2.0"
  last_updated: 2026-07-02
  category: agent
---

# Memory System Design

## Overview

Memory is the persistence layer that lets agents keep continuity across sessions and reason over
accumulated knowledge. The design rule is **start shallow, add structure only when retrieval
quality demands it**: volatile context → session store → cross-session key-value → entity registry
→ temporal knowledge graph. Escalate a layer only when the simpler one measurably fails.

**Freedom level: MEDIUM** — the escalation ladder is the spine; framework choice is contextual.

**Project binding.** Bind concrete stores to `${ctx.tech_bindings}` — e.g. cache
(`${ctx.tech_bindings.cache}`), graph store, and a **single pinned** `embedding_model`. If absent,
prototype on the filesystem.

## When to Activate

Activate when:
- Building agents that persist knowledge across sessions or maintain entity identity.
- Choosing a memory framework (Mem0, Zep/Graphiti, Letta, LangMem, Cognee).
- Adding reasoning over accumulated knowledge or temporal ("as at") queries.

**Do not activate** (adjacent skills own this):
- `multi-agent-patterns` — owns orchestration/handoff between agents (even if they share memory).
- `self-improving-agent-skills` — owns evolving the skills themselves, not runtime memory.
- `ontology-guided-retrieval` — owns querying a curated knowledge graph for grounding answers.

## Memory Layers (pick the shallowest that works)

| Layer | Persistence | Implementation | Use when |
|---|---|---|---|
| Working | Context window | Scratchpad in prompt | Always; keep in attention-favoured positions |
| Short-term | Session | Cache / files | Tool results, conversation state |
| Long-term | Cross-session | Key-value → graph | Preferences, domain knowledge, registries |
| Entity | Cross-session | Entity registry + properties | Stable identity across conversations |
| Temporal KG | Cross-session + history | Graph with validity intervals | Facts that change; time-travel queries |

## Framework Landscape (narrow by dominant retrieval pattern)

| Framework | Architecture | Best for | Trade-off |
|---|---|---|---|
| Mem0 | Vector + graph, pluggable | Fast time-to-prod, multi-tenant | Less multi-agent-specialised |
| Zep/Graphiti | Bi-temporal knowledge graph | Relationship + temporal reasoning | Advanced features cloud-locked |
| Letta | Self-editing tiered memory | Deep introspection, stateful svcs | Overkill for simple cases |
| Cognee | Multi-layer semantic graph (ECL) | Multi-hop reasoning, evolving memory | Heavier ingest |
| LangMem | Memory tools for LangGraph | Teams on LangGraph | Coupled to LangGraph |
| Filesystem | Files + naming conventions | Prototyping | No semantic search/relations |

Compare by *retrieval shape*, not brand, and re-check any benchmark numbers before quoting them —
they date quickly. The stable rule: measure retrieval quality, then add structure.

## Retrieval Strategies

Semantic (embedding similarity) for direct lookups; entity/graph traversal for "everything about
X"; temporal filter for changing facts; hybrid (semantic + keyword + graph) when accuracy matters
and infra budget allows. Load memories just-in-time into attention-favoured positions — don't
preload everything.

## Consolidation & Error Recovery

Consolidate periodically to bound growth; **invalidate, don't discard** (history powers temporal
queries). On retrieval failure: empty → widen search then ask; stale → check `valid_until`,
consolidate; conflict → prefer most recent `valid_from`, surface if low confidence; write failure
→ queue for retry, never block the response. Implementation code: load
`./references/implementation.md` when building stores/consolidation from scratch.

## Examples

```python
# Mem0 — later fact supersedes earlier on retrieval
m.add("User prefers dark mode", user_id="alice")
m.add("User switched to light mode", user_id="alice")
m.search("preferred theme?", user_id="alice")   # -> light mode
```
```python
# Temporal query — where did the user live on a past date?
graph.create_temporal_relationship(user, "LIVES_AT", addr, valid_from=d1, valid_until=d2)
graph.query_at_time({"type": "LIVES_AT", "source_label": "User"}, query_time=march_1)
```

## Guidelines

1. Start filesystem/simple; escalate a layer only when retrieval quality demands it.
2. Track temporal validity for any fact that can change.
3. Pin one embedding model per store; re-embed all entries if it changes.
4. Consolidate on thresholds/schedule; invalidate rather than delete.
5. Always have a fallback when retrieval returns nothing.
6. Respect retention/deletion policy from `${ctx.guardrails}` (privacy of persistent memory).

## Gotchas

1. **Stuffing everything into context**: expensive and degrades attention — retrieve just-in-time.
2. **Ignoring temporal validity**: stale facts poison context; the agent acts on wrong assumptions.
3. **Over-engineering early**: simple filesystem memory can beat specialised tooling; add
   sophistication only when a simpler layer demonstrably fails.
4. **Embedding-model mismatch**: writing with one model and reading with another wrecks recall —
   vector spaces aren't interchangeable. Pin the model; re-embed on change.
5. **Graph schema rigidity**: rigid node/edge types break as the domain evolves; prefer generic
   relations + property bags.
6. **Topic-not-context match**: retrieving "Python (snake)" while discussing "Python (language)".
   Include session/domain metadata and filter on it.

## Integration

- `multi-agent-patterns` — shared memory across coordinated agents.
- `ontology-guided-retrieval` — when the "memory" is a curated grounding graph.
- `self-improving-agent-skills` — persists lessons that improve skills over time.

## References

- `./references/implementation.md` — load when implementing vector/graph/temporal stores or
  consolidation from scratch.
- Zep temporal KG (arXiv:2501.13956); Mem0 (arXiv:2504.19413); Cognee (arXiv:2505.24478) — load
  when evaluating that specific architecture. Re-verify benchmark figures before citing.
- Best practices: https://agentskills.io/skill-creation/best-practices
