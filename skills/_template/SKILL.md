---
name: skill-name-in-kebab-case
description: >
  Third-person, present-tense description written for discovery. State what the skill
  DOES, the concrete triggers that should activate it (keywords, task types), and where
  its boundary ends. This text is injected into the system prompt and decides activation.
license: MIT
metadata:
  author: Social Housing AI
  version: "0.1.0"
  last_updated: 2026-07-02
  layer: "<architecture.md layer, e.g. Context Assembly>"
  priority: "V1 | FF | Later"
---

# Skill Name

## Overview

Two to four sentences: what this skill does and the outcome it produces. Assume a capable
model — only add what it does not already know.

## When to Activate

Activate when:
- Direct trigger (specific task type or keyword)
- Indirect signal (broader pattern indicating relevance)

**Do not activate** (adjacent skills own this):
- `sibling-skill-a` — owns <nearby job>
- `sibling-skill-b` — owns <nearby job>

## Core Concepts

Only the mental models the agent needs and doesn't already have. Challenge every line for
its token cost.

## Workflow / Detailed Topics

Ordered, executable steps or decision tables. State the freedom level (high / medium / low)
and match specificity to fragility.

1. Step one
2. Step two

For long detail, link out: see [reference](./references/topic.md).

## Practical Guidance

Patterns, anti-patterns, and decision frameworks for applying the skill.

## Examples

**Example:**
```
Input:  <describe input>
Output: <expected output>
```

## Guidelines

1. Verifiable rule one
2. Verifiable rule two

## Gotchas

1. **Title**: what goes wrong and how to prevent it.
2. **Title**: another failure mode and the fix.

## Integration

- `related-skill` — how they hand off (plain text, not a link).

## References

- [Own reference](./references/topic.md)
- Related skill: `skill-name`
- External: spec / docs / paper
