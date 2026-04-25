# Knowledge Base Index

**Last updated:** 2026-04-25  
**Total entries:** 42  
**Domains:** 6 (architecture, database, api-design, security, infrastructure, patterns)

---

## Architecture

| Entry | Type | Date | Reliability | Tags | Summary |
|-------|------|------|-------------|------|---------|
| [Hexagonal Pattern Principles](./architecture/hexagonal-pattern-principles.md) | decision | 2026-04-20 | high | architecture, design-pattern, layering | Core architectural pattern emphasizing domain independence from frameworks |
| [Service Boundary Rules](./architecture/service-boundary-rules.md) | constraint | 2026-04-18 | high | architecture, microservices | Defines when to split services and how to model dependencies |
| [Event-Driven Communication](./architecture/event-driven-communication.md) | procedure | 2026-04-15 | medium | architecture, messaging, async | Step-by-step guide for implementing event sourcing patterns |

## Database

| Entry | Type | Date | Reliability | Tags | Summary |
|-------|------|------|-------------|------|---------|
| [Transaction Isolation Levels](./database/transaction-isolation-levels.md) | fact | 2026-04-22 | high | database, transactions, concurrency | Overview of ACID properties and isolation level tradeoffs |
| [Index Strategy](./database/index-strategy.md) | decision | 2026-04-10 | high | database, performance, indexing | Rules for when and how to index columns |

---

## Index Management

**Updated atomically on every ingest:** When new knowledge entries are added to the knowledge base, the index is regenerated completely. This ensures consistency between the index table and the actual files on disk.

**Used for candidate search on every query:** The index table enables fast filtering by domain, type, date, and reliability before fetching full entry contents. Search logic:
1. Filter candidates by domain and tags
2. Sort by reliability and date (newest first)
3. Fetch matching entries and return ranked results

**Regeneration procedure:**
- Scan all markdown files in the KB
- Extract frontmatter and first content line
- Sort entries by domain, then by filename
- Rebuild the index table
- Commit and publish index update

---

## How to Use This Index

1. **Find an entry:** Search by domain name or use the table of contents
2. **Read an entry:** Click the linked title to open the full markdown file
3. **Understand reliability:** Check the Reliability column (high = verified, medium = assumed, low = experimental)
4. **See relationships:** Review the `related` field in each entry's frontmatter to discover connected concepts
