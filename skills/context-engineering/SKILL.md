---
name: context-engineering
description: >
  4-phase Context Engineering workflow for developing complex services with AI tools.
  Defines what to know, how to behave, and what to build at each stage:
  domain knowledge collection → Policy (CLAUDE.md) → implementation Spec → step-by-step implementation.
  Use for new projects, large feature development, or building services with external API integrations.
  Keywords: context engineering, AI development, domain knowledge, CLAUDE.md, service development,
  knowledge base, policy, spec, implementation, new project, service dev
allowed-tools: Read, Write, Bash, Grep, Glob, Agent
---

# context-engineering

A 4-phase workflow for developing complex services with AI tools, defining **what to know, how to behave, and what to build** at each stage.

```
Step 0: Context Gathering  ←  requirements exploration + parameter collection
  ↓
  ├─ Requirements clear  ─────────────────────────────────────────────┐
  │                                                                   │
  └─ Requirements unclear (discovery mode) ─ empty drafts ───────────┤
                                                                      ↓
┌───────────────────────────────────────────────────────────────────────┐
│  Phase 1: Knowledge Base  (discovery: empty draft, fill while coding) │
│    ↓                                                                  │
│  Phase 2: Policy (CLAUDE.md)  (discovery: minimal version)           │
│    ↓                                                                  │
│  Phase 3: Spec  →  [Readiness Gate]                                  │
│    ↑______________________________↓ re-cycle if unmet                │
└───────────────────────────────────────────────────────────────────────┘
  ↓  [Readiness Gate passed — spiral exit]
Phase 4: Implementation (main track)
```

## Usage

```
/context-engineering @<code-path> [--output <output-path>] [--specs <spec-doc-path>]
```

**Examples:**
```
/context-engineering @/path/to/project
/context-engineering @/path/to/project --specs docs/references/
/context-engineering @/path/to/project --output /tmp/context-docs --specs docs/references/
```

---

## Step 0: Context Gathering

At skill start, perform these two steps in order: ① requirements exploration conversation, ② technical parameter collection.

### Step 0-1. Requirements Exploration (HARD-GATE)

**Ask one question at a time.** Wait for the user's answer before moving to the next.

Confirm at least 3 things:

1. **What are you building?**
   > "Describe what you're building in one sentence."

2. **Why build it? (purpose/motivation)**
   > "Why does this need to be built? What problem does it solve or what goal does it achieve?"

3. **Are there known constraints?**
   > "Are there any known constraints? (fixed tech stack, deadlines, integrated systems, performance requirements, etc.)"

After all three answers, output a summary and ask for confirmation:

```
[Requirements Summary]
- REQ-1 What: {summary}
- REQ-2 Why: {summary}
- REQ-3 Constraints: {summary or "none"}
{If additional requirements emerge from conversation, add REQ-4, REQ-5...}

Does this look correct? Confirmed — will collect technical parameters and start Phase 1.
```

> **HARD-GATE**: After confirmation, choose one of two paths:
> - **Requirements clear** → follow normal flow: write Phase 1~3 thoroughly, then enter Phase 4.
> - **Requirements unclear** → enter discovery mode. Generate Phase 1~3 as empty drafts and proceed directly to Phase 4. Fill in via feedback loop as you develop.

### Step 0-2. Technical Parameter Collection

After requirements confirmation, if `@<code-path>` is provided, auto-detect from build files.

| Parameter | Description | Auto-detect |
|-----------|-------------|-------------|
| `{PROJECT_NAME}` | Project name | — (user input or directory name) |
| `{CODE_PATH}` | Absolute path to code repo | `@` argument |
| `{TECH_STACK}` | Tech stack | `build.gradle` / `pom.xml` / `package.json` / `pyproject.toml` / `Cargo.toml` / `go.mod` |
| `{BUILD_CMD}` | Build command | inferred from build files |
| `{TEST_CMD}` | Test command | inferred from build files |
| `{SPEC_PATHS}` | External spec document paths | `--specs` argument |
| `{OUTPUT_PATH}` | Output artifact path | `--output` if specified, otherwise `{CODE_PATH}/docs/` |

After collection, output final summary and ask for user confirmation.

---

## Discovery Mode (when requirements are unclear)

