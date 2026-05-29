# prompt-version-history — Task Breakdown

## How to Use This File

One task at a time, in order. Each phase is an evidence gate — do not proceed until all items are checked and tests pass.

**Workflow per task:**
1. Read the task description fully
2. Run existing tests first (`npm test`) — they must all pass before you touch anything
3. Write failing tests (red phase)
4. Implement until tests pass (green phase)
5. Review the diff manually — no surprise changes
6. Commit with a descriptive message referencing the task
7. If you discovered anything non-obvious, update `CLAUDE.md` or `AGENTS.md`

---

## Phase 0: Foundation ⬜

- [ ] Initialize `package.json` with workspaces (root, `src/api`, `src/frontend`)
- [ ] Configure TypeScript (`tsconfig.json`) for strict mode, path aliases (`@api/*`, `@frontend/*`, `@shared/*`)
- [ ] Set up ESLint + Prettier (no `any`, enforce explicit return types)
- [ ] Set up Vitest for unit tests; confirm smoke test passes (`tests/unit/smoke.test.ts`)
- [ ] Set up Supertest for API integration tests
- [ ] Configure GitHub Actions CI: install → lint → test → build on `push` and `pull_request`
- [ ] Create `.env.example` with all required env vars documented
- [ ] Bootstrap Express server skeleton (`src/api/server.ts`) — health check route only
- [ ] Bootstrap React + Vite frontend skeleton (`src/frontend/main.tsx`)
- [ ] Confirm both `npm run dev` processes start without errors
- [ ] Review all AI config files (this file, `CLAUDE.md`, `AGENTS.md`) before writing any feature code

**Gate:** CI passes. `GET /health` returns `200`. React app renders.

---

## Phase 1: Data Layer ⬜

- [ ] Design and write Drizzle schema for core entities (see `docs/spec.md` §3)
  - `projects` — container for prompt collections
  - `prompts` — versioned prompt text with metadata
  - `prompt_versions` — immutable snapshot per commit
  - `responses` — LLM response linked to a specific version
  - `tags` — label a version (e.g., "v1.0", "production")
- [ ] Write unit tests for schema validation and constraint checks (red)
- [ ] Run `npm run db:generate` to generate migration files
- [ ] Run `npm run db:migrate` against a test DB; verify tables created
- [ ] Write a DB seed script (`src/api/db/seed.ts`) with representative data
- [ ] Write integration tests for each DB query function (red → green)

**Gate:** All DB tests pass. Migrations run clean on fresh PostgreSQL instance. Seed script loads without error.

---

## Phase 2: Core API — Prompt Version CRUD ⬜

- [ ] **Write failing tests first** for all routes below before implementing any handler
- [ ] `POST /api/projects` — create project
- [ ] `GET /api/projects` — list projects (paginated)
- [ ] `POST /api/projects/:id/prompts` — create a prompt (starts at version 1)
- [ ] `GET /api/projects/:id/prompts` — list prompts in project
- [ ] `POST /api/prompts/:id/versions` — commit a new version (content + commit message)
- [ ] `GET /api/prompts/:id/versions` — list version history (newest first)
- [ ] `GET /api/prompts/:id/versions/:versionId` — get a specific version
- [ ] `POST /api/prompts/:id/versions/:versionId/rollback` — create new version from old snapshot
- [ ] Input validation middleware (Zod schemas) for all routes
- [ ] Error handling middleware (structured JSON errors with `code`, `message`, `details`)

**Gate:** All API integration tests pass (≥90% route coverage). No unvalidated inputs reach handlers.

---

## Phase 3: Diff Engine ⬜

- [ ] Write unit tests for diff algorithm (red): character-level, word-level, and line-level diffs
- [ ] Implement `src/api/services/diff.service.ts` — compare two prompt versions
  - Output: unified diff format with additions, deletions, unchanged segments
  - Handle edge cases: empty prompts, identical prompts, completely replaced content
- [ ] `GET /api/prompts/:id/diff?from=:v1&to=:v2` — endpoint returns structured diff
- [ ] Write tests for the diff endpoint with fixture data
- [ ] Implement response comparison: `GET /api/responses/diff?a=:r1&b=:r2`
  - Compare LLM responses across versions (useful for regression tracking)

**Gate:** Diff unit tests pass with edge cases covered. Diff endpoint tested with ≥5 fixture pairs.

---

## Phase 4: Response Tracking ⬜

