---
name: ke:query
description: >
  Direct entry to Knowledge Engine query mode. Answer questions or produce output using stored knowledge.
  Use when you have a question and want to query the knowledge base directly.
  Keywords: query, ask, recall, retrieve, answer, knowledge base, what do we know
allowed-tools: Read, Bash, Grep, Glob
---

# Knowledge Engine Query Mode (ke:query)

Answer questions or produce output directly by querying a stored knowledge base.
No mode detection. No ingest. Read-only: this skill never writes files.

---

## Usage

```
/ke:query "What is the project architecture?"
/ke:query "Summarize the deployment process as a checklist"
/ke:query --kb ~/project-kb "How do we handle authentication?"
```

Optional: `--kb <path>` to override the knowledge base location.

---

## KNOWLEDGE_PATH Resolution

Locate the knowledge base root directory in this order:

1. `--kb <path>` argument
2. Environment variable `KNOWLEDGE_PATH`
3. `./knowledge/index.md` in the current working directory
4. `~/knowledge/index.md`
5. Not found → inform user: *"No knowledge base found. Use /ke:ingest to store knowledge first. I can answer from general knowledge if needed."*

If the KB path exists but is empty or index.md is missing, proceed to Phase 2 with 0 candidates. Do NOT create or initialize anything.

---

## The Query Flow: 7 Phases

### Phase 1: Problem Definition

Rephrase the question precisely. Identify relevant domains. Determine output format.

**Output**: `[Query: rephrased text] [Domains: X, Y] [Format: prose/list/table/code/analysis/document]`

**Decision**: If output format is ambiguous, ask ONE clarifying question only:
> "What format works best? e.g., prose summary, comparison table, bullet list, code example, step-by-step instructions"

---

### Phase 2: Candidate Collection

Read `index.md`. Find entries whose domain, tags, or summary match the query. Read each candidate entry file.

**Output**: List of candidates with brief relevance notes:
```
domain/slug.md — relevant because: [reason]
domain/slug.md — relevant because: [reason]
```

**Decision**: If 0 candidates found, report:
> "No stored knowledge matches this query. Proceeding with general knowledge only."

---

### Phase 3: Ranking & Selection

**Simple query**: Single-domain query returning ≤2 matching candidates. **Complex query**: Multi-domain OR >2 candidates.

Rank candidates by relevance, recency (prefer newer), and reliability (prefer higher). Keep top candidates.

For simple queries with 1–2 matching entries: execute silently, skip to Phase 4.

---

### Phase 4: Context Assembly

Organize selected entries and relevant excerpts into a coherent context block for answering.
Sequence them logically.

---

### Phase 5: Compression

Summarize tangential context to fit the context budget. Prioritize directly relevant information.

For simple queries: execute silently.

---

### Phase 6: Generation

Generate the answer in the requested output format (determined in Phase 1).

Supported formats (non-exhaustive): prose narrative, bullet list, comparison table, code snippet, structured analysis, full document, step-by-step instructions, numbered summary.

---

### Phase 7: Verification

Check the answer for unsupported claims, internal contradictions, and gaps relative to stored knowledge.

**Confidence levels and footer format**:

- **H (High)**: 3+ consistent recent entries; answer is well-grounded
  - Append footer only: `[Confidence: H] — based on N entries. Gaps: {gap description or "none"}`

- **M (Medium)**: 1–2 entries or dated entries; gaps noted, answer is reasonable
  - Append footer: `[Confidence: M] — based on N entries. Gaps: {gap description or "none"}`
  - Add note: *"Consider ingesting more recent information to improve accuracy."*

- **L (Low)**: 0 relevant entries or contradictory entries; answer relies on general knowledge — always warn
  - Append footer: `[Confidence: L] — based on N entries. Gaps: {gap description or "none"}`
  - Add warning: *"⚠ Answer relies primarily on general knowledge, not stored entries. Consider ingesting relevant knowledge before relying on this answer."*

Example footer (M confidence):
```
[Confidence: M] — based on 1 entry (older). Gaps: no recent updates on authentication changes

Consider ingesting more recent information to improve accuracy.
```

---

## Behavior Rules

1. **Read-only always**: Never write files, modify index.md, or create directories.
2. **Graceful degradation**: Missing or empty KB → answer from general knowledge, noted in confidence footer.
3. **Phase visibility**: Show phase transitions only for complex multi-domain queries. Simple queries execute Phases 1–5 silently.
4. **No gates**: Flow without interruption except single-question decision points.
5. **Always append footer**: Query mode always includes a confidence footer. If Low, add the standard warning (see Phase 7 Verification).

---
