# CLAUDE.md Policy Template

> Phase 6 artifact. Use this template to write `{CODE_PATH}/CLAUDE.md`. Target length: 100~150 lines. Move overflows to separate documents.

---

```markdown
# {PROJECT_NAME}

This file provides guidance to Claude Code when working with code in this repository.

> {one-line description}

## Architecture

**Design rationale (SoC + Cohesion/Coupling)**: Code that changes for the same reason stays together; code that changes for different reasons is separated. One module, one concern.

**Layer structure (Ports & Adapters + DDD)**

| Layer | Folder | Ports & Adapters | Responsibility (SRP) | Dependency rule (DIP) | DDD concept |
|-------|--------|-----------------|---------------------|----------------------|-------------|
| domain | `{path}` | — (pure domain) | Business rules & invariants | No external dependencies | Entity, Value Object, Aggregate, Repository port |
| application | `{path}` | — (Use Case) | Use case orchestration (CQS applied) | Domain only | Command Handler, Query Handler |
| infrastructure | `{path}` | Driven Adapter (Secondary) | External system implementations | Domain port inversion (DIP) | Repository impl, API Adapter |
| entry | `{path}` | Driving Adapter (Primary) | Input reception & transformation | Application only | Controller, Consumer |
| config | `{path}` | — | Dependency assembly | All layers | DI configuration |

Dependency direction: `entry -> application -> domain <- infrastructure` (reverse forbidden)

**Responsibility assignment (GRASP Information Expert)**: Responsibility belongs to the object that holds the information. Calculation logic lives in the domain object that owns the data — do not delegate to Service.

**Layer boundary rules (OCP)**: Replace external API → modify Secondary Adapter only. New use case → extend application, existing closed.

## Build & Test

\`\`\`bash
# full build
{BUILD_CMD}

# unit tests
{TEST_CMD}

# specific module test
{MODULE_TEST_CMD}

# run server
{RUN_CMD}
\`\`\`

## Ports / Endpoints

| Port | Protocol | Purpose |
|------|----------|---------|
| {port} | {TCP/HTTP} | {purpose} |

## Core Constraints

{Only items from Phase 1 TC/BL that directly affect development — 5 max}

- TC-1: {constraint}
- BL-1: {rule}

## Non-obvious Gotchas

> Traps that can't be seen from code alone. Only what a new team member must know.

- {gotcha 1}: {description}
- {gotcha 2}: {description}

## References

- [{document name}]({path}): {one-line description}
```

---

## Writing Guidelines

1. **WHAT/WHY/HOW**: what (architecture), why (design intent), how (commands)
2. **Line limit**: project level 100~150 lines, module level 60~100 lines
3. **When over limit**: split to separate `docs/WORKFLOW_*.md` and keep only the link
4. **Gotchas first**: do not write what can be read from code. Traps only.
5. **Command verification**: record only after actually running and confirming it works
