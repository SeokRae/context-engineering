---
title: "Example: Knowledge Entry Title"
domain: "domain-name"
type: "fact|decision|constraint|procedure|reference"
source: "user|document:filename|url:https://example.com"
date: "2026-04-25"
reliability: "high|medium|low"
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

---

## Usage

**Structure Guidelines:**
- Keep entries between 20–80 lines total (including frontmatter)
- Omit empty sections entirely
- Use bullet lists for facts, constraints, decisions, and notes
- One entry = one concept or related cluster

**Naming Convention:**
- Filename: `{domain}/{slugified-title}.md`
- Example: `database/transaction-isolation-levels.md`
- Example: `architecture/hexagonal-pattern-principles.md`

**When to Split:**
- If an entry grows beyond 80 lines, split it into multiple focused entries
- Link related entries using the `related` field in frontmatter

**Frontmatter Fields:**
- `title`: Human-readable concept name
- `domain`: Category (e.g., database, architecture, security, api-design)
- `type`: fact (observable truth), decision (choice made), constraint (limitation), procedure (how-to steps), reference (link to external source)
- `source`: Origin (user, document:FILENAME, url:HTTPS_URL)
- `date`: ISO 8601 (YYYY-MM-DD) when entry was created or last verified
- `reliability`: Confidence level (high = verified, medium = assumed, low = experimental)
- `tags`: Searchable keywords (array)
- `related`: Links to other entries in the KB (array of relative paths)
