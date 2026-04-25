# Template Instructions

This is a template for generating knowledge base entries. **Everything below the `---` line is the actual template structure to fill in.**

## Guidelines for AI Reading This Template

**Structure:**
- Keep entries between 20–80 lines total (including frontmatter)
- Omit empty sections entirely
- Use bullet lists for facts, constraints, decisions, and notes
- One entry = one concept or related cluster

**Naming Convention:**
- Filename: `{domain}/{slugified-title}.md`
- Example: `database/transaction-isolation-levels.md`
- Example: `architecture/hexagonal-pattern-principles.md`

**Frontmatter field reference:**
- `title`: Human-readable concept name
- `domain`: Category (e.g., database, architecture, security, api-design)
- `type`: entry classification — choose one: `fact` (observable truth), `decision` (choice made), `constraint` (limitation), `procedure` (how-to steps), `reference` (link to external source)
- `source`: Origin — format as one of: `user`, `document:FILENAME`, `url:HTTPS_URL`
- `date`: ISO 8601 (YYYY-MM-DD) when entry was created or last verified
- `reliability`: Confidence level — choose one: `high` (verified), `medium` (assumed), `low` (experimental)
- `tags`: Searchable keywords (array)
- `related`: Links to other entries in the KB (array of relative paths, e.g., `["domain/entry-name", "domain/another-entry"]`)

**When to split entries:**
- If an entry grows beyond 80 lines, split it into multiple focused entries
- Link related entries using the `related` field in frontmatter

---

## Template: Knowledge Entry

---
title: "Example: Knowledge Entry Title"
domain: "domain-name"
type: fact
source: "user"
date: "2026-04-25"
reliability: "high"
tags: ["tag1", "tag2", "concept"]
related: ["domain/related-entry", "domain/another-entry"]
---

> One-line summary of this knowledge entry.

## Key Facts

- Fact 1
- Fact 2
- Fact 3

## Constraints

- Constraint 1
- Constraint 2

## Decisions

- Decision 1 and rationale
- Decision 2 and rationale

## Notes

- Implementation note
- Edge case handling
- Additional context
