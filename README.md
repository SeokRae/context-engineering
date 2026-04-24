# context-engineering

Claude Code plugin for AI-assisted service development using a 4-phase Context Engineering workflow.

## Overview

When building complex services with AI tools, you need to answer three questions:
- **What to know** → Knowledge Base (domain specs, constraints)
- **How to behave** → Policy (CLAUDE.md)
- **What to build** → Spec (requirements, design decisions, WBS)

This plugin provides a `/context-engineering` skill that guides you through these phases step by step.

## Installation

```bash
# 1. 마켓플레이스(저장소)를 등록한다
claude plugin marketplace add https://github.com/SeokRae/context-engineering.git

# 2. 플러그인을 설치한다
claude plugin install context-engineering
```

로컬 디렉토리에서 바로 사용할 때:

```bash
claude --plugin-dir /path/to/context-engineering
```

## Usage

```
/context-engineering @<code-path> [--obsidian <obsidian-area>] [--specs <spec-docs-path>]
```

**Example:**
```
/context-engineering @/Users/sr/IdeaProjects/payment/fx --obsidian 20-areas/payment/fix --specs docs/references/
```

## Workflow

```
Step 0: Context Gathering
  ↓
Phase 1: Knowledge Base  →  review
  ↓
Phase 2: Policy (CLAUDE.md)  →  review
  ↓
Phase 3: Spec  →  review
  ↓
Phase 4: Implementation
```

## References

Each phase has a template in `skills/context-engineering/references/`:

| File | Purpose |
|------|---------|
| `phase-checklist.md` | Copy per project, track progress |
| `knowledge-base-template.md` | Phase 1 output template |
| `claude-md-policy-template.md` | Phase 2 CLAUDE.md template |
| `spec-template.md` | Phase 3 FR / design decisions / WBS template |

## License

MIT
