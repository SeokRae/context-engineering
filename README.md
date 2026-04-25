![Version](https://img.shields.io/badge/version-3.0.0-blue)
![License](https://img.shields.io/github/license/SeokRae/context-engineering)
![Claude Code Plugin](https://img.shields.io/badge/Claude_Code-Plugin-blueviolet)

# context-engineering

> [한국어](README.ko.md)

A Claude Code plugin that systematically prepares AI context through a 7-phase pipeline — deciding what information to include, in what structure, at what level of compression.

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
#     Version: 3.0.0
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
| **A — Execution Instruction** | "make a prompt", "how should I write", "give me instructions" | Role + Task + Constraints + Output Format |
| **B — KB Entry** | "save this", "remember this", "take a note" | Markdown entry with frontmatter + Key Facts / Constraints / Decisions / Notes |
| **C — Project Artifacts** | "start a project", "analyze codebase", "I want to build" | CLAUDE.md + spec.md + implementation plan |

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
| [`knowledge-base-template.md`](skills/context-engineering/references/knowledge-base-template.md) | 1 | KB artifact template |
| [`claude-md-policy-template.md`](skills/context-engineering/references/claude-md-policy-template.md) | 2 | CLAUDE.md template |
| [`spec-template.md`](skills/context-engineering/references/spec-template.md) | 3 | Architecture decisions + package structure |
| [`architecture-principles.md`](skills/context-engineering/references/architecture-principles.md) | — | Hexagonal + DDD principles (optional, for complex services) |
| [`entry-template.md`](skills/context-engineering/references/entry-template.md) | 6-B | KB entry format (frontmatter + body) |
| [`index-template.md`](skills/context-engineering/references/index-template.md) | 6-B | KB index format |
| [`context-session-template.md`](skills/context-engineering/references/context-session-template.md) | 6-A | Multi-step task scratch file (deleted after use) |

## Project Structure

```
context-engineering/
├── .claude-plugin/
│   ├── plugin.json          # Plugin metadata (v3.0.0)
│   └── marketplace.json     # Marketplace registration
├── skills/
│   ├── context-engineering/
│   │   ├── SKILL.md         # Orchestrator — full 7-phase pipeline
│   │   └── references/      # 8 artifact templates
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
