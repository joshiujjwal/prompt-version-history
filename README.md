# prompt-version-history

> 🚧 **Status: Early Development**

Git-like version control system for AI prompts and their responses — enabling teams to track prompt evolution, compare outputs across versions, and roll back to previous prompt states.

---

## Why This Exists

Prompt engineering is iterative and collaborative, but most teams track prompt changes in Notion docs, Slack threads, or not at all. `prompt-version-history` brings software engineering discipline to prompt management: commit, diff, branch, and roll back prompts the same way you manage code.

---

## Tech Stack

| Layer      | Technology                        |
|------------|-----------------------------------|
| Frontend   | React 18 + TypeScript + Vite      |
| Backend    | Node.js + Express + TypeScript    |
| Database   | PostgreSQL 15                     |
| ORM        | Drizzle ORM                       |
| Auth       | JWT + bcrypt                      |
| Testing    | Vitest (unit) + Supertest (API)   |
| CI         | GitHub Actions                    |

---

## Getting Started

```bash
# Clone
git clone https://github.com/joshiujjwal/prompt-version-history.git
cd prompt-version-history

# Install dependencies
npm install

# Configure environment
cp .env.example .env
# Edit .env with your PostgreSQL credentials and JWT secret

# Run database migrations
npm run db:migrate

# Start development servers (API + frontend)
npm run dev

# Run tests
npm test
```

---

## Project Structure

```
prompt-version-history/
├── src/
│   ├── api/                  # Express backend
│   │   ├── routes/           # Route handlers
│   │   ├── middleware/       # Auth, error handling, validation
│   │   ├── services/         # Business logic
│   │   └── db/               # Drizzle schema + migrations
│   ├── frontend/             # React SPA
│   │   ├── components/       # Reusable UI components
│   │   ├── hooks/            # Custom React hooks
│   │   ├── pages/            # Route-level page components
│   │   ├── lib/              # API client, utilities
│   │   └── types/            # Frontend-specific types
│   └── shared/
│       └── types/            # Types shared between API and frontend
├── tests/
│   ├── unit/                 # Vitest unit tests
│   ├── integration/          # API integration tests (Supertest)
│   └── e2e/                  # End-to-end tests (Playwright)
├── docs/
│   ├── spec.md               # Feature specification
│   └── adr/                  # Architecture Decision Records
├── .github/
│   ├── copilot-instructions.md
│   └── instructions/
├── CLAUDE.md                 # Claude agent context
├── AGENTS.md                 # Codex agent config
└── TODO.md                   # Evidence-gated task breakdown
```

---

## Contributing

1. **Tests first** — write failing tests before implementing anything
2. **Small PRs** — one feature or fix per PR, under 400 lines diff
3. **Evidence required** — PR descriptions must include: test output, manual verification steps, and before/after comparison for UI changes
4. **No unreviewed merges** — all PRs require at least one approval
5. **Update context files** — if you learn something non-obvious about the codebase, add it to `CLAUDE.md`

---

## License

MIT

---

## 🚀 Improvement Proposals

### First-Principles Analysis
- **The core insight is correct**: prompts are software artefacts that evolve over time, and current tooling treats them as disposable strings rather than versioned assets — applying version control discipline to prompts is a genuine unmet need in production LLM workflows.
- **"Git for prompts" is an analogy that breaks down at the diff level**: code diffs are syntactically meaningful (a changed variable name is unambiguous); prompt diffs can have semantic impact invisible to line-level diffing (moving a sentence changes the model's attention distribution) — the diff viewer must be semantically aware, not just textual.
- **The value of version history is proportional to the quality of the response history stored alongside it** — if only prompts are versioned but not their outputs across different model versions and configurations, the system is a changelog, not an experiment tracker.
- **Branches in git represent parallel development workflows; in prompt engineering, branches represent parallel hypothesis tests** — the branching UX should reflect this: each branch should show its win-rate or quality score relative to the trunk, not just the text changes.

### Key Risks & Assumptions
- **Assumes teams are already doing prompt engineering collaboratively** — many teams have one person who owns prompts and treats them as config files; the collaboration features (branches, PRs for prompts) require a team workflow that doesn't yet exist and must be sold, not just built.
- **JWT + bcrypt auth for a team collaboration tool means building user management from scratch** — consider Clerk or Auth.js to eliminate this undifferentiated work and ship the core versioning features faster.
- **PostgreSQL is the right database for structured metadata but a poor store for large response payloads** — storing full LLM outputs (potentially 4K+ tokens each) in Postgres will cause table bloat; responses should be stored in object storage (S3/R2) with metadata in Postgres.
- **No mention of LLM integration for running prompts directly** — without the ability to execute a versioned prompt and capture the response in-product, the tool is a passive registry rather than an active experiment platform.

### Concrete Improvement Ideas
1. **Add in-product prompt execution with response capture** — integrate OpenAI/Anthropic APIs so users can run any versioned prompt directly from the UI and have the response automatically linked to that version; this transforms the tool from a changelog into an experiment platform (highest impact).
2. **Build a semantic diff renderer** — go beyond line-by-line diff; highlight sentences/phrases that were added, removed, or reordered, and annotate diffs that cross structural boundaries (system prompt vs user message vs examples); this makes prompt evolution meaningful.
3. **Add a version comparison runner** — select two prompt versions and run them both against the same input; display side-by-side outputs with a rating widget; build a win-rate metric per branch over time.
4. **Implement prompt templates with variable slots** — let users define `{{variable}}` placeholders in versioned prompts; the version history then tracks changes to the template structure separately from runtime variable substitution; this matches how production prompts actually work.
5. **Add team-level analytics dashboard** — show which prompt versions are in production, how many executions each version has received, average latency, and cost per version; this turns the tool into a prompt observability platform, not just a history viewer.
6. **Export to common formats** — support exporting a prompt version as an OpenAI messages array JSON, a LangChain PromptTemplate, or an Anthropic system/human/assistant block; meet users where their production code lives.