Enter when Step 0 HARD-GATE determines requirements are unclear.

**Full flow**:

```
Phase 1 empty draft → Phase 2 minimal → Phase 3 empty draft → enter Phase 4 immediately
       ↑____________________↑_________________↑  fill via feedback loop
                                                          ↓
                                    [user requests Readiness Gate when ready]
                                                          ↓
                                            switch to main-track implementation
```

**Per-phase discovery artifacts**:

| Phase | Initial state | Fill while coding |
|-------|--------------|-------------------|
| Phase 1 | Glossary/constraint table with empty rows + `> 🔍 Discovering` | discovered domain terms, new constraints |
| Phase 2 | Project name + build commands only + `> 🔍 Discovering` | non-obvious gotchas, coding conventions |
| Phase 3 | SPEC title only + `> 🔍 Discovering` | architecture decisions |

**Readiness Gate transition**: User directly requests Readiness Gate when they feel sufficiently explored. If gate passes, switch to main-track implementation.

**Feedback loop**: Findings during Phase 4 are immediately reflected in the relevant phase document based on their nature (see Phase 4 feedback loop table).

---

## Phase 1: Knowledge Base

**Purpose**: Build a knowledge base for AI to understand this project's domain.

### Step 1-1. Context Assessment (Scenario Detection)

Analyze the project state and select one of 4 scenarios:

| Scenario | Condition | Description |
|----------|-----------|-------------|
| **A. Greenfield** | no spec + no code | brand new project |
| **B. Spec-first** | spec exists + no code | docs exist, not yet implemented |
| **C. Code-first** | no spec + code exists | existing code, no docs |
| **D. Full context** | spec exists + code exists | both available |

Detection criteria:
- Presence of `{SPEC_PATHS}` → spec documents exist
- Source files in `{CODE_PATH}` → code exists

**Mixed state handling**: For intermediate states ('spec is outdated', 'code only partially documented'), select the closest scenario and note uncertainties in the rationale. Wrong scenario selection affects the entire downstream flow — user confirmation is required.

Output detection result and get user confirmation before proceeding to Source Scan:

```
[Context Assessment] Scenario {A/B/C/D} ({name}) — {one-line rationale}
Uncertainties: {list if any, omit if none}

Is this assessment correct? Confirmed — will start Source Scan.
```

### Step 1-2. Source Scan (conditional)

Collect knowledge differently depending on scenario:

**Scenario A (Greenfield)**:
- Extract domain concepts, constraints, purpose from Step 0 conversation
- Write draft based on conversation without external references
- Mark artifact as "Draft (verification needed)"

**Scenario B (Spec-first)**:
- Explore `{SPEC_PATHS}` documents in parallel with Explore sub-agent
- Collect protocol specs, API guides, vendor docs
- Compress each document to a one-line summary

**Scenario C (Code-first)**:
- Scan `{CODE_PATH}` codebase
- Package structure, key class/module patterns
- Existing design patterns, existing test strategy

**Scenario D (Full context)**:
- Run Scenario B + C in parallel (spec exploration and code scan simultaneously)

### Step 1-3. Knowledge Base Assembly

Always execute regardless of scenario. Organize collected information into the following structure.

**Domain Glossary**:

```markdown
| Term | Definition | Source |
|------|-----------|--------|
| {term} | {one-line definition} | {document name or "conversation"} |
```

**Constraint Registry** (same 5-column format as `knowledge-base-template.md`):

```markdown
| ID | Category | Description | Impact | Priority |
|----|----------|-------------|--------|----------|
| TC-1 | Technical | {constraint description} | {impact} | High |
| BL-1 | Business | {rule description} | {impact} | Medium |
| OC-1 | Organizational | {schedule/access limit} | {impact} | Low |
```

Categories: `TC` technical constraint, `BL` business rule, `OC` organizational constraint (schedule/external dependencies)

**Reference Document Manifest** (omit for Scenario A):

```markdown
| Path | Type | Summary | Updated |
|------|------|---------|---------|
| {path} | spec/code/api | {one line} | {date} |
```

**Output**: `{OUTPUT_PATH}/knowledge-base.md`
- Scenario A: add `> ⚠️ Draft — based on Step 0 conversation, verify during implementation` at top
- Discovery mode: create tables with empty rows + `> 🔍 Discovering — fill in as you develop.`