- [ ] Write failing tests for response storage and retrieval
- [ ] `POST /api/prompts/:id/versions/:versionId/responses` — store a response
  - Fields: `model`, `provider`, `response_text`, `latency_ms`, `token_count`, `metadata` (JSONB)
- [ ] `GET /api/prompts/:id/versions/:versionId/responses` — list responses for a version
- [ ] `GET /api/prompts/:id/responses/timeline` — all responses across all versions, time-ordered
- [ ] Aggregate stats endpoint: `GET /api/prompts/:id/stats`
  - Average latency per version, response length trends, model distribution

**Gate:** Response CRUD tests pass. Timeline query returns correct ordering across versions.

---

## Phase 5: Auth & Teams ⬜

- [ ] Write tests for auth middleware and token lifecycle (red)
- [ ] `POST /api/auth/register` — email + password, returns JWT
- [ ] `POST /api/auth/login` — returns JWT + refresh token
- [ ] `POST /api/auth/refresh` — exchange refresh token for new JWT
- [ ] JWT middleware protecting all `/api/projects/*` and `/api/prompts/*` routes
- [ ] Team invites: `POST /api/projects/:id/members` — invite user by email
- [ ] Role-based access: `owner` can delete, `editor` can commit versions, `viewer` read-only
- [ ] Write tests for RBAC enforcement on all protected routes

**Gate:** Auth tests pass. RBAC tests confirm each role boundary. No route reachable without valid JWT.

---

## Phase 6: Frontend — Core UI ⬜

- [ ] Write component tests (React Testing Library) for all components before implementing (red)
- [ ] `ProjectListPage` — list user's projects, create new
- [ ] `PromptListPage` — prompts in a project with latest version summary
- [ ] `VersionHistoryPage` — timeline of commits for a prompt (like `git log`)
- [ ] `VersionDetailPage` — view a specific version's prompt text + all its responses
- [ ] `DiffViewPage` — side-by-side or unified diff between two versions
- [ ] `CommitPage` — text editor to author a new prompt version with commit message
- [ ] API client (`src/frontend/lib/api.ts`) — typed wrapper around all backend endpoints
- [ ] Auth pages: Login, Register

**Gate:** All component tests pass. App navigable end-to-end with seed data.

---

## Phase 7: Polish & Harden ⬜

- [ ] Write Playwright e2e tests for critical paths:
  - Create project → add prompt → commit version → view diff
  - Rollback a version → verify new version created from old snapshot
  - Log in → log out → confirm protected routes redirect
- [ ] Rate limiting on auth endpoints (`express-rate-limit`)
- [ ] API response caching for expensive diff computations (Redis or in-memory LRU)
- [ ] Database indexes on hot query paths (verify with `EXPLAIN ANALYZE`)
- [ ] Audit log: record who committed what version and when
- [ ] Accessibility audit on frontend (axe-core)
- [ ] Load test the diff endpoint with `k6` or `autocannon`

**Gate:** All e2e tests pass. No critical a11y violations. Diff endpoint handles 50 req/s without degradation.

---

## Phase 8: Ship ⬜

- [ ] Write deployment docs in `docs/deploy.md` (Docker Compose + managed Postgres)
- [ ] `Dockerfile` for API + multi-stage `Dockerfile` for frontend
- [ ] `docker-compose.yml` for local full-stack bring-up
- [ ] GitHub Actions deploy workflow (build → push to GHCR → optional deploy hook)
- [ ] Populate `README.md` with real commands (replace all TODO placeholders)
- [ ] Tag release `v0.1.0`, write `CHANGELOG.md` entry
- [ ] Final manual smoke test against production-equivalent environment

**Gate:** `docker compose up` runs the full stack. All tests pass in CI. Release tagged.

---

## Parking Lot 🅿️

> Ideas to revisit once core is stable

- CLI tool (`pvh`) — commit/diff/rollback prompts from the terminal
- VS Code extension — inline version history in editor
- Webhook notifications on new version commits
- OpenAI / Anthropic direct integration to run a prompt against a model from the UI
- Import/export prompt sets as JSON or YAML
- Branching model — fork a prompt into an experimental branch

---

## Lessons Learned 📝

> Fill this in as you build. Each entry prevents the next AI agent from repeating your mistakes.

<!-- Example: "Drizzle's `.returning()` does not work with batch inserts on PostgreSQL < 15. Use individual inserts." -->
