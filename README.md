![Version](https://img.shields.io/badge/version-3.1.0-blue)
![License](https://img.shields.io/github/license/SeokRae/context-engineering)
![Claude Code Plugin](https://img.shields.io/badge/Claude_Code-Plugin-blueviolet)

# context-engineering

> [한국어](README.ko.md)

A Claude Code plugin that systematically prepares AI context through a 7-phase pipeline — deciding what information to include, in what structure, at what level of compression.

## When / How / How Much

| Question | Answer |
|----------|--------|
| **When** | Information comes from 2+ sources, or a simple copy-paste would cause the AI to miss context |
| **How** | Run `/context-engineering` → answer 3 questions → the 7-phase pipeline handles the rest. Jump directly to a sub-skill if prior phases are already done |
| **How Much** | Compression scales with input size — see the Token Budget table below |

## Decision Table

### Level 1: Do I need Context Engineering?

| Situation | Use CE? | Reason |
|-----------|---------|--------|
| Single question, no extra context needed | No | A plain prompt is enough |
| Editing one file with a clear scope | No | Direct instruction is faster |
| Combining information from 2+ sources | **Yes** | AI will miss context without it |
| Need to reference previous decisions | **Yes** | KB retrieval keeps things consistent |
| Starting a project (CLAUDE.md + spec) | **Yes** | Format C generates all project artifacts |
| Writing the same type of prompt repeatedly | **Yes** | Format A produces a reusable instruction |
| Accumulating domain knowledge | **Yes** | Format B saves a KB entry |

### Level 2: Where do I enter?

| Scenario | Entry point | Prerequisite |
|----------|------------|-------------|
| Starting from scratch | `/context-engineering` | — |
| Re-running gather only | `:gather` | — |
| Phase 1/3 artifacts already exist | `:build` | gather output required |
| Compressed context block exists | `:compose` | build output required |
| Re-validating existing output | `:verify` | compose output required |
| KB housekeeping | `:verify --consolidate` | Maintenance mode (independent of pipeline) |

> Each sub-skill checks for its prerequisite artifacts and exits with guidance if they are missing.

### Level 3: What context to prioritize?

Once you know which sub-skill to enter, decide what context sources take priority for your task type:

| Task type | Priority context | De-prioritize | Output |
|-----------|-----------------|---------------|--------|
| Q&A / factual answer | Retrieval-heavy: KB entries, source evidence, authoritative docs | Conversation history, speculative notes | — (direct answer) |
| Project setup | Architecture principles, CLAUDE.md policy, spec template, implementation plan | Unrelated KB entries | C |
| Knowledge base entry | Consolidation check, duplicate detection, stale entry check | Raw source text (already distilled) | B |
| Execution instruction | Role, task, constraints, output format | Verbose background (compress in Phase 5) | A |

> This table guides Phase 2 source selection and Phase 5 compression priorities. See [context-source-strategy.md](skills/context-engineering/references/context-source-strategy.md) for the underlying RAG / Memory / Tool Result / System Prompt classification.

## Token Budget Allocation

Phase 5 compresses collected context to fit within a target token budget. Compression tier is determined by input size:

| Scale | Word count | Target compression | Strategy |
|-------|-----------|-------------------|----------|
| Small (single file / question) | < 500 words | None needed | Pass through as-is |
| Medium (module / feature) | 500–2,000 words | ≤ 50% | Summarize Notes → remove duplicates |
| Large (full project) | 2,000+ words | ≤ 30% | Remove Notes → condense Key Facts |

**Compression order** (what gets cut first):
1. Notes section — summarized or removed
2. Key Facts — condensed where purpose allows
3. Constraints / Decisions — **never compressed** (preserved verbatim regardless of budget pressure)

> For large sources (2,000+ words), Phase 5 uses progressive loading: scan headers first, then read only the sections relevant to Phase 1 purpose.

## Highlights

Context Engineering is the practice of **designing what goes into an LLM's context window** — beyond writing a single good prompt, it decides what information to give AI, when, and in what structure. RAG, memory architecture, token budget management, and system prompt design all fall under this umbrella.

- **7-phase gated pipeline** — each phase auto-evaluates criteria and passes or pauses for confirmation
- **4 modular sub-skills** — enter the pipeline at any phase when prior phases are already complete
- **3 output formats** auto-determined from your stated intent (execution instruction / KB entry / project artifacts)
- **KB maintenance mode** — `--consolidate` detects duplicate, stale, and orphaned entries
- **Pure markdown** — no runtime dependencies, no build step

## Quick Start

```bash
# 1. Add to marketplace
claude plugin marketplace add https://github.com/SeokRae/context-engineering.git

# 2. Install plugin
claude plugin install context-engineering
```

Restart Claude Code after installation for changes to take effect.

**Verify installation:**
```bash
claude plugin list
#   ❯ context-engineering@context-engineering
#     Version: 3.1.0
#     Scope: user
#     Status: ✔ enabled
```

**First use:** Type `/context-engineering` and answer the 3 questions about your purpose, constraints, and success criteria. The pipeline handles the rest.

To run directly from a local directory:
```bash
claude --plugin-dir /path/to/context-engineering
```

## Pipeline

```
Phase 1 --[G1]--> Phase 2 --[G2]--> Phase 3 --[G3]-->
Phase 4 --[G4]--> Phase 5 --[G5]--> Phase 6 --[G6]--> [Output]
                                                          |
                                                     (G6 fail)
                                                          v
                                                       Phase 7
```

> **Interactive visualization** → [context-engineering-cycle.html](https://seokrae.github.io/context-engineering/context-engineering-cycle.html)

| Sub-skill | Phases | What it does |
|-----------|--------|--------------|
| `/context-engineering:gather` | 1–3 | Define problem (3 questions) → collect candidates → select (Keep/Skip/Merge) |
| `/context-engineering:build` | 4–5 | Structure into Key Facts/Constraints/Decisions/Notes → compress with token budget |
| `/context-engineering:compose` | 6 | Auto-determine format (A/B/C) and generate output |
| `/context-engineering:verify` | 7 | 4-check validation (omission/conflict/speculation/consistency) + confidence rating |

Each gate auto-passes when criteria are clearly met. When ambiguous, the pipeline pauses for user confirmation.

## Output Types

Phase 6 auto-determines the output format from Phase 1 success criteria:

| Format | Trigger signals | Output |
|--------|----------------|--------|
| **A — Execution Instruction** (System Prompt) | "make a prompt", "how should I write", "give me instructions" | Role + Task + Constraints + Output Format — assembled as a system prompt for the target AI |
| **B — KB Entry** (Memory) | "save this", "remember this", "take a note" | Markdown entry with frontmatter + Key Facts / Constraints / Decisions / Notes |
| **C — Project Artifacts** (Persistent System Prompt) | "start a project", "analyze codebase", "I want to build" | CLAUDE.md (persistent system prompt) + spec.md + implementation plan |

If the signal is ambiguous, Phase 6 asks you to choose A, B, or C once.

## Usage

### Full pipeline

Runs all 7 phases from Phase 1 through Phase 6 (Phase 7 triggered automatically if Phase 6 gate fails).

```
/context-engineering
```

### Sub-skills — direct phase entry

Use when re-working a specific phase, returning via a feedback loop, or when prior phases are already complete.

| Command | Phases | When to use |
|---------|--------|-------------|
| `/context-engineering:gather` | 1–3 | New problem, or re-defining scope / re-collecting sources |
| `/context-engineering:build` | 4–5 | Re-structuring or re-compressing existing context |
| `/context-engineering:compose` | 6 | Re-generating output with updated context |
| `/context-engineering:verify` | 7 | Manual verification pass or `--consolidate` KB maintenance |

```
/context-engineering:gather
/context-engineering:build
/context-engineering:compose
/context-engineering:verify
/context-engineering:verify --consolidate
```

## Reference Templates

Phase artifact templates are in `skills/context-engineering/references/`:

| File | Phase | Purpose |
|------|-------|---------|
| [`phase-checklist.md`](skills/context-engineering/references/phase-checklist.md) | All | 7-phase progress checklist |
| [`knowledge-base-template.md`](skills/context-engineering/references/knowledge-base-template.md) | 6-C | KB artifact template (Format C output) |
| [`claude-md-policy-template.md`](skills/context-engineering/references/claude-md-policy-template.md) | 6-C | CLAUDE.md template (Format C output) |
| [`spec-template.md`](skills/context-engineering/references/spec-template.md) | 6-C | Architecture decisions + package structure (Format C output) |
| [`architecture-principles.md`](skills/context-engineering/references/architecture-principles.md) | — | Hexagonal + DDD principles (optional, for complex services) |
| [`entry-template.md`](skills/context-engineering/references/entry-template.md) | 6-B | KB entry format (frontmatter + body) |
| [`index-template.md`](skills/context-engineering/references/index-template.md) | 6-B | KB index format |
| [`context-session-template.md`](skills/context-engineering/references/context-session-template.md) | 6-A | Multi-step task scratch file (deleted after use) |
| [`context-source-strategy.md`](skills/context-engineering/references/context-source-strategy.md) | 2 | RAG / Memory / Tool Result / System Prompt strategy classification |
| [`failure-cases.md`](skills/context-engineering/references/failure-cases.md) | All | Pipeline failure scenarios + recovery protocols |

## Project Structure

```
context-engineering/
├── .claude-plugin/
│   ├── plugin.json          # Plugin metadata (v3.1.0)
│   └── marketplace.json     # Marketplace registration
├── skills/
│   ├── context-engineering/
│   │   ├── SKILL.md         # Orchestrator — full 7-phase pipeline
│   │   └── references/      # 10 artifact templates and reference docs
│   ├── gather/SKILL.md      # Phase 1–3
│   ├── build/SKILL.md       # Phase 4–5
│   ├── compose/SKILL.md     # Phase 6
│   └── verify/SKILL.md      # Phase 7
├── README.md
├── README.ko.md
└── LICENSE
```

## Contributing

1. Fork and clone the repository
2. Edit the relevant `SKILL.md` file(s)
3. Test by running `/context-engineering` in a real Claude Code session
4. Verify Phase 1 questions appear and gates behave correctly
5. Submit a pull request

When modifying skills, follow the guidelines in [`CLAUDE.md`](CLAUDE.md):
- Maintain phase consistency (gather → build → compose → verify)
- Sync templates in `references/` with inline structures in SKILL.md files
- Place each skill's core behavioral contract within the first 5,000 tokens (compaction safety)

## License

[MIT](LICENSE)
