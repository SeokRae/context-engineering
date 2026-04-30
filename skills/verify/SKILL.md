---
name: verify
description: >
  Phase 7: Final holistic verification. Automatically invoked on G6 failure, or run manually.
  Checks for omissions, conflicts, speculation, and consistency, then reports confidence. Artifacts are not modified — only the verification log is recorded.
  Keywords: verification, holistic check, confidence, feedback loop, omission, conflict, speculation
allowed-tools: Read, Write, Bash, Grep, Glob
---

# Verify (Phase 7)

Final Verification phase. Checks whether the compose output actually satisfies the Phase 1 criteria. Does not modify the output — only reports verification results. Records the verification log (`_verify-log.md`).

---

## Prerequisites

Confirm what to verify:

```
compose output (execution instruction / KB entry / project artifacts)
Phase 1 analysis results (purpose / constraints / success criteria)
```

If no output exists:
> "Please run `/context-engineering:compose` first."
Stop.

---

## Trigger

- **Automatic**: compose auto-invokes when G6 is not met
- **Manual**: run `/context-engineering:verify` directly (re-verify existing artifacts)

---

## Phase 7. Final Verification

**Purpose**: Check the output against 4 criteria and report confidence.

### 4 Checks

| Check | Criteria |
|-------|----------|
| **Omission** | Are all Phase 1 purpose/constraints reflected in the output? |
| **Conflict** | Are there no contradictions within the output? |
| **Speculation** | Are there no claims unsupported by collected sources? Review `[source: general]` tagged items first. |
| **Consistency** | Does the output format match the Phase 1 success criteria? |

### Check Output Format

```
[Phase 7 Verification Result]

Omission: {N items reflected / Unreflected: {item name} — {description}}
Conflict: {none / {itemA} ↔ {itemB} — {description}}
Speculation: {none / {item name} — no source evidence}
Consistency: {match / {mismatch detail} — Phase 1 success criteria: {original text}}

[Confidence: H/M/L]
{confidence description}
```

### Confidence Criteria

| Level | Condition | Meaning |
|-------|-----------|---------|
| **H** (High) | 3+ sources, no conflicts, no speculation | Output is grounded in sufficient evidence |
| **M** (Medium) | 1-2 sources or minor gaps present | Reasonable but can be improved with additional information |
| **L** (Low) | Insufficient sources or conflicts present | Relies on general knowledge — review required |

M level: "Gathering more information can improve accuracy."
L level: "⚠ Sources are insufficient. Re-run gather or provide additional information."

---

## Feedback Loop

Guide the return to the appropriate Phase based on the issue type:

| Issue Type | Return Phase | Command |
|------------|--------------|---------|
| Omission — purpose/constraints not reflected | Phase 1-2 | `/context-engineering:gather` |
| Conflict — contradiction between Phase 3 selected items | Phase 3 | `/context-engineering:gather` (re-select) |
| Conflict — contradiction during Phase 4 structuring | Phase 4 | `/context-engineering:build` re-run |
| Speculation — insufficient source evidence | Phase 2 | `/context-engineering:gather` (add sources) |
| Consistency — output format mismatch | Phase 6 | `/context-engineering:compose` re-run |

### Automatic Recovery Support

Once the return Phase is determined in the feedback loop, propose automatic execution to the user:

```
[Phase 7 Feedback]
Issue: {issue type} — {detailed description}
Return target: Phase {N}
Recommended action: {specific correction}

Run `{command}` automatically? (Y/n)
```

- If the user responds "Y", "네" (yes), "진행" (proceed), or no response → automatically run the command
- If the user responds "n", "아니오" → provide the command only and stop
- When running automatically, **pass the issue context along**:
  - Failed check items
  - Specific unmet content
  - What was lacking in the previous attempt

> Returning without context risks the target Phase repeating the same mistake.

When all criteria are met:
```
Verification complete. [Confidence: H] The output is ready to use.
```

---

## G7 Gate

AI auto-evaluates the following criteria:

| Criteria | Evaluation |
|----------|------------|
| Confidence determined | Is one of H/M/L clearly assigned? |
| Feedback Loop complete | Was return guidance given per issue type, or was "all criteria met" concluded? |
| Self-scoring rubric complete | Was a rubric score (0-100) produced? |
| Log recorded | Was the result recorded in `_verify-log.md`? |

**G7 Pass**: Verification complete — output is ready to use. Delete `_phase1-result.md` if it exists.

**G7 Failure**: Return to the appropriate Phase per feedback loop
```
Phase 7 result: {verification summary}
Unmet items: {check item} — {reason}
Recommended action: {specific correction direction}
Run `{command}` automatically? (Y/n)
```

---

## Saving Verification Results

On each verification run, append the result to `_verify-log.md`:

    ### {YYYY-MM-DD HH:mm} — {output format: A/B/C}

    | Check | Result | Detail |
    |-------|--------|--------|
    | Omission | Pass/Fail | {content} |
    | Conflict | Pass/Fail | {content} |
    | Speculation | Pass/Fail | {content} |
    | Consistency | Pass/Fail | {content} |

    Confidence: {H/M/L}
    Action: {none / feedback loop → Phase N}

### Comparing Previous Results

If `_verify-log.md` already exists:
1. Read the previous verification results
2. Compare Pass/Fail changes for the same check items
3. If a repeated failure pattern is found, state it explicitly:

    [Repeated Failure Pattern]
    {check item} has failed {N} consecutive times — estimated cause: {cause}
    Recommended action: {specific Phase return guidance}

> Related: [Circuit Breaker Protocol](../context-engineering/references/failure-cases.md) — present user options when the same gate fails ~3 times consecutively.

---

## Self-scoring Rubric

After verification, calculate the overall score using the rubric below:

| Criteria | Points | Score |
|----------|--------|-------|
| Phase 1 purpose coverage | 30 | {0-30} |
| Source evidence sufficiency | 25 | {0-25} |
| Internal consistency | 25 | {0-25} |
| Output format appropriateness | 20 | {0-20} |
| **Total** | **100** | **{total}** |

Score interpretation: 80+ = H, 50-79 = M, 49- = L

---

## Memory Consolidation (Manual Run)

Running `/context-engineering:verify --consolidate` manually inspects the KB:

1. **Duplicate detection**: Entries within the same domain with 70%+ overlapping `tags` → Suggest merge
2. **Staleness detection**: Entries with `date` older than 90 days → Suggest re-verification or archive
3. **Orphan detection**: Entries not referenced in any other entry's `related` → Suggest linking

Report results to the user only. Actual deletion/modification proceeds after user approval.
