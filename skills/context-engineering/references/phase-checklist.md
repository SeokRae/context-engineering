# Context Engineering 7-Phase Checklist

Copy and use at the start of a project.

---

## Project Info

- **Project**: 
- **Date**: 
- **Purpose**: 

---

## gather (Phase 1-3)

### Phase 1. Problem Definition

- [ ] Question 1 complete: purpose (problem to solve)
- [ ] Question 2 complete: constraints (required background information)
- [ ] Question 3 complete: success criteria (form of the output)
- [ ] Scope Reduction complete: Session Scope Declaration block created (minimum problem / Must-have / Nice-to-have / Excluded)
- [ ] Role auto-inferred
- [ ] G1 passed: purpose / constraints / success criteria / scope all clear

### Phase 2. Context Candidate Collection

- [ ] Sources determined based on Phase 1 signals
- [ ] KB existence checked (skip KB search if not present)
- [ ] Candidate collection from determined sources complete
- [ ] G2 passed: all required sources collected, no obvious Omissions

### Phase 3. Context Selection

- [ ] Relevance / Recency / Reliability criteria applied
- [ ] Keep / Skip / Merge classification complete
- [ ] G3 passed: criteria applied consistently, Skip reasons clear

---

## build (Phase 4-5)

### Phase 4. Context Structuring

- [ ] `_phase1-result.md` existence confirmed (restore from file if not in conversation context)
- [ ] Key Facts section written
- [ ] Constraints section written
- [ ] Decisions section written
- [ ] Notes section written (if applicable)
- [ ] `[source: ...]` tag attached to all items
- [ ] Empty sections removed
- [ ] G4 passed: no ambiguous items, no cross-section duplicates, source tags attached

### Phase 5. Context Compression

- [ ] Duplicate items removed
- [ ] Tangential information replaced with one-line summaries
- [ ] Constraints / Decisions original text preserved
- [ ] G5 passed: Constraints / Decisions preserved + budget met + over-compression avoided

---

## compose (Phase 6)

### Phase 6. Execution Instruction Generation

- [ ] `_phase1-result.md` existence confirmed (restore from file if not in conversation context)
- [ ] Phase 1 success criteria signal re-confirmed
- [ ] Output format determined (instruction / KB entry / project Artifacts)
- [ ] Output format auto-determined and user confirmation complete
- [ ] Output generated
- [ ] G6 passed: success criteria matched, no Omissions / Conflicts / Speculation

**Output format**: ☐ Instruction  ☐ KB entry  ☐ Project Artifacts

---

## verify (Phase 7) — on G6 failure or manual run

### Phase 7. Final Verification

- [ ] Omission check: Phase 1 purpose/constraints reflected
- [ ] Conflict check: no contradictions in output
- [ ] Speculation check: no unsupported claims
- [ ] Consistency check: output format matches success criteria
- [ ] Confidence rating: H / M / L
- [ ] Return to relevant Phase if feedback loop needed
- [ ] Auto-run suggestion complete when return needed (with problem context)

---

## Self-Check Criteria

| Phase | Key Criteria |
|-------|-------------|
| G1 | All three of purpose / constraints / success criteria clear + Session Scope Declaration block created |
| G2 | All required sources collected, no obvious Omissions |
| G3 | Relevance / Recency / Reliability applied consistently, Skip reasons clear |
| G4 | No ambiguous items, no cross-section duplicates, source tags attached |
| G5 | Constraints / Decisions preserved, budget met, over-compression avoided |
| G6 | success criteria matched, no Omissions / Conflicts / Speculation |
| G7 | Confidence H/M/L rated, feedback loop complete |