> **Usage path**: This file is linked in Phase 2 CLAUDE.md references section. Without the link, AI won't find this file in future sessions.

> **Gate**: Normal mode — after saving, run self-assessment and output:
>
> ```
> [Phase 1 Self-Assessment]
> | Item | Status | Action |
> |------|--------|--------|
> | Term sufficiency (5+) | OK / {N terms, key ones missing} | — / supplement Source Scan |
> | Constraint categories (2+) | OK / {uncategorized} | — / expand constraint registry |
> | Source attribution | OK / {items without source} | — / add sources |
>
> Phase 1 complete — `knowledge-base.md` draft saved. Review and proceed to Phase 2?
> ```
>
> Discovery mode — skip self-assessment. Proceed to Phase 2 without confirmation.

---

## Phase 2: Policy (CLAUDE.md)

**Purpose**: Define policy for how AI should behave in this project.

### Step 2-1. CLAUDE.md Level Decision

| Level | Location | Line limit | Purpose |
|-------|----------|-----------|---------|
| Project | `{CODE_PATH}/CLAUDE.md` | 100~150 lines | project-wide common |
| Module | `{CODE_PATH}/{module}/CLAUDE.md` | 60~100 lines | module-specific |

### Step 2-2. CLAUDE.md Writing

WHAT/WHY/HOW 3-section structure:

> **Architecture section guide**: Complex domain services → Hexagonal + DDD (see `references/architecture-principles.md`). Simple scripts/pipelines → describe module structure freely.

```markdown
# {PROJECT_NAME}

> {one-line description}

## Architecture

{3~5 core patterns — include WHY / simple projects: module structure only}

## Build & Test

\`\`\`bash
{BUILD_CMD}
{TEST_CMD}
\`\`\`

## Ports / Endpoints

| Port | Purpose |
|------|---------|

## Core Constraints

{Phase 1 constraints that directly affect development}

## Non-obvious Gotchas

- {gotcha 1}
- {gotcha 2}

## References

- [Knowledge Base]({OUTPUT_PATH}/knowledge-base.md): Phase 1 domain knowledge base
```

> **Required**: Always include `knowledge-base.md` link in the References section. Without this link, AI won't find the KB in future sessions.

Items exceeding 150 lines → split to separate document and link.
Discovery mode: fill project name, tech stack, build commands only + `> 🔍 Discovering`. Add the rest while developing.

**Output**: `{CODE_PATH}/CLAUDE.md` (+ per-module CLAUDE.md)

> **Draft nature**: Phase 2 CLAUDE.md is a draft. Non-obvious Gotchas and coding conventions can only be known after implementation — update in Phase 4 feedback loop. Fill only what's knowable now.

> **Gate**: Normal mode — after saving, run self-assessment and output:
>
> ```
> [Phase 2 Self-Assessment]
> | Item | Status | Action |
> |------|--------|--------|
> | Build command verified | OK / {unverified} | — / re-check build files |
> | High constraints reflected | OK / {missing constraints} | — / add to Core Constraints |
> | KB link exists | OK / missing | — / add to References section |
>
> Phase 2 complete — `CLAUDE.md` draft saved. Review and proceed to Phase 3?
> ```
>
> Discovery mode — skip self-assessment. Proceed to Phase 3 without confirmation.

---

## Phase 3: SPEC

**Purpose**: Document architecture decisions for how to build it.

### Step 3-1. SPEC (Technical Specification)

```markdown
# SPEC — {PROJECT_NAME}

## Architecture Decisions
| Decision | Alternatives | Choice | Rationale |
|----------|-------------|--------|-----------|

## Package Structure
Dependency direction: `entry → application → domain ← infrastructure` (reverse forbidden)
→ [references/architecture-principles.md](references/architecture-principles.md)

## Implementation Plan
| # | Item | REQ | Depends | Done Criteria |
|---|------|-----|---------|--------------|
| 1 | {unit} | REQ-1 | — | {verifiable criterion} |

## Requirements Traceability
| REQ-ID | Requirement | PRD Feature | Implementation Item | Status |
|--------|-------------|-------------|---------------------|--------|
| REQ-1 | {what} | {feature} | Plan #1 | not implemented |
| REQ-2 | {why} | {feature} | Plan #N | not implemented |
| REQ-3 | {constraint} | {non-func/feature} | Plan #N | not implemented |
```

