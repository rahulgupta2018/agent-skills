---
name: agent-discovery
description: >
  Analyse an existing agent, agent prompt, or AGENT.md to inventory responsibilities, tool usage, 
  reusable procedures, context-heavy sections, failure points, and candidate skill extraction opportunities.
  Use when refactoring or decomposing an existing agent into smaller, more focused agents.
license : Apache-2.0
compatibility: >
  Requires a readable agent definition or prompt. Works best when tool lists, 
  non-functional requirements, and sample tasks are available.
metadata:
  skill-type: analysis
  skill-category: analysis
  skill-subcategory: discovery
  skill-version: 1.0.0
allowed-tools: Read Write
---

## Purpose 
Identify what an agent is doing, how it is doing it, and what skills are being used. This skill is useful for understanding existing agents, identifying areas for improvement, and extracting reusable skills for new agents.

## Workflow
### Step 1: Parse the current agent definition or prompt
Extract: 
  - Purpose
  - Responsibilities
  - Inputs/ Outputs
  - Tool usage
  - Reusable procedures
  - Context-heavy sections
  - Failure handling
  - Candidate skill extraction opportunities
  - formatting rules
  - domain rules 

### Step 2: Categorize content
Classify each instruction into one of:
  - Identify and categorize the extracted content into relevant sections
  - orchestration logic
  - reusable procedures
  - context-heavy sections
  - domain policy/ reference material
  - formatting-only guidance 
  - redundant/ obsolete content

### Step 3: Identify skill candidates
Create a skill candidate if ALL apply:
  - The procedure is reusable across tasks or agents
  - The procedure is multi-step or specialised
  - keeping it always in promt would waste context
  - the capability has a crisp purpose and trigger

### Step 4: Identify anti-patterns
Flag:
  - Monolithic or overly complex prompts that could be broken down into smaller, more focused agents
  - duplicated instructions or procedures that could be refactored into reusable skills
  - mixed policy and execution logic that could be separated for clarity
  - unclear routing 
  - hidden assumptions about tools / files / data
  - missing validation criteria

## Output Format 
### Current Capabilities 
### Candidate skills
### Keep in Orchestrator
### Remove / Simplify 
### Rinks

## Guardrails
- Do not create skills for trivial or one-step behaviors.
- Do not assume external tools exist unless explicitly stated in the agent definition or prompt.
- Distinguish domain knowledge from procedure. 
- Ensure all extracted skills have clear triggers and purposes. 
- Avoid duplicating existing skills. 
- Maintain separation of concerns between orchestration and execution logic.





  

