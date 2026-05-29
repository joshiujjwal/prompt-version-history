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
