# Context Source Strategy

> Classification of the 4 context sourcing strategies used by the pipeline.

## Strategy Classification

| Strategy | Definition | Pipeline Mapping | Example |
|----------|------------|-----------------|---------|
| **RAG** (Retrieval-Augmented Generation) | Searches stored knowledge for query-relevant information and injects it | Phase 2 KB search (keyword matching → tag expansion → related entry tracking) | Search "transaction isolation" in KB index.md → load 3 related entries |
| **Memory** (Persistent Knowledge) | Structured knowledge store that persists across sessions | Entire KB system (Format B storage + `--consolidate` maintenance) | Store fact in `domain/transaction-isolation.md` → search it in the next session |
| **Tool Result** (Dynamic Injection) | Calls tools at execution time to dynamically collect the latest state | Phase 2 dynamic context injection (`git log`, `find`, `head`) | Collect recent changes with `git log --oneline -5` → basis for Keep/Skip decisions |
| **System Prompt** (Instruction Placement) | Places assembled context in a structured container that the model reads as authoritative instructions; existing system prompts (CLAUDE.md, rules/) are also read as sources in Phase 2 | Phase 6 output (Format A execution instruction, Format C CLAUDE.md) + Phase 2 existing prompt reading | Format A: assemble into Role+Task+Constraints+Output Format structure → inject into model; Format C: generate CLAUDE.md → auto-injected every subsequent session |

## Mapping to Phase 2 Source Decisions

Each row in the Phase 2 "automatic source determination" table maps to one of the strategies above:

| Phase 1 Signal | Collection Source | Strategy |
|---------------|-------------------|----------|
| Question / task request | Scan KB `index.md` → read matching entries | **RAG** |
| Project / codebase mention | `git log`, `find`, build files | **Tool Result** |
| Document / file / URL provided | Read full file / fetch URL | **RAG** (one-time) |
| "저장해줘" (save it) / user data input | Full input text | **Memory** (write) |
| Existing CLAUDE.md / rules/ present | Read existing system prompt files | **System Prompt** (read) |

> **The authoritative source decision logic is in `gather/SKILL.md` Phase 2.** This file defines the classification taxonomy only.

> **Dual role of System Prompt**: The other three strategies handle only the *input* side of context, but System Prompt spans both input (reading existing CLAUDE.md/rules/) and output (generating a new system prompt in Phase 6). It functions as a source in Phase 2 and as the final delivery format in Phase 6.
