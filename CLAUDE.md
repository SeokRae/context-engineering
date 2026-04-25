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

## Skill Modification Guidelines

1. **Variable consistency**: only reference variables defined in Step 0 (e.g., `{OUTPUT_PATH}`) in later phases
2. **Template sync**: keep `references/` templates and SKILL.md inline artifact structures in sync
3. **Gate message format**: standardize to `"Phase {N} complete — \`{artifact}\` draft saved. Review and proceed to Phase {N+1}?"`

## Verification

After modifying a skill, run `/context-engineering @<path>` in a real project to confirm Step 0 parameter collection works correctly.
