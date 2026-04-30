---
name: compose
description: >
  Phase 6: Generates the final output matching Phase 1 success criteria.
  Receives the context compressed by build and assembles it into one of: execution instruction / KB entry / project artifacts.
  Keywords: execution instruction, prompt, role task constraints output format, knowledge entry, CLAUDE.md, spec
allowed-tools: Read, Write, Bash, Grep, Glob
---

# Compose (Phase 6)

Final output generation stage. Combines the structured and compressed context from build with Phase 1 success criteria to assemble the appropriate output.

---

## Prerequisites

Before execution, verify build artifacts:

```
Phase 1 analysis results (purpose / constraints / success criteria / Role / scope-declaration)
Structured and compressed context blocks (Key Facts / Constraints / Decisions / Notes)
```

Verification order:
1. Do the above artifacts exist in the conversation context? → Use them if present
2. If not, check whether `_phase1-result.md` exists → Read it to restore Phase 1 results if found
3. If Phase 1 results are restored but build artifacts (structured/compressed blocks) are missing:
   > "Please run `/context-engineering:build` first. (Structured/compressed blocks are required)"
   Stop.
4. If both are missing:
   > "Please run `/context-engineering:gather` first."
   Stop.

---

## Output Budget Guide

Target size for Phase 6 output:

| Output Format | Target Size | If Exceeded |
|--------------|-------------|-------------|
| A. Execution Instruction | 200–800 words | Condense Notes to one-line summaries → if still exceeded, split artifacts |
| B. KB Entry | 20–80 lines | Split the entry (see entry-template guidelines) |
| C. Project Artifacts | CLAUDE.md 100–150 lines | If over 150 lines, extract to a separate docs/ file |

> **Immutable rule**: Constraints and Decisions must not be deleted or condensed even under output budget pressure. These items were already guaranteed to be preserved in build Phase 5.

---

## Phase 6. Execution Instruction Generation

**Purpose**: Automatically determine the output format based on the Phase 1 success criteria signal and generate the final output.

### Automatic Output Format Determination

Read the signal from the Phase 1 success criteria answer and determine the format:

| success criteria signal | Output Format |
|------------------------|---------------|
| "지시해줘" (give me instructions), "프롬프트 만들어줘" (make me a prompt), "어떻게 써야 해" (how should I write it) | **A. Execution Instruction** |
| "저장해줘" (save it), "기억해줘" (remember it), "노트해줘" (note it) | **B. KB Entry** |
| "프로젝트 시작" (start a project), "개발하고 싶어" (I want to develop), "코드베이스 분석" (analyze codebase) | **C. Project Artifacts** |

After automatic determination, confirm exactly once:
> "Output format: **{decided format name}** ({A/B/C}). Enter A/B/C to change."

- If the user responds affirmatively ("yes", "proceed", Enter, etc.) → proceed with the determined format
- If A/B/C is entered → switch to that format

If the signal does not match any row in the table, present choices:
> "Please select output format: (A) Execution Instruction  (B) KB Entry  (C) Project Artifacts"

---

## G6 Gate

After compose completes, the AI automatically evaluates:

| Criteria | Evaluation |
|----------|------------|
| success criteria match | Does the output match the result form defined in Phase 1? |
| No omissions | Are all Phase 1 purpose/constraints reflected in the output? |
| No conflicts | Are there no contradictions within the output? |
| No speculation | Does the output contain no unsupported claims? |

**G6 Pass**: Complete — present the output. Delete `_phase1-result.md` if it exists.

**G6 Fail**: Automatically invoke Phase 7
```
G6 unmet: {criteria} — {reason}
Running verify for detailed validation.
```

---

> Refer to the sections below for the detailed structure of each format.

---

### A. Execution Instruction (System Prompt)

Assemble using Role + Task + Constraints + Output Format structure:

```markdown
## Role
{AI role inferred from Phase 1 answers and context}

## Task
{Phase 1 purpose — describe the problem to be solved concretely}

## Constraints
{All Constraints items from Phase 5}
{Additional constraints derived from Phase 1 constraints}

## Output Format
{Form of the result defined in Phase 1 success criteria}
```

