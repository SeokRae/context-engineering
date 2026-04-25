# SPEC — {PROJECT_NAME}

> Phase 6 artifact. Technical definition of how to build it.

---

## Architecture Decisions

| Decision | Alternatives | Choice | Rationale |
|---------|-----------|------|------|
| {what was decided} | {alt1}, {alt2} | {chosen} | {reason} |

---

## Package Structure

Dependency direction: `entry -> application -> domain <- infrastructure` (reverse forbidden)

```
{layer}: {folder path}  ← {responsibility}
```

> Principles, layer definitions, language-specific folder names → [references/architecture-principles.md](references/architecture-principles.md)

---

## Data Flow

```
{input} → {ComponentA} → {ComponentB} → {output}
```

---

## Implementation Plan

| # | Item | REQ | Depends | Done Criteria |
|------|---------|-----|------|---------|
| 1 | {first implementation unit} | REQ-1 | — | {verifiable criterion} |

---

## Requirements Traceability

| REQ-ID | Requirement | PRD Feature | Implementation Item | Status |
|--------|---------|---------|---------|------|
| REQ-1 | {what} | {feature} | Implementation Plan #1 | not implemented |
| REQ-2 | {why} | {feature} | Implementation Plan #N | not implemented |
| REQ-3 | {constraint} | {non-func/feature} | Implementation Plan #N | not implemented |
