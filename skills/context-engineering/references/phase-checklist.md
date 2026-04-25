# Context Engineering 4-Phase Checklist

Copy this file when starting a new project.

---

## Project Info

- **Project**: 
- **Code path**: 
- **Output path**: 
- **Tech stack**: 
- **Start date**: 

---

## Step 0: Context Gathering

- [ ] Requirements exploration — confirm What / Why / known constraints
- [ ] Requirements summary user confirmation (HARD-GATE passed)
- [ ] Technical parameters collected (PROJECT_NAME, CODE_PATH, TECH_STACK, etc.)
- [ ] Final parameter summary user confirmation

## Phase 1: Knowledge Base

- [ ] Context Assessment — scenario detection (A/B/C/D) + user confirmation
- [ ] Source Scan complete (conditional on scenario)
  - [ ] A: Extract domain concepts from Step 0 conversation
  - [ ] B: Explore sub-agent parallel scan of spec documents
  - [ ] C: Codebase scan (package structure, patterns, test strategy)
  - [ ] D: B + C run in parallel
- [ ] Domain Glossary written (minimum 5 terms)
- [ ] Constraint Registry written (TC/BL/OC categories)
- [ ] Reference Document Manifest written (scenarios B/C/D)
- [ ] `knowledge-base.md` saved (Scenario A marked as "Draft")
- [ ] **Self-Assessment performed** (normal mode) — output with gate message

## Phase 2: Policy (CLAUDE.md)

- [ ] CLAUDE.md level decided (project / module)
- [ ] Architecture rules defined (layer separation, pattern constraints)
- [ ] Coding conventions documented (naming, test patterns)
- [ ] Build & Test commands verified
- [ ] Non-obvious gotchas recorded
- [ ] Items exceeding 150 lines split to separate document + linked
- [ ] `{CODE_PATH}/CLAUDE.md` saved
- [ ] **Self-Assessment performed** (normal mode) — output with gate message

## Phase 3: Spec

- [ ] Architecture decision rationale complete (D-*) — alternatives / choice / rationale included
- [ ] Package/module structure finalized
- [ ] **Requirements traceability table written** — REQ-ID → PRD feature → implementation item
- [ ] **Self-Assessment performed** — before Readiness Gate output
- [ ] `spec.md` saved

## Phase 4: Implementation

- [ ] Per-module skeleton generated
- [ ] Core domain objects implemented
- [ ] Adapters implemented
- [ ] Tests written + passing
- [ ] Feedback loop applied (findings → update relevant phase)
  - [ ] Domain/term error → Phase 1 `knowledge-base.md` updated
  - [ ] AI behavior rule change → Phase 2 `CLAUDE.md` updated
  - [ ] Architecture/design change → Phase 3 `spec.md` updated
- [ ] **Requirements traceability table updated** — completed REQ-IDs marked as "implemented"
- [ ] Architecture documentation written
- [ ] Onboarding guide written
- [ ] **Implementation Retrospective completed** — requirements achievement / process retrospective / final document state

---

## Artifact Self-Assessment Criteria

Performed immediately before the phase gate, in normal mode only. Format:

```
| Item | Status | Action |
|------|--------|--------|
| {criterion} | OK / {gap description} | — / {specific action} |
```

**Phase 1 (Knowledge Base)**

| Criterion | Met condition | Action if unmet |
|-----------|--------------|-----------------|
| Term sufficiency | Glossary 5+ terms, no key domain concepts missing | Re-run Source Scan or supplement Step 0 conversation |
| Constraint categories | At least 2 of TC/BL/OC categories used | Expand constraint registry entries |
| Source attribution | All terms have source (document name / conversation) recorded | Supplement items missing source |

**Phase 2 (CLAUDE.md)**

| Criterion | Met condition | Action if unmet |
|-----------|--------------|-----------------|
| Build command verified | BUILD_CMD, TEST_CMD execution confirmed | Re-check build files and update |
| High constraints reflected | Phase 1 High-priority constraints included in Core Constraints section | Supplement Core Constraints section |
| KB link exists | `knowledge-base.md` link present in References section | Add link to References section |

**Phase 3 (Spec)**

| Criterion | Met condition | Action if unmet |
|-----------|--------------|-----------------|
| Decision rationale completeness | All architecture decisions include alternatives and rationale | Supplement missing decision entries |
| Implementation plan dependencies mapped | Implementation plan includes dependency relationships and done criteria | Supplement implementation plan entries |
| REQ coverage verified | All REQ-IDs mapped to implementation plan items | Supplement requirements traceability table |
