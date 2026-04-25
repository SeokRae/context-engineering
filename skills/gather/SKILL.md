---
name: gather
description: >
  context-engineering Step 0 + Phase 1 (Knowledge Base) + Phase 2 (CLAUDE.md).
  Use to start a new project or rework/update existing KB and CLAUDE.md.
  Keywords: context gathering, knowledge base, CLAUDE.md, requirements exploration, domain knowledge, gather
allowed-tools: Read, Write, Bash, Grep, Glob, Agent
---

# context-engineering:gather

Run context gathering (Step 0) → Knowledge Base (Phase 1) → CLAUDE.md (Phase 2).

## Usage

```
/context-engineering:gather [@<code-path>] [--output <output-path>]
```

## Context Resolution

Resolve paths in this order:

1. `@<code-path>` provided → `{CODE_PATH}` = that path
2. Not provided → search current directory for `docs/knowledge-base.md`
3. Not found → ask "Please provide the project path (e.g. `@/path/to/project`)"

`{OUTPUT_PATH}` = `--output` value if specified, otherwise `{CODE_PATH}/docs/`

## Existing Artifacts Check

If `{OUTPUT_PATH}/knowledge-base.md` or `{CODE_PATH}/CLAUDE.md` exist:

```
[Current State]
- knowledge-base.md: {N terms, N constraints if exists / "not found"}
- CLAUDE.md: {exists / not found}

Rework (start over) or update (keep existing, modify parts)?
```

- **Rework** → start from Step 0
- **Update** → load files and proceed only with the parts to modify

## Step 0: Context Gathering

Ask one question at a time. Wait for the user's answer before the next question.

1. "Describe what you're building in one sentence."
2. "Why does this need to be built? What problem does it solve or what goal does it achieve?"
3. "Are there any known constraints? (fixed tech stack, deadlines, integrated systems, performance requirements, etc.)"

After all three answers, output a summary and ask for confirmation:

```
[Requirements Summary]
- REQ-1 What: {summary}
- REQ-2 Why: {summary}
- REQ-3 Constraints: {summary or "none"}
{If additional requirements emerge from the conversation, add REQ-4, REQ-5...}

Does this look correct?
```

If `@<code-path>` is provided, auto-detect tech stack from build files:
`build.gradle` / `pom.xml` / `package.json` / `pyproject.toml` / `Cargo.toml` / `go.mod`

## Phase 1: Knowledge Base

### Scenario Detection

| Scenario | Condition | Description |
|----------|-----------|-------------|
| A. Greenfield | no spec + no code | brand new project |
| B. Spec-first | spec exists + no code | documents only |
| C. Code-first | no spec + code exists | existing code, no docs |
| D. Full context | spec exists + code exists | both available |

For mixed states (outdated spec, partial docs, etc.), select the closest scenario, note uncertainties, and ask for confirmation:

```
[Context Assessment] Scenario {A/B/C/D} ({name}) — {one-line rationale}
Uncertainties: {list if any}

Is this assessment correct?
```

### Knowledge Base Writing

Collect domain terms, constraints, and reference documents to write `{OUTPUT_PATH}/knowledge-base.md`.

**Domain Glossary**:
```
| Term | Definition | Source |
|------|-----------|--------|
| {term} | {one-line definition} | {document name or "conversation"} |
```

**Constraint Registry**:
```
| ID | Category | Description | Impact | Priority |
|----|----------|-------------|--------|----------|
| TC-1 | Technical | {constraint} | {impact} | High |
| BL-1 | Business | {rule} | {impact} | Medium |
| OC-1 | Organizational | {schedule/access limit} | {impact} | Low |
```

Categories: `TC` technical constraint, `BL` business rule, `OC` organizational constraint

**Output**: `{OUTPUT_PATH}/knowledge-base.md`

> **Gate**: Normal mode — after saving, run self-assessment and output in this format:
>
> ```
> [Phase 1 Self-Assessment]
> | Item | Status | Action |
> |------|--------|--------|
> | Term sufficiency (5+) | OK / {N terms, key ones missing} | — / supplement Source Scan |
> | Constraint categories (2+) | OK / {uncategorized} | — / expand constraint registry |
> | Source attribution | OK / {items without source} | — / add sources |
>
> Phase 1 complete — `knowledge-base.md` saved. Review and proceed to Phase 2?
> ```
>
> Discovery mode — skip self-assessment. Proceed to Phase 2 without confirmation.

## Phase 2: CLAUDE.md

> **Architecture section**: Complex domain services → Hexagonal + DDD. Simple scripts/pipelines → describe module structure only.

```markdown
# {PROJECT_NAME}

This file provides guidance to Claude Code when working with code in this repository.

> {one-line description}

## Architecture

{3~5 core patterns — include WHY / simple projects: module structure only}

## Build & Test

\`\`\`bash
{BUILD_CMD}
{TEST_CMD}
\`\`\`

## Core Constraints

{TC/BL items from Phase 1 that directly affect development — max 5}

## Non-obvious Gotchas

{To be discovered during implementation — update in Phase 4 feedback loop}

## References

- [Knowledge Base]({OUTPUT_PATH}/knowledge-base.md): Phase 1 domain knowledge base
```

> **Required**: Always include the `knowledge-base.md` link in the References section. Without this link, AI won't find the KB in future sessions.
> **Draft nature**: Non-obvious Gotchas can only be known after implementation — update in Phase 4.

**Output**: `{CODE_PATH}/CLAUDE.md`

> **Completion**: Normal mode — after saving, run self-assessment and output in this format:
>
> ```
> [Phase 2 Self-Assessment]
> | Item | Status | Action |
> |------|--------|--------|
> | Build command verified | OK / {unverified} | — / re-check build files |
> | High constraints reflected | OK / {missing constraints} | — / add to Core Constraints |
> | KB link exists | OK / missing | — / add to References section |
>
> gather complete — use `/context-engineering:spec` to write PRD·SPEC or `/context-engineering:valid` to check the Readiness Gate.
> ```
>
> Discovery mode — skip self-assessment. Output completion message only.
