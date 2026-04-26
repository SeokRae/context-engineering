# Context Engineering

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

> A Claude Code plugin for systematically building context to send to AI — 7-Phase pipeline.

## Project Nature

No source code. **Pure markdown** Claude Code plugin:
- `.claude-plugin/plugin.json` — Plugin metadata (v3.1.0)
- `skills/context-engineering/SKILL.md` — Orchestrator (full 7-Phase pipeline)
- `skills/context-engineering/references/` — 10 templates and reference documents
- `skills/gather/SKILL.md` — Phase 1-3 (problem definition → collection → selection)
- `skills/build/SKILL.md` — Phase 4-5 (structuring → compression)
- `skills/compose/SKILL.md` — Phase 6 (execution instruction generation)
- `skills/verify/SKILL.md` — Phase 7 (final verification)

## Pipeline

```
Phase 1 →[G1]→ Phase 2 →[G2]→ Phase 3 →[G3]→
Phase 4 →[G4]→ Phase 5 →[G5]→ Phase 6 →[G6]→ [output]
                                                  ↓ (G6 fail)
                                                Phase 7
```

| Sub-skill | Phase | Role |
|-----------|-------|------|
| `gather` | 1-3 | Problem definition → candidate collection → selection |
| `build` | 4-5 | Structuring → compression |
| `compose` | 6 | Execution instruction generation |
| `verify` | 7 | Final verification |

Phase 6 automatically determines output format based on Phase 1 success criteria signal:
- **Format A**: Execution instruction (Role + Task + Constraints + Output Format)
- **Format B**: KB entry (frontmatter + Key Facts/Constraints/Decisions/Notes)
- **Format C**: Project artifacts (CLAUDE.md + spec.md + implementation plan)

## Skill Modification Guidelines

1. **Phase consistency**: Maintain the gather → build → compose → verify flow
2. **Template sync**: Always keep `references/` files and SKILL.md inline structures in sync
3. **Gate format**: Follow `"Phase N result: {summary}\n Unmet items: {content}\n Please revise or approve to continue."` format
4. **Gate behavior**: Clearly met criteria → auto-pass, ambiguous → request user confirmation
5. **Compaction Survival**: Place each SKILL.md's core behavioral contract within the **first 5,000 tokens**. Put supplementary information after — it may be truncated during context compaction

## Verification

After modifying a skill, run `/context-engineering @<path>` in a real project to confirm Phase 1 queries start correctly.
