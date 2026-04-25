# context-engineering

This file provides guidance to Claude Code when working with code in this repository.

> 4-phase Context Engineering workflow Claude Code plugin for developing complex services with AI tools

## Project Nature

No source code. **Pure markdown** Claude Code plugin:
- `plugin.json` — plugin metadata
- `skills/context-engineering/SKILL.md` — full workflow skill (Step 0 → Phase 4)
- `skills/context-engineering/references/` — phase artifact templates
- `skills/gather/SKILL.md` — sub-skill: Step 0 + Phase 1 (KB) + Phase 2 (CLAUDE.md)
- `skills/spec/SKILL.md` — sub-skill: Phase 3 (PRD + SPEC)
- `skills/valid/SKILL.md` — sub-skill: Readiness Gate validation
- `skills/impl/SKILL.md` — sub-skill: Phase 4 (implementation)

## ke Skill Structure

New in v2.0. Universal knowledge engine, domain-agnostic:
- `skills/ke/SKILL.md` — main 7-phase engine + auto mode detection (`/ke`)
- `skills/ke/references/entry-template.md` — knowledge entry format template
- `skills/ke/references/index-template.md` — index.md format template
- `skills/ingest/SKILL.md` — direct ingest entry (`/ke:ingest`)
- `skills/query/SKILL.md` — direct query entry (`/ke:query`)

## Skill Modification Guidelines

1. **Variable consistency**: only reference variables defined in Step 0 (e.g., `{OUTPUT_PATH}`) in later phases
2. **Template sync**: keep `references/` templates and SKILL.md inline artifact structures in sync
3. **Gate message format**: standardize to `"Phase {N} complete — \`{artifact}\` draft saved. Review and proceed to Phase {N+1}?"`

## Verification

After modifying a skill, run `/context-engineering @<path>` in a real project to confirm Step 0 parameter collection works correctly.

## ke Design Invariants

These rules must hold for the ke skill and sub-skills:

1. **Mode detection**: `/ke` auto-detects Ingest vs. Query from input signals. Never ask the user which mode unless genuinely ambiguous. Ambiguous → 1 question with options 1/2/3.
2. **KNOWLEDGE_PATH resolution order**: `--kb` arg → `KNOWLEDGE_PATH` env → `./knowledge/index.md` → `~/knowledge/index.md` → ask user.
3. **Lightweight gates**: Only 2 checkpoints — after Phase 3 (all-duplicate ingest) and after Phase 7 (conflicts for ingest; confidence footer for query). Never gate on anything else.
4. **Query is read-only**: `/ke:query` and `/ke:ingest`'s Query mode never write files. Only Ingest writes.
5. **Concept granularity**: One knowledge entry per concept, not per document or session. Large documents are split into multiple entries.
6. **Index sync**: index.md is updated atomically on every ingest. Re-sort domain tables by date descending on every update.
