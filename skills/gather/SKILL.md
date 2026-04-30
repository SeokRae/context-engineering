---
name: gather
description: >
  Phase 1-3: Define the problem, collect context candidates, and select only what is needed.
  The first stage of the 7-Phase Context Engineering pipeline.
  Keywords: problem definition, context collection, selection, relevance, recency, reliability
allowed-tools: Read, Write, Bash, Grep, Glob, Agent
---

# Gather (Phase 1-3)

The starting point of context engineering. Clarify user intent, collect candidates from relevant sources, and select only what is needed.

---

## Phase 1. Problem Definition

**Purpose**: Clarify what the user wants from the AI. This becomes the baseline for all subsequent Phases.

### Query Approach

Ask in **purpose → constraints → success criteria** order, one at a time, using multiple-choice format when possible.

**Question 1** (purpose — confirming the Task):
> "What is the problem you are trying to solve right now?"

**Question 2** (constraints — confirming Phase 2 sources + Constraints):
> "What background information would the AI need to do this task well? (e.g., existing code, documentation, prior decisions)"

**Question 3** (success criteria — confirming Output Format):
> "What form should the output take? (e.g., execution instruction, saved note, project documentation)"

**Additional question**: Permitted only once if ambiguity remains unresolved after the 3 questions. No more than 4 questions total.

### Scope Reduction

A processing step automatically performed by the AI after collecting answers to the 3 questions (not an additional question):

**A. Problem scope reduction**
- Analyze the purpose answer to identify the "minimum problem to solve in this session"
- Already small enough → auto-pass
- Can be decomposed → present 2-3 options, let the user choose

**B. Source scope declaration**
- Classify expected sources from the constraints answer:
  - **Must-have**: Sources directly mentioned or clearly needed — collect first in Phase 2
  - **Nice-to-have**: Sources that may indirectly help — collect after Must-have if budget allows

**C. Output Session Scope declaration block**
```
--- Session Scope ---
Minimum problem: {reduced purpose}
Must-have sources: {list}
Nice-to-have sources: {list}
Excluded: {explicitly out of scope}
```

### Role Auto-Inference

Automatically infer the role the AI should take from the three answers. Do not ask the user directly about the role.

### G1 Gate

The AI automatically evaluates the following criteria:

| Criterion | Evaluation |
|-----------|-----------|
| purpose clear | Is the problem to be solved specifically defined? |
| constraints clear | Is the scope of background information needed understood? |
| success criteria clear | Is the form of the output confirmed? |
| Role inferable | Can the AI's role be inferred from the three answers? |
| scope reduction complete | Has the Session Scope declaration block been generated? |

**Auto-pass**: Criteria clearly met → proceed to Phase 2

**User confirmation** (when ambiguous or unmet):
```
Phase 1 result: {summary}
  Unmet items: {item name} — {reason}
  Please revise or approve to continue.
```

---

### Phase 1 Result Preservation Against Compaction

After G1 passes, record in `_phase1-result.md` to prevent loss during context compaction:

    --- Phase 1 Result ---
    purpose: {confirmed purpose}
    constraints: {confirmed constraints}
    success criteria: {confirmed success criteria}
    Role: {inferred role}
    scope-declaration: {session scope declaration block}

At the start of Phase 2, if Phase 1 results are absent from the conversation context, restore from `_phase1-result.md`.
Delete after pipeline completion (G6 pass or G7 pass).

---

## Phase 2. Context Candidate Collection

**Purpose**: Determine sources based on Phase 1 answers and collect candidate context.

### Automatic Source Determination

Collect Must-have sources from Phase 1 Session Scope declaration first, then add Nice-to-have sources when budget allows.

Determine sources from the constraints answer signals in Phase 1:

| Phase 1 Signal | Collection Source |
|---------------|-----------------|
| Document / file / URL provided | Read full text of that file / fetch URL |
| Project / codebase mentioned | Explore codebase, existing spec, CLAUDE.md |
| "저장해줘" (save it) / user data input | Full text of the input |
| Question / task request | Scan KB `index.md` → read matching entries |
| Composite (multiple signals mixed) | Combine above sources — run sub-agents in parallel |

