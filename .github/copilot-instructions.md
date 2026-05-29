# GitHub Copilot Instructions — prompt-version-history

## Project Context

This is a TypeScript monorepo for a Git-like version control system for AI prompts and their LLM responses. Teams use it to track prompt evolution, compare outputs across versions, and roll back to previous prompt states.

**Stack:** TypeScript (strict) · React 18 · Node.js 20 · Express · Drizzle ORM · PostgreSQL 15 · Vitest · Playwright

---

## Coding Conventions

### TypeScript
- Always use `strict: true` settings — no `any`, no implicit `any`, no `!` without an explanatory comment
- Explicit return types on every function and method
- Use `type` for data shapes; `interface` only when extending or merging declarations
- Use `z.infer<typeof Schema>` to derive types from Zod schemas — don't duplicate type definitions

### Backend (Express + Drizzle)
- Route handlers must be thin: validate input (Zod) → call service → return response
- All business logic belongs in `src/api/services/` — no DB calls in routes
- All DB queries belong in `src/api/db/queries/` — no Drizzle calls outside this directory
- Use `drizzle-zod` to auto-generate Zod schemas from Drizzle table definitions where possible
- Never use `SELECT *` — always specify columns explicitly in Drizzle queries
- Always handle the "not found" case and return a typed `404` response

### Frontend (React + React Query)
- Components are presentational — they receive data and callbacks as props
- Data fetching lives in hooks (`src/frontend/hooks/`) using `useQuery` and `useMutation`
- All API calls go through `src/frontend/lib/api.ts` — never call `fetch` directly in a component
- Prefer `const` arrow functions for components: `const MyComponent = () => { ... }`
- Use Tailwind CSS utility classes exclusively — no `style` props, no CSS modules
- Forms: React Hook Form + Zod resolver — no uncontrolled inputs

---

## Testing Conventions

- **Write the test before the implementation** — this is non-negotiable
- Test file names mirror the source file: `src/services/foo.ts` → `tests/unit/services/foo.test.ts`
- Test descriptions are complete sentences: `"returns 422 when commit message is empty"`
- Unit tests must not touch the database, filesystem, or network — use `vi.mock()` for dependencies
- Integration tests use a real PostgreSQL test database (`TEST_DATABASE_URL`)
- Never use `expect(true).toBe(true)` — every assertion must be meaningful
- Keep test setup in `beforeEach`, teardown in `afterEach` — no shared mutable state across tests

---

## Boundaries (Things Copilot Must Not Do)

- **Do not refactor** existing code unless explicitly asked — suggest in a comment instead
- **Do not remove or skip tests** — if a test is failing, fix the implementation, not the test
- **Do not use** `any`, `unknown` without a cast, or type assertions without a comment
- **Do not inline** database queries in route handlers or services
- **Do not call** `fetch` or axios directly in React components — always use `src/frontend/lib/api.ts`
- **Do not add** new environment variables without updating `.env.example` and `CLAUDE.md`
- **Do not hardcode** port numbers, URLs, or secrets in source files
- **Do not mutate** existing `prompt_versions` rows — versions are immutable by design
- **Do not use** `console.log` in committed code — use the structured logger at `src/api/lib/logger.ts`

---

## Domain Glossary

| Term | Meaning |
|------|---------|
| `prompt` | A named prompt belonging to a project — like a file in a Git repo |
| `version` | An immutable snapshot of prompt content — like a commit |
| `commit message` | Required text describing why a version was created |
| `rollback` | Creating a new version whose content copies an older version |
| `response` | An LLM output stored against a specific prompt version |
| `diff` | Structured comparison between two prompt version contents |
| `tag` | A named pointer to a specific version (e.g., "production") |
| `project` | A container for related prompts — like a Git repository |
