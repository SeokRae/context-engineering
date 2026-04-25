---
name: ke
description: >
  7-phase Context Engineering knowledge engine.
  Ingest information into a structured knowledge base or query it to produce results.
  Universal — not tied to any specific domain or output type.
  Keywords: knowledge, context engineering, ingest, query, knowledge base, store, remember, recall
allowed-tools: Read, Write, Bash, Grep, Glob
---

# Knowledge Engine (ke)

A universal 7-phase knowledge engine. It operates in two modes:

- **Ingest**: Accepts any information (text, documents, URLs, notes) and stores it as structured entries in a knowledge base.
- **Query**: Accepts any question or output request and produces a result by drawing on stored knowledge.

Both modes run the same 7-phase engine. Phase behavior varies by mode.

---

## Usage

```
/ke                          — auto-detect mode, full 7-phase engine
/ke "some text or question"  — auto-detect mode from the input
/ke:ingest                   — force ingest mode
/ke:query                    — force query mode
```

Optional argument: `--kb <path>` to override the knowledge base location.

---

## KNOWLEDGE_PATH Resolution

The engine needs a root directory for all knowledge entries and the index. Resolution order:

1. `--kb <path>` argument passed to the skill
2. Environment variable `KNOWLEDGE_PATH`
3. `./knowledge/index.md` in the current working directory
4. `~/knowledge/index.md` as the user default
5. Not found → ask once: "Where should I store knowledge? (default: `~/knowledge/`)"

On first use, if the resolved directory does not exist, create it and write an empty `index.md`.

---

## Phase 0: Mode Detection

Before entering the 7-phase engine, auto-detect the operating mode.

**Ingest signals** (any one is sufficient):
- User provides a document, file path, or URL
- User says "remember this", "store this", "note that", "save this"
- Content is longer than the framing question
- User makes factual statements without asking for anything in return

**Query signals** (any one is sufficient):
- A question mark is present
- User requests output: "write me...", "analyze...", "compare...", "summarize..."
- User references stored knowledge: "based on what I told you...", "what do we know about..."

**Ambiguous**: Ask ONE question — no more:

> "Is this information to store, or a question to answer?
> (1) Store it  (2) Answer it  (3) Both"

Option 3 (Both) runs Ingest phases 1–7 first, then Query phases 1–7 on the same input.

When mode is forced via `:ingest` or `:query`, skip detection entirely.

---

## The 7-Phase Engine

### Phase 1: Problem Definition (문제 정의)

| | Ingest | Query |
|---|---|---|
| **Action** | Classify the input: domain, knowledge type, importance | Rephrase the question precisely; identify relevant domains; determine output format |
| **Output** | `[Domains: X] [Type: fact/decision/constraint/procedure/reference] [Importance: H/M/L]` | `[Query: rephrased text] [Domains: X, Y] [Format: prose/list/table/code/analysis/document]` |
| **Decision** | If domain is genuinely unclear, ask: "Which domain fits best? e.g., finance, health, legal, project-X" | If output format is ambiguous, ask: "What format do you need? e.g., prose summary, comparison table, bullet list" |

Note (Ingest): An entry belongs to one domain; use the most specific one. If cross-domain, choose primary.

Type definitions:
- `fact` — an observable or reported truth
- `decision` — a choice made and its rationale
- `constraint` — a limitation, rule, or boundary condition
- `procedure` — step-by-step instructions for doing something
- `reference` — a pointer to an external source

Importance scale: H = must retain long-term, M = useful context, L = ephemeral or low-signal.

---

### Phase 2: Candidate Collection (후보 수집)

| | Ingest | Query |
|---|---|---|
| **Action** | Read the full input content. If a file path is given, read the file. If a URL, fetch it. If pasted text, use as-is. | Read `index.md`. Find entries whose domain, tags, or summary match the query. Read each candidate entry file. |
| **Output** | Full content loaded (internal — not shown to user unless debugging) | List of candidates with brief relevance notes: `domain/slug.md — relevant because: ...` |
| **Decision** | If input is 10,000+ words: "Store as a single entry or split by section?" | If 0 candidates found: report gracefully — "No stored knowledge matches this query. Proceeding with general knowledge only." |

---

### Phase 3: Selection (선택)

| | Ingest | Query |
|---|---|---|
| **Action** | Filter content. Label each chunk: **Keep** (new, high-value), **Skip** (trivial, redundant), **Merge** (overlaps an existing entry). Check `index.md` for existing entries in the same domain with similar titles or tags. | Rank candidates by relevance, recency, and reliability. Discard entries with no meaningful bearing on the query. |
| **Output** | Internal — not shown unless complex | Ranked list of entries to use (internal) |
| **Decision** | If Merge candidates are found: "Entry `domain/slug.md` already covers this. Update it or keep both?" | If all candidates are low-relevance: note the gap — "Available knowledge is tangential. Answer will rely more on general reasoning." |

Merge heuristic: Two entries merge if they share >70% semantic overlap AND cover the same narrow concept. Related-but-distinct concepts (e.g., "Python async patterns" vs. "asyncio event loops") should remain separate entries.

---

### Phase 4: Structuring (구조화)

| | Ingest | Query |
|---|---|---|
| **Action** | Format each Keep chunk into a standard knowledge entry (see Entry Format below). One entry per concept. Generate a slugified filename. | Organize selected entries and relevant excerpts into a coherent context block for answering. Sequence them logically. |
| **Output** | Structured entry draft (internal) | Internal context assembled |
| **Decision** | None — proceed automatically | None — proceed automatically |

