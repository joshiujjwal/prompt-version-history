# AGENTS.md — prompt-version-history

AI agent configuration following the OpenAI Codex standard.

---

## Setup

```bash
# Install dependencies (Node.js ≥ 20 required)
npm install

# Set up environment
cp .env.example .env
# Configure DATABASE_URL, JWT_SECRET, JWT_REFRESH_SECRET in .env

# Run migrations
npm run db:migrate

# Seed the database
npm run db:seed
```

---

## Running the Project

```bash
# Development (API + frontend concurrently)
npm run dev

# API only (port 3001)
npm run dev:api

# Frontend only (port 5173)
npm run dev:frontend

# Production build
npm run build
```

---

## Testing

**Always write tests before implementing (red/green TDD). Never skip tests.**

```bash
# All tests
npm test

# Unit tests only
npm run test:unit

# Integration tests (requires TEST_DATABASE_URL in .env)
npm run test:integration

# E2E tests (requires both dev servers running)
npm run test:e2e

# Watch mode during development
npm run test:watch

# Coverage report
npm run test:coverage
```

### Test File Conventions
- Unit tests: `tests/unit/<mirrors-src-path>.test.ts`
- Integration tests: `tests/integration/<resource>.test.ts`
- E2E tests: `tests/e2e/<feature>.spec.ts`
- Test files **must not** import from `../../../src/api` in frontend test files or vice versa
- Use `describe` blocks named after the function/component under test
- Each `it`/`test` description must be a complete sentence: `"returns 404 when prompt does not exist"`

---

## Code Style

### TypeScript
- `strict: true` — no `any`, no `!` non-null assertions without a comment explaining why
- Explicit return types on all functions (enforced by ESLint)
- Use `type` for object shapes; use `interface` only when you need declaration merging
- Path aliases: `@api/*`, `@frontend/*`, `@shared/*` — never use relative `../../..` imports across workspace boundaries

### Express / Backend
- Route files export an Express Router — no business logic in route handlers, only delegation to services
- All request bodies validated with **Zod** before reaching the service layer
- Services return typed results — never throw HTTP errors from services (only from middleware)
- Database calls go through query functions in `src/api/db/queries/` — no inline Drizzle calls in services

### React / Frontend
- Functional components only — no class components
- Data fetching via **React Query** hooks in `src/frontend/hooks/`
- Components in `src/frontend/components/` must be pure: no direct `fetch`, no `useQuery`
- Use Tailwind CSS utility classes — no inline `style` props, no CSS modules
- All forms use **React Hook Form** with Zod resolver for validation

### General
- No `console.log` in committed code — use the structured logger
- No hardcoded URLs or secrets — all via `process.env` (typed via `src/api/lib/env.ts`)
- File naming: `kebab-case.ts` for files, `PascalCase` for React components

---

## Pull Request Requirements

Every PR **must** include in the description:

1. **What changed** — one paragraph summary
2. **Test evidence** — paste the output of `npm test` (or the relevant subset)
3. **Manual verification** — describe the exact steps you took to verify the change works
4. **For UI changes** — before/after screenshots or screen recording
5. **Spec reference** — link to the relevant section of `docs/spec.md` or explain why this is out of scope

PRs without test evidence will not be merged.

---

## Architecture Constraints

- `src/shared/types/` is the only cross-workspace import boundary — both API and frontend may import from here
- The diff service (`src/api/services/diff.service.ts`) must remain a pure function (no DB, no I/O)
- Prompt version content is always stored as plain text — no base64, no compression
- Version numbers are immutable once assigned — never reuse or renumber
- All destructive operations (delete project, remove member) are **owner-only** and must be confirmed by checking role in `project_members`

---

## Key Files

| File | Purpose |
|------|---------|
| `src/api/db/schema.ts` | Drizzle schema — single source of truth for all tables |
| `src/api/services/diff.service.ts` | Pure diff algorithm — the core engine |
| `src/frontend/lib/api.ts` | All API calls from the frontend — typed client |
| `src/shared/types/index.ts` | Types shared between API and frontend |
| `docs/spec.md` | Feature specification with data model and API design |
| `TODO.md` | Task breakdown with evidence gates |
| `CLAUDE.md` | Extended context for Claude-based agents |
