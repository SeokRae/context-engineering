---
name: build
description: >
  Phase 4-5: Structures and compresses the selected context.
  Receives gather Artifacts and organizes them into a model-readable form, retaining only the essentials.
  Keywords: context structuring, compression, key facts, constraints, decisions
allowed-tools: Read, Write, Bash, Grep, Glob
---

# Build (Phase 4-5)

The stage for Context Structuring and Context Compression. Receives the context selected by gather, organizes it into a model-readable form, and retains only the essentials.

---

## Prerequisites

Before running, verify gather Artifacts:

```
Phase 1 analysis result (purpose / constraints / success criteria / Role / scope-declaration)
Phase 3 selection result (list of Keep items)
```

Verification order:
1. Do the above Artifacts exist in the conversation context? → Use them if present
2. If not, check whether the `_phase1-result.md` file exists → If present, read it to restore the Phase 1 result
3. If the Phase 1 result has been restored but the Phase 3 selection result (Keep items) is missing:
   > "Please run `/context-engineering:gather` first. (Phase 3 selection result is required)"
   Stop.
4. If both are missing:
   > "Please run `/context-engineering:gather` first."
   Stop.

---

## Phase 4. Context Structuring

**Purpose**: Organize the selected context into a section structure that is easy for the model to read.

### Structure

```markdown
## Key Facts
- {key fact} [source: {source name}]

## Constraints
- {constraint} [source: {source name}]

## Decisions
- {decision and rationale} [source: {source name}]

## Notes
- {supplementary information} [source: {source name}]
```

`[source: ...]` tag rules:
- The source name is the identifier of the source collected in Phase 2 (filename, URL, "user", "KB:{entry-title}")
- Items derived from multiple sources: `[source: file.md, user]`
- Items derived from general knowledge: `[source: general]`
- During Phase 7 speculation checks, `[source: general]` items are treated as primary review targets

### Structuring Rules

- Each item expresses a single fact / constraint / decision
- Empty sections are omitted entirely
- No Omission between items — if the same content appears in two sections, consolidate it into the more appropriate section
- Key Facts includes only items directly related to the Phase 1 purpose
- Attach `[source: ...]` tags to all items — they serve as the basis for Phase 7 speculation verification

### G4 Gate

The AI automatically evaluates the following criteria:

| Criterion | Evaluation |
|-----------|------------|
| No ambiguous items | Does each item express a single clear fact / constraint / decision? |
| No Conflict between sections | Does the same content not exist in two sections? |
| Purpose reflected | Do Key Facts connect to the Phase 1 purpose? |
| Source tags attached | Do all items have a `[source: ...]` tag? |

**Auto-pass**: Criteria clearly met → proceed to Phase 5

**User confirmation** (when ambiguous or unmet):
```
Phase 4 result: {summary}
  Unmet items: {item name} — {reason}
  Please revise or approve to continue.
```

---

## Phase 5. Context Compression

**Purpose**: Remove unnecessary parts, retaining only essential facts, Constraints, and Decisions.

### Token Budget Estimation

Measure the context size before compression:

1. **Measure**: Estimate the total word count of Keep items using `wc -w`
2. **Budget decision**:

   | Task scale | Word count basis | Target compression ratio |
   |------------|-----------------|--------------------------|
   | Small (single file / question) | < 500 words | No compression needed |
   | Medium (module / feature unit) | 500–2,000 words | Compress to 50% or less |
   | Large (entire project) | 2,000+ words | Compress to 30% or less |

3. **Strategy when over budget**: Summarize Notes section first → condense Key Facts → condense Constraints last

### Budget Example Scenarios

| Scenario | Input | Target | After compression | Main method |
|----------|-------|--------|-------------------|-------------|
| Single API question | 300 words | — | 300 words | No compression needed |
| Module refactoring | 1,200 words | ≤50% | ~550 words | Summarize Notes + remove duplicates |
| Full project analysis | 4,000 words | ≤30% | ~1,100 words | Remove Notes + condense Key Facts |

### Incremental Loading

When a source is large (2,000+ words):
- Do not read the full text at once
- First scan only the file list / table of contents / headers
- Selectively read only the sections relevant to the Phase 1 purpose
- Record the path and line range of each section read

### Compression Rules

- **Remove duplicates**: If items with the same meaning appear multiple times, merge them into one
- **Summarize Tangential information**: Content indirectly related to the Phase 1 purpose → replace with a one-line summary
- **Preserve essentials**: Constraints / Decisions items retain original text — these are not subject to compression
- **Key Facts priority**: Facts directly related to the purpose retain original text

### G5 Gate

The AI automatically evaluates the following criteria:

| Criterion | Evaluation |
|-----------|------------|
| Constraints preserved | Are all Constraints still present after compression? |
| Decisions preserved | Are Decisions and their rationale retained without loss? |
| Over-compression prevention | Has information needed to achieve the Phase 1 purpose not been removed? |
| Budget met | Is the post-compression word count within the target budget? |

**Auto-pass**: Criteria clearly met → pass to compose

**User confirmation** (when ambiguous or unmet):
```
Phase 5 result: {summary}
  Unmet items: {item name} — {reason}
  Please revise or approve to continue.
```

---

## After Completion

Take the build Artifacts (structured and compressed context block) and proceed to the next stage:

```
/context-engineering:compose   — Run Phase 6 execution instruction generation
```
