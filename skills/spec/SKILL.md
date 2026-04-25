---
name: spec
description: >
  context-engineering Phase 3 — PRD (product requirements) + SPEC (technical specification).
  Use to write new or rework/update existing PRD and SPEC.
  Keywords: PRD, SPEC, requirements, technical spec, implementation plan, spec
allowed-tools: Read, Write, Bash, Grep, Glob
---

# context-engineering:spec

Write PRD (product requirements) and SPEC (technical specification).

## Usage

```
/context-engineering:spec [@<code-path>] [--output <output-path>]
```

## Context Resolution

Resolve paths in this order:

1. `@<code-path>` provided → `{CODE_PATH}` = that path
2. Not provided → search current directory for `docs/prd.md` or `docs/spec.md`
3. Not found → ask "Please provide the project path (e.g. `@/path/to/project`)"

`{OUTPUT_PATH}` = `--output` value if specified, otherwise `{CODE_PATH}/docs/`

## Existing Artifacts Check

**Knowledge Base load**: If `{OUTPUT_PATH}/knowledge-base.md` exists, use domain terms and constraints as context.

**PRD/SPEC detection**: If `{OUTPUT_PATH}/prd.md` or `{OUTPUT_PATH}/spec.md` exist:

```
[Current State]
- prd.md: {N features if exists / "not found"}
- spec.md: {N architecture decisions if exists / "not found"}

Rework (start over) or update (keep existing, modify parts)?
```

## Phase 3-1: PRD

```markdown
# PRD — {PROJECT_NAME}

## Purpose
{Problem this product/feature solves — one paragraph}

## Usage Scenarios
{Who uses it, in what situations}

## Feature Requirements
| REQ | Feature | Success Criteria | Priority |
|-----|---------|-----------------|----------|
| REQ-1 | {feature name}: {one-line description} | {verifiable criterion} | High/Medium/Low |

## Constraints
| Constraint | Impact | Priority |
|------------|--------|----------|
| {description} | {impact} | High/Medium/Low |
```

**Output**: `{OUTPUT_PATH}/prd.md`

## Phase 3-2: SPEC

```markdown
# SPEC — {PROJECT_NAME}

## Architecture Decisions
| Decision | Alternatives considered | Choice | Rationale |
|----------|------------------------|--------|-----------|
| {what was decided} | {alt1}, {alt2} | {chosen} | {reason} |

## Package Structure
Dependency direction: `entry → application → domain ← infrastructure` (reverse forbidden)
* Simple projects (CLI, scripts) may use a different module structure appropriate for the project.

```
{layer}: {folder path}  ← {responsibility}
```

## Data Flow
```
{input} → {ComponentA} → {ComponentB} → {output}
```

## Implementation Plan
| # | Item | REQ | Depends | Done Criteria |
|---|------|-----|---------|--------------|
| 1 | {module/feature} | REQ-1 | — | {verifiable criterion} |
| 2 | {module/feature} | REQ-2 | 1 | {verifiable criterion} |

## Requirements Traceability
| REQ-ID | Requirement | PRD Feature | Implementation Item | Status |
|--------|-------------|-------------|---------------------|--------|
| REQ-1 | {what} | {feature} | Plan #1 | not implemented |
| REQ-2 | {why} | {feature} | Plan #N | not implemented |
| REQ-3 | {constraint} | {non-func/feature} | Plan #N | not implemented |
```

Discovery mode: add `> 🔍 Discovering — fill in as requirements are confirmed.` to requirements traceability section.

**Output**: `{OUTPUT_PATH}/spec.md`

> **Completion**: Normal mode — after saving, run self-assessment and output in this format:
>
> ```
> [Phase 3 Self-Assessment]
> | Item | Status | Action |
> |------|--------|--------|
> | Decision rationale completeness | OK / {N decisions without rationale} | — / improve those items |
> | Implementation order dependencies | OK / {items missing dependency info} | — / update plan |
> | REQ coverage | OK / {unmapped REQ-IDs} | — / update traceability table |
>
> `prd.md` and `spec.md` saved. Use `/context-engineering:valid` to check the Readiness Gate or `/context-engineering:impl` to start implementation.
> ```
>
> Discovery mode — skip self-assessment. Output completion message only.