> Strategy classification (RAG / Memory / Tool Result / System Prompt) → [Context Source Strategy](../context-engineering/references/context-source-strategy.md)

### KB Search Enhancement

**Check KB existence**:
1. Search for the `index.md` file path (use the configured KB path if set, otherwise scan common locations)
2. If `index.md` does not exist:
   - **Skip all 4 KB search steps below**
   - Notify: "KB is not configured. Proceeding without KB." then continue collection using only other sources from the Phase 1 signal table
   - Reference: [Failure Cases — Scenario 7](../context-engineering/references/failure-cases.md)
3. If `index.md` exists → execute the 4 KB search steps below:

When finding candidates in KB `index.md`, search in the following order:

1. **Keyword matching**: Scan the index.md table using core words from Phase 1 purpose
2. **Tag expansion**: Read the `tags` field of matched entries and add other entries with the same tags as candidates
3. **Related entry tracking**: Follow the `related` field of matched entries to add linked entries as candidates
4. **Reliability priority**: When multiple entries cover the same topic, prioritize `reliability: high`

If search results exceed 5, filter in Phase 3.

### Dynamic Context Injection

For project/codebase sources, dynamically check the latest state at collection time:

- Recent changes: `git log --oneline -5`
- Project structure: `find . -name "*.md" -maxdepth 2 -not -path "./.git/*"`
- Build file status: `head -20 package.json` or `head -20 build.gradle` (if the file exists)

Include results in candidate context to use as a basis for Phase 3 Keep/Skip decisions.

### Collection Precautions

- If a source exceeds 10,000 words, check whether to split by section
- If 0 candidates found in KB, notify immediately: "No relevant entries found in stored knowledge. Proceeding with general knowledge."
- For large sources, use progressive loading: scan table of contents/headers first → selectively collect only purpose-relevant sections
- Record the size (word count) of each collected source — used for build Phase 5 budget estimation

### G2 Gate

The AI automatically evaluates the following criteria:

| Criterion | Evaluation |
|-----------|-----------|
| Source completeness | Have all sources needed to achieve Phase 1 purpose been collected? |
| No obvious omissions | Are all materials mentioned in the constraints answer included? |

**Auto-pass**: Criteria clearly met → proceed to Phase 3

**User confirmation** (when omissions are suspected):
```
Phase 2 result: {summary}
  Collected sources: {source list}
  Unmet items: {item} — Is this source also needed?
  Please revise or approve to continue.
```

---

## Phase 3. Context Selection

**Purpose**: Retain only what is actually needed for Phase 1 purpose from the collected candidates.

### Evaluate Against 3 Criteria

| Criterion | Description |
|-----------|------------|
| **Relevance** | Is it directly connected to Phase 1 purpose? |
| **Recency** | If more recent information exists, replace older content |
| **Reliability/Confidence** | Quality of the source (direct statement > official docs > indirect reference) |

### Classification

Classify each candidate item:

- **Keep**: Meets Relevance, Recency, and Reliability criteria → pass to next stage
- **Skip**: Does not meet criteria — specify reason
- **Merge**: 70%+ semantic overlap with an existing Keep item → consolidate into the higher-confidence item

### G3 Gate

The AI automatically evaluates the following criteria:

| Criterion | Evaluation |
|-----------|-----------|
| Criteria applied consistently | Were Relevance, Recency, and Reliability applied equally to all items? |
| Skip reasons clear | Does each skipped item have a stated reason? |
| Keep items sufficient | Does Keep contain the information needed to achieve Phase 1 purpose? |

**Auto-pass**: Criteria clearly met → pass to build

**User confirmation** (when ambiguous or unmet):
```
Phase 3 result: {summary}
  Keep: {N}  Skip: {N}  Merge: {N}
  Unmet items: {item name} — {reason}
  Please revise or approve to continue.
```

---

## After Completion

With gather artifacts (Phase 1 analysis + selected context set), proceed to the next stage:

```
/context-engineering:build   — Phase 4-5 structuring and compression
```
