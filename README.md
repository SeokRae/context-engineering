# context-engineering

> [한국어](README.ko.md) | English

A Claude Code plugin that defines step-by-step **"what to know, how to behave, and what to build"** when developing complex services with AI tools.

## What is Context Engineering?

Context Engineering is **the practice of designing what goes into an LLM's context window**. It goes beyond writing a single good prompt — it decides what information to give to AI, when, and in what structure. RAG, memory architecture, token budget management, and system prompt design all fall under this category.

This plugin addresses one aspect of that — **persistent context preparation** — a workflow to systematically create and maintain `knowledge-base.md` and `CLAUDE.md` so AI starts each development session with the right domain knowledge and behavior rules.

## Overview

When developing complex services with AI, three questions must be answered:

- **What to know** → Knowledge Base (domain knowledge, constraints)
- **How to behave** → Policy (CLAUDE.md)
- **What to build** → Spec (PRD + architecture decisions + implementation plan)

This plugin can be used in two ways:

- **`/context-engineering`** — Sequential flow from Step 0 to Phase 4 (full workflow)
- **Sub-skills** — Direct entry to a specific phase (for rework or feedback loop)

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
#     Version: 1.0.0
#     Scope: user
#     Status: ✔ enabled
```

To use directly from a local directory:
```bash
claude --plugin-dir /path/to/context-engineering
```

## Skills

### `/context-engineering` — Full workflow

Runs sequentially from Step 0 through Phase 4.

```
/context-engineering @<code-path> [--output <output-path>] [--specs <spec-doc-path>]
```

**Examples:**
```
/context-engineering @/path/to/project
/context-engineering @/path/to/project --specs docs/references/
/context-engineering @/path/to/project --output /tmp/context-docs --specs docs/references/
```

### Sub-skills — Direct entry by phase

Use when reworking a specific phase in an ongoing project, or returning to a phase via the feedback loop.

| Command | Handles | When to use |
|---------|---------|-------------|
| `/context-engineering:gather` | Step 0 + Phase 1 (KB) + Phase 2 (CLAUDE.md) | New project start, KB or CLAUDE.md rework/update |
| `/context-engineering:spec` | Phase 3 — PRD + SPEC | Writing or updating requirements and technical spec |
| `/context-engineering:impl` | Phase 4 — Implementation | Starting implementation, entering feedback loop |
| `/context-engineering:valid` | Readiness Gate | Pre-Phase 4 validation, discovery mode exit check |

Each sub-skill auto-detects existing artifacts (knowledge-base.md, CLAUDE.md, prd.md, spec.md) and offers rework or update options.

```
/context-engineering:gather @/path/to/project
/context-engineering:spec @/path/to/project
/context-engineering:valid
/context-engineering:impl @/path/to/project
```

## Workflow

> **[📊 Interactive Diagram](https://seokrae.github.io/context-engineering/context-engineering-cycle.html)** — Click each phase to see the step-by-step details.

```
Step 0 — Context Gathering
  · Requirements exploration (What · Why · Constraints — one question at a time)
  · Assign REQ-IDs when summary is confirmed (REQ-1 · REQ-2 · REQ-3…)
  · Clear     → write Phase 1~3 thoroughly, then enter Phase 4
  · Unclear   → discovery mode: generate empty drafts → go directly to Phase 4
                 ↓
Phase 1 — Knowledge Base           [Self-Assessment] [User confirmation]
  · Context Assessment — A/B/C/D scenario detection
  · Source Scan  A: conversation  B: spec docs  C: code  D: B+C
  · knowledge-base.md (Glossary · Constraints · Manifest)
                 ↓
Phase 2 — Policy (CLAUDE.md)       [Self-Assessment] [User confirmation]
  · knowledge-base.md link required
  · Non-obvious Gotchas draft (updated via Phase 4 feedback)
                 ↓
Phase 3 — Spec              [Self-Assessment] [Readiness Gate ⑥REQ]
  · PRD feature requirements + SPEC architecture decisions + package structure
  · Requirements traceability (REQ-ID → PRD feature → SPEC → implementation plan)
  · Gate: AI summarizes current state → user decides pass/fail
  ↑──────────────────── re-cycle to relevant phase if unmet
                 ↓  Gate passed
Phase 4 — Implementation                          [Retrospective]
  · Build + Test + PRD success criteria achieved
  · Update REQ-ID status on each completion (not implemented → implemented)
  · Feedback loop: findings → immediately update the relevant phase document
  · Output Implementation Retrospective when all done (achievement · process · documents)
```

**Discovery mode**: When requirements are unclear, generate Phase 1~3 as empty drafts and go directly to Phase 4. Fill in while developing, then switch to the main track when the user requests the Readiness Gate.

## Reference Documents

Phase artifact templates are in `skills/context-engineering/references/`:

| File | Purpose |
|------|---------|
| [`phase-checklist.md`](skills/context-engineering/references/phase-checklist.md) | Per-project progress checklist |
| [`knowledge-base-template.md`](skills/context-engineering/references/knowledge-base-template.md) | Phase 1 artifact template |
| [`claude-md-policy-template.md`](skills/context-engineering/references/claude-md-policy-template.md) | Phase 2 CLAUDE.md template |
| [`spec-template.md`](skills/context-engineering/references/spec-template.md) | Phase 3 SPEC template (architecture decisions + package structure) |
| [`architecture-principles.md`](skills/context-engineering/references/architecture-principles.md) | Hexagonal + DDD principles (for complex services, optional) |

## License

MIT
