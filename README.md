# context-engineering

> English | [한국어](README.ko.md)

A Claude Code plugin that implements a 7-phase pipeline for systematically preparing context before giving it to an AI — what information to include, in what structure, at what level of compression.

## What is Context Engineering?

Context Engineering is **the practice of designing what goes into an LLM's context window**. It goes beyond writing a single good prompt — it decides what information to give to AI, when, and in what structure. RAG, memory architecture, token budget management, and system prompt design all fall under this category.

This plugin addresses **persistent context preparation** — a repeatable workflow to collect, select, structure, compress, and verify context so every AI session starts with exactly the right information.

## Pipeline

```
Phase 1 --[G1]--> Phase 2 --[G2]--> Phase 3 --[G3]-->
Phase 4 --[G4]--> Phase 5 --[G5]--> Phase 6 --[G6]--> [Output]
                                                          |
                                                     (G6 fail)
                                                          v
                                                       Phase 7
```

| Sub-skill | Phases | Role |
|-----------|--------|------|
| `/context-engineering:gather`  | 1-3 | Problem definition → candidate collection → selection |
| `/context-engineering:build`   | 4-5 | Structuring → compression |
| `/context-engineering:compose` | 6   | Output generation |
| `/context-engineering:verify`  | 7   | Final verification |

Each gate auto-passes when criteria are clearly met. When ambiguous, the pipeline pauses and asks for user confirmation before continuing.

## Output Types

Phase 6 auto-determines the output format from Phase 1 success criteria:

| Format | Trigger signals | Output |
|--------|----------------|--------|
| **A — Execution Instruction** | "make a prompt", "how should I write", "give me instructions" | Role + Task + Constraints + Output Format |
| **B — KB Entry** | "save this", "remember this", "take a note" | Markdown entry with frontmatter + Key Facts / Constraints / Decisions / Notes |
| **C — Project Artifacts** | "start a project", "analyze codebase", "I want to build" | CLAUDE.md + spec.md + implementation plan |

If the signal is ambiguous, Phase 6 asks the user to choose A, B, or C once.

## Installation

```bash
# 1. Add to marketplace
claude plugin marketplace add https://github.com/SeokRae/context-engineering.git

# 2. Install plugin
claude plugin install context-engineering
```

Restart Claude Code after installation for changes to take effect.

Verify installation:
```bash
claude plugin list
#   ❯ context-engineering@context-engineering
#     Version: 3.0.0
#     Scope: user
#     Status: ✔ enabled
```

To use directly from a local directory:
```bash
claude --plugin-dir /path/to/context-engineering
```

## Usage

### Full pipeline

Runs all 7 phases in sequence from Phase 1 through Phase 6 (Phase 7 triggered automatically if Phase 6 gate fails).

```
/context-engineering
```

### Sub-skills — direct phase entry

Use when re-working a specific phase, returning via a feedback loop, or when prior phases are already complete.

| Command | Phases | When to use |
|---------|--------|-------------|
| `/context-engineering:gather`  | 1-3 | New problem, or re-defining scope / re-collecting sources |
| `/context-engineering:build`   | 4-5 | Re-structuring or re-compressing existing context |
| `/context-engineering:compose` | 6   | Re-generating output with updated context |
| `/context-engineering:verify`  | 7   | Manual verification pass or `--consolidate` KB maintenance |

```
/context-engineering:gather
/context-engineering:build
/context-engineering:compose
/context-engineering:verify
/context-engineering:verify --consolidate
```

## Reference Documents

Phase artifact templates are in `skills/context-engineering/references/`:

| File | Purpose |
|------|---------|
| [`phase-checklist.md`](skills/context-engineering/references/phase-checklist.md) | Per-project 7-phase progress checklist |
| [`knowledge-base-template.md`](skills/context-engineering/references/knowledge-base-template.md) | Phase 1 KB artifact template |
| [`claude-md-policy-template.md`](skills/context-engineering/references/claude-md-policy-template.md) | Phase 2 CLAUDE.md template |
| [`spec-template.md`](skills/context-engineering/references/spec-template.md) | Phase 3 SPEC template (architecture decisions + package structure) |
| [`architecture-principles.md`](skills/context-engineering/references/architecture-principles.md) | Hexagonal + DDD principles (optional, for complex services) |
| [`entry-template.md`](skills/context-engineering/references/entry-template.md) | KB entry format (frontmatter + body) |
| [`index-template.md`](skills/context-engineering/references/index-template.md) | KB index format |
| [`context-session-template.md`](skills/context-engineering/references/context-session-template.md) | Multi-step task context scratch file (deleted after use) |

## License

MIT
