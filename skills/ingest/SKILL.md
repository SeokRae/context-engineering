---
name: ke:ingest
description: >
  Direct entry to Knowledge Engine ingest mode. Store information into a structured knowledge base.
  Use when you have information to store and don't need mode detection.
  Keywords: ingest, store, remember, save, knowledge base, note
allowed-tools: Read, Write, Bash, Grep, Glob
---

# Knowledge Engine: Ingest Mode (ke:ingest)

Store any information (documents, notes, URLs, facts, decisions) into a structured knowledge base without mode detection. This sub-skill skips query behavior entirely and runs only the ingest-relevant phases of the Knowledge Engine.

---

## Usage

```
/ke:ingest                      — ingest from next message or stdin
/ke:ingest "text or file path"  — ingest specific content
/ke:ingest --kb <path>          — override knowledge base location
```

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

## Ingest Flow (7 Phases)

### Phase 1: Problem Definition

Classify the input:
- **Domain**: The subject area (e.g., project-name, architecture, legal, process)
- **Type**: fact | decision | constraint | procedure | reference
  - `fact` — an observable or reported truth
  - `decision` — a choice made and its rationale
  - `constraint` — a limitation, rule, or boundary condition
  - `procedure` — step-by-step instructions for doing something
  - `reference` — a pointer to an external source
- **Importance**: H (must retain long-term), M (useful context), L (ephemeral)

Output: `[Domain: X] [Type: fact/decision/constraint/procedure/reference] [Importance: H/M/L]`

If domain is genuinely unclear, ask: "Which domain fits best? e.g., {list examples}"

---

### Phase 2: Candidate Collection

Read the full input content:
- If a file path: read the file
- If a URL: fetch the content
- If pasted text: use as-is

Output: Full content loaded (internal — shown to user only if debugging is needed)

If input is 10,000+ words: "Store as a single entry or split by section?"

---

### Phase 3: Selection

Filter content. Label each chunk:
- **Keep**: new, high-value material
- **Skip**: trivial, redundant, or out-of-scope
- **Merge**: overlaps an existing entry

Check `index.md` for existing entries in the same domain with similar titles or tags.

Output: Internal — not shown unless complex

**Quality Checkpoint A**: If ALL content is classified as Skip or Merge with no new material:

> "Everything in this input is already stored. Options:
> (1) Skip — nothing to add
> (2) Update existing entries with today's date to mark as re-verified"

**Merge heuristic**: Two entries merge if they share >70% semantic overlap AND cover the same narrow concept. Related-but-distinct concepts should remain separate.

If Merge candidates are found: "Entry `domain/slug.md` already covers this. Update it or keep both?"

---

### Phase 4: Structuring

Format each Keep chunk into a standard knowledge entry. Generate a slugified filename.

Output: Structured entry draft (internal)

Entry format rules:
- Frontmatter fields: `title`, `domain`, `type`, `source`, `date`, `reliability`, `tags`, `related`
- Body sections: `## Key Facts`, `## Constraints`, `## Decisions`, `## Notes`
- Omit any empty section entirely
- Target length: 20–80 lines including frontmatter

---

### Phase 5: Compression

Remove redundant examples. Trim verbose prose to essential facts. Aim for 20–80 lines per entry.

Output: Compressed entry draft (internal)

If entry exceeds 80 lines after compression: "This entry is large. Split into `{title-part-a}` and `{title-part-b}`?"

If the user declines the split: accept and proceed to Phase 6. Large entries (>100 lines) are harder to retrieve but user choice is final.

---

### Phase 6: Execution

Write each entry file to `{KNOWLEDGE_PATH}/{domain}/{slug}.md`.

Update `index.md`: add a new row to the correct domain table (or create the table if the domain is new).

Output: `Stored: {domain}/{slug}.md` (one line per entry written)

---

### Phase 7: Verification

Scan existing entries for semantic conflicts with the new entry. Add `related` cross-references where appropriate. Verify `index.md` row count matches files on disk.

Output: Silent on success.

**On conflict**: "Conflict with `domain/existing.md`: [description]. Keep new entry, update existing, or discard?"

**Semantic conflict** = new entry contradicts or supersedes a stored fact (e.g., different values for the same factual claim). Minor thematic overlap does NOT trigger a conflict.

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

**Reliability scale**:
- `high` — direct user statement, official documentation, code in the repository
- `medium` — secondhand source, older but verified document
- `low` — inferred behavior, external claim not yet verified

Storage path: `{KNOWLEDGE_PATH}/{domain}/{slugified-title}.md`

**Granularity**: one entry per **concept** — not per document, session, or conversation turn. If one document contains three distinct concepts, create three entries.

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

## Phase Skipping

For simple inputs, multiple phases execute in a single step without visible output:

| Scenario | Behavior |
|----------|----------|
| Short simple ingest (1–2 sentences, clear domain) | Phases 1–5 execute silently; user sees only Phase 6 confirmation |
| Large document ingest (1000+ words) | All phases shown with progress markers |

Rule: show phase transitions when the input is complex. For simple inputs, phases execute silently and the user sees only the final output.
