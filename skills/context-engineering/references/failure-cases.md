# Failure Cases & Recovery

> Pipeline-level failure scenarios and response protocols. The thresholds below are recommended guidelines, not hard rules.

## 1. Context Window Overflow

**Symptom**: Target budget exceeded even after Phase 5 compression
**Cause**: Too many sources collected in Phase 2, or too many Keep items in Phase 3
**Response**:
1. Return to Phase 3 — raise Keep criteria from "relevant" to "essential"
2. Strengthen Phase 5 overflow strategy: remove the entire Notes section → replace Key Facts with one-line summaries
3. If still over budget: convert Keep items with the lowest Reliability to Skip, starting from the lowest

## 2. Conflict Between Keep Items

**Symptom**: Keep items A and B contradict each other during Phase 4 structuring
**Cause**: Information from different points in time or different sources both retained as Keep
**Response**:
1. Compare the source and date of both items
2. Prioritize the item with the most recent date and highest Reliability
3. If equal, ask the user once: "{itemA} vs {itemB} — which one is currently valid?"
4. Record the resolution in the Decisions section: "Conflict resolved: {A} selected, {B} discarded, reason: {reason}"

## 3. Pipeline Abandonment Criteria

Consider stopping when the following signals accumulate:

| Signal | Recommended Threshold | Response |
|--------|-----------------------|----------|
| Phase 1 purpose revised repeatedly | ~3 times | "It is recommended to break the problem into smaller parts. Please pick just one core focus." |
| Phase 7 consecutive L grades | ~2 times | "It is recommended to add more sources or narrow the problem scope before retrying." |
| Accumulated Gate failures | ~5 times | "Working directly may be more efficient. Would you like to abort the pipeline?" |

## 4. Cascading Gate Failures (Circuit Breaker)

**Symptom**: The same gate fails repeatedly → Feedback Loop → fails again
**Response**:
- ~2 consecutive failures at the same gate: compare failure reasons and report the pattern
- ~3 consecutive failures at the same gate:
  ```
  Phase {N} gate has failed {N} times consecutively.
  Estimated cause: {pattern}
  Options: (1) Manually approve this gate and continue  (2) Return to previous Phase  (3) Abort pipeline
  ```

> Related: verify Phase 7 [Repeated Failure Pattern Detection](../../verify/SKILL.md) — detects and reports consecutive Fail patterns in `_verify-log.md`.

## 5. Speculation Infiltration Prevention

**Problem**: Unfounded claims caught only at G6 / Phase 7
**Possible responses within the current pipeline (Phase 3 + Phase 7)**:
- During Phase 3 selection: apply Reliability criteria strictly — items with unclear sources should be Skip rather than Keep
- Phase 7 verification: detect claims without source evidence in the "Speculation" check → Feedback Loop to re-collect in Phase 2

> **Note**: `[source: ...]` inline tags have been added to the Phase 4 structuring step. Each item is tagged with its source to serve as evidence for Phase 7 Speculation verification. See the Phase 4 structuring rules in `skills/build/SKILL.md` and the Speculation check criteria in `skills/verify/SKILL.md` for details.

## 6. Mid-Session Interruption Recovery

**Symptom**: Session ends while pipeline is running
**Cause**: User interruption, connection drop, timeout
**Response**:
1. Check whether `_phase1-result.md` exists
2. Check whether `_context-session.md` exists
3. Resume from the sub-skill corresponding to the last completed Phase's artifacts
4. If no artifacts exist at all, restart from `/context-engineering:gather`

## 7. Complete Absence of Sources

**Symptom**: No sources available to collect in Phase 2
**Cause**: New project, insufficient documentation, general question
**Response**:
1. Inform the user: "No relevant items were found in stored knowledge. Proceeding with general knowledge."
2. In Phase 3, if Keep items total 0, determine whether it is possible to proceed using only the Phase 1 purpose
3. If proceeding is not possible: "Sources are insufficient. Please provide documents or URLs that can be referenced."

> Related: [gather Phase 2 KB search enhancement](../../gather/SKILL.md) — guard condition that skips the search step when no KB exists.

## 8. Compaction During Pipeline Execution

**Symptom**: The model's Context window is compressed while a Phase is in progress
**Cause**: Conversation grows long enough to trigger automatic compaction
**Response**:
1. Restore Phase 1 results from `_phase1-result.md`
2. Verify whether the current Phase's artifacts remain in the conversation
3. If artifacts are lost, re-evaluate starting from that Phase's Gate
4. Core behavioral contracts in each SKILL.md are placed within the first 5,000 tokens, so behavioral rules are preserved

---

## Normal vs Failure Quick Reference

| Metric | Normal Range | Failure Signal |
|--------|-------------|----------------|
| Phase 3 Skip ratio | 30–70% | 90%+ (source quality issue) or 0% (no filtering) |
| Phase 5 compression ratio | Within target range | More than 2× over target |
| Phase 7 Reliability | H or M | L (insufficient sources or Conflict) |
| Accumulated Gate failures | 0–2 | ~5+ (pipeline not suitable) |
