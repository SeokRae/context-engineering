# Template Instructions

This is a template for generating the knowledge base index. The values below are **EXAMPLES** showing the structure — replace them with actual values from your knowledge base.

## Guidelines for AI Reading This Template

**Index generation:**
- Replace example header titles (Architecture, Database) with actual domain names from your KB
- Replace example entries with real entries from `knowledge-base/` directory
- Extract metadata (Title, Type, Date, Reliability, Tags, first line summary) from each entry's frontmatter and content

**Index updates:**
- Regenerate completely when new entries are added
- Sort entries by domain, then by filename
- Ensure the count matches actual entries on disk

**Index structure:**
- One section per domain (alphabetical or by frequency)
- Table columns: Entry | Type | Date | Reliability | Tags | Summary
- Summary = first content line (the `>` blockquote) from each entry file
- Links = relative paths from index location to entry files

---

## Template: Knowledge Base Index

# Knowledge Base Index

> Last updated: {YYYY-MM-DD} | Total entries: {N} | Domains: {domain-a, domain-b, ...}

---

## Architecture (example domain)

| Entry | Type | Date | Reliability | Tags | Summary |
|-------|------|------|-------------|------|---------|
| [Hexagonal Pattern Principles](./architecture/hexagonal-pattern-principles.md) | decision | 2026-04-20 | high | architecture, design-pattern, layering | Core architectural pattern emphasizing domain independence from frameworks |
| [Service Boundary Rules](./architecture/service-boundary-rules.md) | constraint | 2026-04-18 | high | architecture, microservices | Defines when to split services and how to model dependencies |
| [Event-Driven Communication](./architecture/event-driven-communication.md) | procedure | 2026-04-15 | medium | architecture, messaging, async | Step-by-step guide for implementing event sourcing patterns |

## Database (example domain)

| Entry | Type | Date | Reliability | Tags | Summary |
|-------|------|------|-------------|------|---------|
| [Transaction Isolation Levels](./database/transaction-isolation-levels.md) | fact | 2026-04-22 | high | database, transactions, concurrency | Overview of ACID properties and isolation level tradeoffs |
| [Index Strategy](./database/index-strategy.md) | decision | 2026-04-10 | high | database, performance, indexing | Rules for when and how to index columns |

---

## How to Use This Index

1. **Find an entry:** Search by domain name or use the table of contents
2. **Read an entry:** Click the linked title to open the full markdown file
3. **Understand reliability:** Check the Reliability column (high = verified, medium = assumed, low = experimental)
4. **See relationships:** Review the `related` field in each entry's frontmatter to discover connected concepts