Writing principles:
- Role: One sentence, described as a specific expert role
- Task: Described at an actionable level without ambiguity
- Constraints: List form; each item understandable on its own
- Output Format: Specify format, length, and structure

### Tool Context (Execution Instruction only)

If the Phase 1 purpose requires tool usage, add a `## Tools` section to the instruction:

```markdown
## Tools

| Tool | Purpose | Usage Example |
|------|---------|---------------|
| {tool name} | {purpose for this task} | {call example or pattern} |
```

MCP Tools (if applicable):
- context7: Library documentation lookup — `resolve-library-id` → `get-library-docs`
- playwright: Web page verification — `browser_navigate` → `browser_snapshot`

**Inclusion criteria** (based on Phase 1 purpose signals):
- "code", "implement", "test" signals → include Bash, Read, Write, Agent
- "documentation", "library" signals → include context7 MCP
- "web", "browser", "UI" signals → include playwright MCP
- No tool-usage signal → omit `## Tools` section

### Context Retention Instruction (Execution Instruction only)

If the execution instruction (Format A) directs a task with **3 or more steps**, add the following block at the bottom of the instruction:

    ## Context Checkpoints

    If this task has 3 or more steps:
    1. After each major step completes, record in `_context-session.md`:
       - What has been completed so far
       - Key facts discovered
       - Information needed for the next step
    2. Before starting the next step, re-read `_context-session.md`
    3. After all steps are complete, delete `_context-session.md`

Reference: [Context Session Template](../context-engineering/references/context-session-template.md)

---

### B. KB Entry

Save in standard knowledge entry format:

```markdown
---
title: {descriptive title}
domain: {domain-slug}
type: fact | decision | constraint | procedure | reference
source: user | document:{filename} | url:{https://...}
date: {YYYY-MM-DD}
reliability: high | medium | low
tags: [tag1, tag2]
related: []
---

# {Title}

> {one-line summary}

## Key Facts
{Key Facts items from build artifacts}

## Constraints
{Constraints items from build artifacts}

## Decisions
{Decisions items from build artifacts}

## Notes
{Notes items from build artifacts}
```

Save path: `{KNOWLEDGE_PATH}/{domain}/{slugified-title}.md`

After saving, update `index.md`:
- Add a row to the relevant domain table
- If the domain section does not exist, create a new section in alphabetical order
- Recalculate header counts (Total entries, Domains)
- Maintain descending date sort order

Reference: [Entry Template](../context-engineering/references/entry-template.md) | [Index Template](../context-engineering/references/index-template.md)

---

### C. Project Artifacts

Generate three documents for project development:

**CLAUDE.md** (AI behavior policy — Persistent System Prompt):

```markdown
# {PROJECT_NAME}

This file provides guidance to Claude Code when working with code in this repository.

> {one-line project description}

## Architecture
{Architecture description based on Phase 4 Key Facts + Decisions}
{Layer structure table}

## Build & Test
{Build and test commands}

## Core Constraints
{Phase 4 Constraints items — up to 5, TC/BL prioritized}

## Non-obvious Gotchas
{Easy-to-miss pitfalls from Phase 4 Notes}

## References
- [Knowledge Base]({OUTPUT_PATH}/knowledge-base.md)
```

Reference: [CLAUDE.md Template](../context-engineering/references/claude-md-policy-template.md) | [Knowledge Base Template](../context-engineering/references/knowledge-base-template.md)

**spec.md** (Technical Specification):

```markdown
# {PROJECT_NAME} Spec

## Architecture Decisions
| Decision | Alternatives | Choice | Rationale |

## Package Structure
{Package structure by layer}

## Implementation Plan
| # | Item | REQ | Depends | Done Criteria |

## Requirements Traceability
| REQ-ID | Feature | Implementation | Status |
```

Reference: [Spec Template](../context-engineering/references/spec-template.md)

**Implementation Plan**: Describe the first implementation item at a level where work can begin immediately.
