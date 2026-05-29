# ADR 0002 — Use Drizzle ORM over Prisma

**Date:** 2025  
**Status:** Accepted  
**Deciders:** Project team

---

## Context

The project needs a PostgreSQL ORM for Node.js + TypeScript. The two dominant choices are Prisma and Drizzle ORM. Both provide type safety, but they have different tradeoffs relevant to this project.

---

## Decision

We will use **Drizzle ORM** for all database interactions because it stays closer to SQL, has zero runtime overhead from a query builder (no binary), and keeps the schema in TypeScript rather than a separate `.prisma` DSL file.

---

## Consequences

### Positive
- Schema lives in `src/api/db/schema.ts` — a single TypeScript file, no custom DSL to learn
- Drizzle generates SQL migrations as plain `.sql` files that are easy to review and audit
- Drizzle Studio provides a visual browser for the DB during development
- Lighter dependency footprint — no Prisma Engine binary
- Full SQL expressiveness when needed — complex joins don't require workarounds

### Negative / Trade-offs
- Less ecosystem tooling than Prisma (fewer plugins, blog posts, Stack Overflow answers)
- No built-in relation-level cascade safety — must be enforced at the SQL schema level
- Drizzle's `.returning()` on batch inserts requires PostgreSQL ≥ 15 (we require PG 15 anyway)

### Neutral
- Migration workflow is explicit: `npm run db:generate` then `npm run db:migrate` — two steps vs Prisma's one

---

## Alternatives Considered

| Option | Why rejected |
|--------|-------------|
| Prisma | Binary engine adds ~50MB; schema DSL is a second language to maintain; `npx prisma generate` required after every schema change |
| Raw `pg` / `postgres.js` | Too much boilerplate for CRUD; no type-safe query builder |
| TypeORM | Decorator-heavy; historically buggy with complex migrations |

---

## References

- [Drizzle ORM docs](https://orm.drizzle.team)
- [Drizzle vs Prisma comparison](https://orm.drizzle.team/docs/prisma-comparisons)
