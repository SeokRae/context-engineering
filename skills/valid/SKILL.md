---
name: valid
description: >
  context-engineering Readiness Gate check.
  Use after Phase 3, before Phase 4, or to exit discovery mode.
  AI summarizes current state only — user decides whether to pass.
  Keywords: readiness gate, validation, valid, gate, spiral exit, checkpoint
allowed-tools: Read, Bash, Glob
---

# context-engineering:valid

Inspect current project artifacts and present Readiness Gate results to the user.

## Usage

```
/context-engineering:valid [@<code-path>] [--output <output-path>]
```

## Context Resolution

Resolve paths in this order:

1. `@<code-path>` provided → `{CODE_PATH}` = that path
2. Not provided → search current directory for `docs/prd.md` or `docs/spec.md`
3. Not found → ask "Please provide the project path (e.g. `@/path/to/project`)"

`{OUTPUT_PATH}` = `--output` value if specified, otherwise `{CODE_PATH}/docs/`

## Artifact Scan

Read the following files to assess current state:
- `{OUTPUT_PATH}/knowledge-base.md` — domain terms and constraints
- `{CODE_PATH}/CLAUDE.md` — policy
- `{OUTPUT_PATH}/prd.md` — feature requirements and success criteria
- `{OUTPUT_PATH}/spec.md` — architecture decisions and implementation plan

## Readiness Gate Output

**HARD-GATE (user confirmation required)**: AI presents current state summary only. User decides whether to pass. AI must not declare "passed" unilaterally.

```
[Readiness Gate]
① Do all PRD features have verifiable success criteria?
   → {N features, M with criteria / missing: ...}
② Do all SPEC architecture decisions include alternatives and rationale?
   → {N decisions, M with rationale / missing: ...}
③ Are there no unresolved domain terms?
   → {N unresolved — list names / "none"}
④ Does CLAUDE.md reflect actual constraints?
   → {N Phase 1 constraints, M reflected in CLAUDE.md}
⑤ Can the first implementation plan item start immediately?
   → {first item name + dependencies}
⑥ Are all REQ-IDs mapped to implementation plan items?
   → {N REQs, M mapped / unmapped: REQ-X, REQ-Y}

If any criterion is unmet, use the corresponding command below.
If all pass, say "Proceed to Phase 4" or "/context-engineering:impl".
```

| Criterion | Unmet action |
|-----------|-------------|
| ① PRD success criteria missing | `/context-engineering:spec` to improve PRD |
| ② SPEC rationale missing | `/context-engineering:spec` to improve SPEC |
| ③ Unresolved domain terms | `/context-engineering:gather` to improve KB |
| ④ CLAUDE.md constraints missing | `/context-engineering:gather` to update CLAUDE.md |
| ⑤ Implementation plan not ready | `/context-engineering:spec` to improve SPEC |
| ⑥ REQ unmapped | `/context-engineering:spec` to update requirements traceability |