Discovery mode: generate SPEC as empty draft + `> 🔍 Discovering`. Requirements traceability also `> 🔍 Discovering — fill in as requirements are confirmed.` Fill in while developing.

**Output**: `{OUTPUT_PATH}/spec.md`

### Step 3-2. Readiness Gate (decide whether to iterate)

**HARD-GATE (user confirmation required)**: AI summarizes the current state of each criterion. User makes the pass/fail decision. AI must not declare "passed" unilaterally.

Run Phase 3 self-assessment first, then output the gate:

```
[Phase 3 Self-Assessment]
| Item | Status | Action |
|------|--------|--------|
| Decision rationale completeness | OK / {N decisions without rationale} | — / improve those items |
| Implementation order dependencies | OK / {items missing dependency} | — / update plan |
| REQ coverage | OK / {unmapped REQ-IDs} | — / update traceability table |

[Readiness Gate]
① Do all SPEC architecture decisions include alternatives and rationale?
   → {current state: N decisions, M with rationale / missing: ...}
② Are there no unresolved domain terms?
   → {current state: N unresolved — list names}
③ Does CLAUDE.md reflect actual constraints?
   → {current state: N Phase 1 constraints, M reflected in CLAUDE.md}
④ Do all PRD features have verifiable success criteria?
   → {current state: N features, M with success criteria}
⑤ Can the first implementation plan item start immediately?
   → {current state: ready / blockers: ...}
⑥ Are all REQ-IDs mapped to implementation plan items?
   → {current state: N REQs, M mapped / unmapped: REQ-X, REQ-Y}

If any criterion is unmet, specify which phase to return to.
If all pass, say "Proceed to Phase 4".
```

| Criterion | Phase to return to if unmet |
|-----------|----------------------------|
| ① SPEC architecture decisions lack rationale | Phase 3 |
| ② Unresolved domain terms | Phase 1 |
| ③ CLAUDE.md missing constraints | Phase 2 |

If any is unmet, return to the corresponding phase, reinforce, then re-cycle back to Phase 3.

In discovery mode, this gate becomes the **spiral exit checkpoint**. The user decides when to check — when they feel sufficiently explored, they request the Readiness Gate. AI presents the current state; user makes the final transition decision.

---

## Phase 4: Implementation

**Purpose**: Implement based on the plan (or exploration) and verify at each step.

### Per Implementation Unit

**Entry criteria**:
- Normal mode: SPEC confirmed
- Discovery mode: SPEC empty draft generated (entry allowed even if empty)

**Done criteria**: all checklist items must pass before moving to the next unit

- [ ] `{BUILD_CMD}` succeeds
- [ ] `{TEST_CMD}` passes (new + no regression)
- [ ] SPEC architecture decisions followed — skip in discovery mode if SPEC is empty
- [ ] Findings reflected in the relevant phase document
- [ ] Requirements traceability table updated — mark completed REQ-IDs as "implemented"

### Feedback Loop — return to the relevant phase when findings arise

When implementation reveals mismatches, update the appropriate phase document based on the nature of the finding.

| Finding | Update target |
|---------|--------------|
| Domain concept error, term mismatch, new constraint | Phase 1 `knowledge-base.md` |
| AI behavior rule change needed, new gotcha | Phase 2 `CLAUDE.md` |
| Architecture decision reversal, package structure change | Phase 3 `spec.md` |

Update procedure:
1. Record the finding and reason in one line
2. Update the item in the relevant phase document
3. Continue implementation after updating

> **Principle**: Don't fit implementation to documents — update documents to match reality. Findings can be reflected in any earlier phase.

### Post-Phase 4: Implementation Retrospective

After all implementation units pass their done criteria, output the following retrospective.

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

---

## References

- [4-Phase Checklist](references/phase-checklist.md)
- [Knowledge Base Template](references/knowledge-base-template.md)
- [CLAUDE.md Policy Template](references/claude-md-policy-template.md)
- [SPEC Template](references/spec-template.md)
- [Architecture Principles](references/architecture-principles.md)