Entry format rules:
- Frontmatter fields: `title`, `domain`, `type`, `source`, `date`, `reliability`, `tags`, `related`
- Body sections: `## Key Facts`, `## Constraints`, `## Decisions`, `## Notes`
- Omit any empty section entirely
- Target length: 20–80 lines including frontmatter

---

### Phase 5: Compression (압축)

| | Ingest | Query |
|---|---|---|
| **Action** | Remove redundant examples. Trim verbose prose to essential facts. Aim for 20–80 lines per entry. | Summarize tangential context to fit the context budget. Keep high-relevance excerpts verbatim. |
| **Output** | Compressed entry draft (internal) | Compressed context (internal) |
| **Decision** | If entry exceeds 80 lines after compression: "This entry is large. Split into `{title-part-a}` and `{title-part-b}`?" If the user declines the split: accept and proceed to Phase 6. Large entries (>100 lines) are harder to retrieve but user choice is final. | None — proceed automatically |

---

### Phase 6: Execution (실행)

| | Ingest | Query |
|---|---|---|
| **Action** | Write each entry file to `{KNOWLEDGE_PATH}/{domain}/{slug}.md`. Update `index.md`: add a new row to the correct domain table (or create the table if the domain is new). | Generate the answer in the requested output format using the assembled context. Honor the format determined in Phase 1. |
| **Output** | `Stored: {domain}/{slug}.md` (one line per entry written) | The answer — prose, list, table, code block, analysis, or full document, as appropriate |
| **Decision** | None | None |

Supported query output formats (non-exhaustive): prose narrative, bullet list, comparison table, code snippet, structured analysis, full document, step-by-step instructions, numbered summary.

---

### Phase 7: Verification (검증)

| | Ingest | Query |
|---|---|---|
| **Action** | Scan existing entries for semantic conflicts with the new entry. Add `related` cross-references where appropriate. Verify `index.md` row count matches files on disk. | Check the answer for unsupported claims, internal contradictions, and gaps relative to stored knowledge. |
| **Output** | Silent on success. Conflict: "Conflict with `domain/existing.md`: [description]. Keep new entry, update existing, or discard?" | Confidence footer appended to answer: `[Confidence: H/M/L] — based on N entries. Gaps: {gap description or "none"}` |
| **Decision** | On conflict: ask user to resolve (keep new / update existing / discard) | None — gaps are reported in the confidence footer, not as a blocking question |

Semantic conflict = new entry contradicts or supersedes a stored fact (e.g., different values for the same factual claim). Minor thematic overlap does NOT trigger a conflict.

Confidence levels:
- High (H): 3+ consistent recent entries; answer is well-grounded
- Medium (M): 1–2 entries or dated entries; note gaps, answer is reasonable
- Low (L): 0 relevant entries or contradictory entries; answer relies on general knowledge — always append a warning

---

## Phase Skipping

For simple inputs, multiple phases execute in a single step without visible output:

| Scenario | Behavior |
|----------|----------|
| Short simple ingest (1–2 sentences, clear domain) | Phases 1–5 execute silently; user sees only Phase 6 confirmation |
| Query with 1–2 matching index entries | Phases 3 and 5 execute silently |
| Large document ingest (1000+ words) | All phases shown with progress markers |
| Multi-domain or complex query | All phases shown with progress markers |

Rule: show phase transitions when the input is complex. For simple inputs, phases execute silently and the user sees only the final output.

---

## Quality Checkpoints

There are exactly **two** checkpoints. No other gates.

**Checkpoint A — After Phase 3 (Ingest only)**

If ALL content is classified as Skip or Merge with no new material:

> "Everything in this input is already stored. Options: (1) Skip — nothing to add. (2) Update existing entries with today's date to mark as re-verified."

**Checkpoint B — After Phase 7 (both modes)**

- Ingest: surface any unresolved conflicts before closing. If none, complete silently.
- Query: the confidence footer is always shown. If confidence is Low, add: "Consider ingesting more information on this topic before relying on this answer."

The engine flows without interruption except at these two points and the single-question decision points within each phase.

---

## Knowledge Entry Format

```markdown
---
title: {descriptive title}
domain: {domain-slug}
type: fact | decision | constraint | procedure | reference
source: user | document:filename | url:https://...
date: {YYYY-MM-DD}
reliability: high | medium | low
  # high — direct user statement, official documentation, code in the repository
  # medium — secondhand source, older but verified document
  # low — inferred behavior, external claim not yet verified
tags: [tag1, tag2]
related: [domain/other-entry]
---

# {Title}

> {one-line summary}

## Key Facts
## Constraints
## Decisions
## Notes
```

Storage path: `{KNOWLEDGE_PATH}/{domain}/{slugified-title}.md`

Granularity: one entry per **concept** — not per document, session, or conversation turn. If one document contains three distinct concepts, create three entries.

Empty sections are omitted entirely from the written file.

---

## Index Format

```markdown
# Knowledge Index

> Last updated: YYYY-MM-DD | Total entries: N | Domains: domain-a, domain-b

## {domain}

| Entry | Type | Date | Reliability | Tags | Summary |
|-------|------|------|-------------|------|---------|
| [title](domain/slug.md) | fact | YYYY-MM-DD | high | tag1, tag2 | one-line summary |
```

Index update rules:
- Add a new row when an entry is created
- Update the row when an entry is modified
- Remove the row when an entry is deleted
- Regenerate the header counts (`Total entries`, `Domains`) on every update
- Sort entries within a domain by date descending (newest first)

---

## References

- [Entry Template](references/entry-template.md)
- [Index Template](references/index-template.md)
