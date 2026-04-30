---
name: context-engineering
description: >
  7-Phase Context Engineering pipeline — systematically builds context to send to AI.
  Problem Definition → Candidate Collection → Selection → Structuring → Compression → Execution Instruction Generation → Verification.
  Keywords: context engineering, ai context, prompt, knowledge base, CLAUDE.md, spec, 7-phase
allowed-tools: Read, Write, Edit, Bash, Grep, Glob, Agent
---

# Context Engineering

A 7-Phase pipeline for systematically building context to send to AI. Handles knowledge accumulation, prompt assembly, and project document generation in a single flow.

---

## Overall Flow

```
[input]
  ↓
Phase 1 →[G1]→ Phase 2 →[G2]→ Phase 3 →[G3]
                                          ↓
                         Phase 4 →[G4]→ Phase 5 →[G5]
                                                   ↓
                                         Phase 6 →[G6]→ [output]
                                                          ↓ (G6 fail)
                                                        Phase 7
```

---

## Gate Behavior

After each Phase completes, the AI automatically evaluates the criteria:

```
[Auto-pass]      Criteria clearly met → proceed to next Phase without notifying the user

[User confirmation] Ambiguous or criteria unmet →
  "Phase N result: {summary}
   Unmet items: {content}
   Please revise or approve to continue."
```

---

## Sub-skill Entry Points

Each phase can be run individually:

| Command | Responsible Phase | Purpose |
|---------|-------------------|---------|
| `/context-engineering:gather` | Phase 1-3 | Problem Definition → Candidate Collection → Selection |
| `/context-engineering:build` | Phase 4-5 | Structuring → Compression |
| `/context-engineering:compose` | Phase 6 | Execution Instruction Generation |
| `/context-engineering:verify` | Phase 7 | Final Verification (re-runnable) |

**Mid-entry condition**: Only allowed when the previous sub-skill's artifacts exist.
If artifacts are missing, the user is guided to run the prerequisite sub-skill first.

---

## Full Execution (`/context-engineering`)

Calls sub-skills in order:

### Step 1: Run gather
`/context-engineering:gather` — Phase 1-3 + G1-G3

Completion criteria:
- Phase 1 analysis result (purpose / constraints / success criteria / Role / scope-declaration)
- Phase 3 selection result (list of Keep items)

### Step 2: Run build
`/context-engineering:build` — Phase 4-5 + G4-G5

Completion criteria:
- Structured and compressed context block (Key Facts / Constraints / Decisions / Notes (if applicable))

### Step 3: Run compose
`/context-engineering:compose` — Phase 6 + G6

Completion criteria:
- G6 pass: final output (instruction / KB entry / project artifacts)
- G6 fail: automatically move to Phase 7

### Step 4: Run verify (when G6 fails)
`/context-engineering:verify` — Phase 7

Completion criteria:
- Confidence indicator (H/M/L)
- Feedback Loop guidance by issue type
- Self-scoring rubric score (0-100)
- Results recorded in `_verify-log.md`

---

## Output Types

Automatically determined based on Phase 1 success criteria:

| Purpose | Output |
|---------|--------|
| Execution instruction | Role + Task + Constraints + Output Format |
| Knowledge storage | KB entry + index.md update |
| Project development | CLAUDE.md + spec.md + implementation plan |

> Automatically determined in Phase 6, then confirmed by the user. See compose SKILL.md for details.

---

> Pipeline failure scenarios + recovery protocols → [Failure Cases & Recovery](references/failure-cases.md)

> Track overall Phase progress → [Phase Checklist](references/phase-checklist.md)

## Skill Modification Guidelines

1. **Phase consistency**: Maintain the gather → build → compose → verify flow
2. **Gate format**: Follow `"Phase N result: {summary}\n Unmet items: {content}\n Please revise or approve to continue."` format
3. **Template sync**: Always keep `references/` templates and SKILL.md inline structures in sync
4. **Compaction Survival**: Place each SKILL.md's core behavioral contract within the first 5,000 tokens. Put supplementary information after — it may be truncated during context compaction without affecting core behavior.

## Verification

After modifying a skill, run `/context-engineering @<path>` in a real project to confirm Phase 1 queries start correctly.
