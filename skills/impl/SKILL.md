---
name: impl
description: >
  context-engineering Phase 4 — step-by-step implementation.
  Iterate through SPEC implementation plan: implement unit → verify → feedback loop.
  Keywords: implementation, Phase 4, build, test, impl
allowed-tools: Read, Write, Edit, Bash, Grep, Glob, Agent
---

# context-engineering:impl

Execute step-by-step implementation based on the SPEC implementation plan.

## Usage

```
/context-engineering:impl [@<code-path>] [--output <output-path>]
```

## Context Resolution

Resolve paths in this order:

1. `@<code-path>` provided → `{CODE_PATH}` = that path
2. Not provided → search current directory for `docs/spec.md`
3. Not found → ask "Please provide the project path (e.g. `@/path/to/project`)"

`{OUTPUT_PATH}` = `--output` value if specified, otherwise `{CODE_PATH}/docs/`

## SPEC Check

If `{OUTPUT_PATH}/spec.md` exists, display the implementation plan and confirm start point:

```
[Implementation Plan]
| # | Item | REQ | Depends | Done Criteria |
...

Which item to start from? (number or "from the beginning")
```

If not found: "Without SPEC, this runs in discovery mode — continue?"

## Per-Unit Checklist

After completing each implementation unit, verify all of the following before moving on:

- [ ] `{BUILD_CMD}` succeeds
- [ ] `{TEST_CMD}` passes (new tests + no regression)
- [ ] SPEC architecture decisions followed (skip if SPEC is empty)
- [ ] PRD feature success criteria verified (separate from build success)
- [ ] Findings reflected in relevant phase documents
- [ ] Requirements traceability table updated — mark completed REQ-IDs as "implemented"

## Feedback Loop

When implementation reveals mismatches with documentation:

| Finding | Update target | Command |
|---------|--------------|---------|
| Domain term error, new constraint | `knowledge-base.md` | `/context-engineering:gather` |
| New gotcha, AI behavior rule change | `CLAUDE.md` | `/context-engineering:gather` |
| Feature requirement change | `prd.md` | `/context-engineering:spec` |
| Architecture decision reversal, package structure change | `spec.md` | `/context-engineering:spec` |

> **Principle**: Don't fit implementation to documents — update documents to match reality.

## Post-Implementation Retrospective

After all units pass their done criteria, output the following.

Discovery mode: use abbreviated "discovery retrospective" (discovered knowledge summary + empty item count + priority improvement areas).

```
[Implementation Retrospective]

## Requirements Achievement
| REQ-ID | Requirement | Achieved | Notes |
|--------|-------------|:--------:|-------|
| REQ-1 | {what} | ✅/❌ | {one line} |
| REQ-2 | {why} | ✅/❌ | {one line} |
| REQ-3 | {constraint} | ✅/❌ | {one line} |

## Process Retrospective
- What worked well: {effective decisions/approaches across Phase 1~4}
- Discovered late: {items fed back to Phase 1/2/3 during implementation}
- Improvements: {workflow points to improve in next project}

## Final Document State
| Document | Last update summary | Feedback loop updates |
|----------|--------------------:|:--------------------:|
| knowledge-base.md | {summary} | {N} |
| CLAUDE.md | {summary} | {N} |
| spec.md | {summary} | {N} |
```
