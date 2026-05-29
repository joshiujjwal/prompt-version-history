# Feature Specification — prompt-version-history

**Version:** 0.1  
**Status:** Draft  
**Last updated:** 2025

---

## 1. Overview

### Problem Statement

Teams building LLM-powered features iterate on prompts constantly, but have no structured way to:
- Know which prompt version is running in production
- Compare how a model responds to v1 vs v4 of a prompt
- Roll back to a previous prompt when a new version regresses quality
- Collaborate on prompt changes with review/approval workflows

### Solution

A web application modeled on Git semantics (commit, history, diff, rollback) applied to prompt management. Every change to a prompt creates an immutable version snapshot. Responses from LLMs can be stored against the version that generated them, enabling longitudinal quality tracking.

---

## 2. Functional Requirements

### 2.1 Prompt Versioning
- [ ] Users can create **projects** to group related prompts
- [ ] Each project can contain multiple **prompts** (e.g., "summarization prompt", "extraction prompt")
- [ ] Every change to a prompt text creates a new **version** with an auto-incrementing number and a required commit message
- [ ] Versions are immutable — existing versions can never be edited, only new ones created
- [ ] Users can **roll back** to any prior version by creating a new version whose content copies the target snapshot

### 2.2 Diff & Comparison
- [ ] Users can select any two versions of a prompt and view a structured diff
- [ ] Diff supports three modes: character-level, word-level, line-level
- [ ] Diff UI shows additions in green, deletions in red, unchanged in grey
- [ ] Users can compare **responses** from two different versions side-by-side

### 2.3 Response Tracking
- [ ] Users can attach LLM responses to a specific prompt version
- [ ] Each response stores: `model`, `provider`, `response_text`, `latency_ms`, `token_count`, and arbitrary `metadata`
- [ ] Timeline view shows all responses across all versions of a prompt, ordered by time
- [ ] Stats view shows: average latency per version, response length trends, model usage distribution

### 2.4 Teams & Collaboration
- [ ] Projects are owned by a user
- [ ] Project owners can invite collaborators by email
- [ ] Three roles: `owner` (full control), `editor` (commit versions, add responses), `viewer` (read-only)
- [ ] All version commits are attributed to the committing user

### 2.5 Tagging
- [ ] Users can apply named tags to any version (e.g., `v1.0`, `production`, `approved`)
- [ ] Tags are mutable — can be moved to a different version (like Git tags)

---

## 3. Non-Functional Requirements

- [ ] API response time < 200ms at p99 for read operations on projects with < 1000 versions
- [ ] Diff computation < 500ms for prompts up to 10,000 characters
- [ ] Frontend initial load < 2s on a 4G connection
- [ ] All API endpoints authenticated — no anonymous access to data
- [ ] PostgreSQL as the sole data store (no external services required for core features)
- [ ] Full TypeScript coverage — `strict: true`, no `any` in production code

---

## 4. Data Model

### `users`
```sql
id          UUID PRIMARY KEY DEFAULT gen_random_uuid()
email       TEXT UNIQUE NOT NULL
password_hash TEXT NOT NULL
display_name TEXT
created_at  TIMESTAMPTZ DEFAULT now()
```

### `projects`
```sql
id          UUID PRIMARY KEY DEFAULT gen_random_uuid()
owner_id    UUID REFERENCES users(id) ON DELETE CASCADE
name        TEXT NOT NULL
description TEXT
created_at  TIMESTAMPTZ DEFAULT now()
updated_at  TIMESTAMPTZ DEFAULT now()
```

### `project_members`
```sql
project_id  UUID REFERENCES projects(id) ON DELETE CASCADE
user_id     UUID REFERENCES users(id) ON DELETE CASCADE
role        TEXT CHECK (role IN ('owner', 'editor', 'viewer')) NOT NULL
joined_at   TIMESTAMPTZ DEFAULT now()
PRIMARY KEY (project_id, user_id)
```

### `prompts`
```sql
id          UUID PRIMARY KEY DEFAULT gen_random_uuid()
project_id  UUID REFERENCES projects(id) ON DELETE CASCADE
name        TEXT NOT NULL
description TEXT
created_by  UUID REFERENCES users(id)
created_at  TIMESTAMPTZ DEFAULT now()
```

### `prompt_versions`
```sql
id              UUID PRIMARY KEY DEFAULT gen_random_uuid()
prompt_id       UUID REFERENCES prompts(id) ON DELETE CASCADE
version_number  INTEGER NOT NULL
content         TEXT NOT NULL
commit_message  TEXT NOT NULL
committed_by    UUID REFERENCES users(id)
committed_at    TIMESTAMPTZ DEFAULT now()
parent_version_id UUID REFERENCES prompt_versions(id)  -- null for v1
UNIQUE (prompt_id, version_number)
```

### `responses`
```sql
id              UUID PRIMARY KEY DEFAULT gen_random_uuid()
version_id      UUID REFERENCES prompt_versions(id) ON DELETE CASCADE
model           TEXT NOT NULL          -- e.g., "gpt-4o"
provider        TEXT NOT NULL          -- e.g., "openai"
response_text   TEXT NOT NULL
latency_ms      INTEGER
token_count     INTEGER
metadata        JSONB                  -- arbitrary key-value, e.g., temperature, top_p
created_by      UUID REFERENCES users(id)
created_at      TIMESTAMPTZ DEFAULT now()
```

### `tags`
```sql
id          UUID PRIMARY KEY DEFAULT gen_random_uuid()
prompt_id   UUID REFERENCES prompts(id) ON DELETE CASCADE
version_id  UUID REFERENCES prompt_versions(id) ON DELETE CASCADE
name        TEXT NOT NULL
created_by  UUID REFERENCES users(id)
created_at  TIMESTAMPTZ DEFAULT now()
UNIQUE (prompt_id, name)
```

---

## 5. API Design

### Base URL: `/api`

All endpoints require `Authorization: Bearer <token>` except `/auth/*`.

#### Auth
| Method | Path | Description |
|--------|------|-------------|
| POST | `/auth/register` | Create account |
| POST | `/auth/login` | Get JWT + refresh token |
| POST | `/auth/refresh` | Rotate JWT |

#### Projects
| Method | Path | Description |
|--------|------|-------------|
| POST | `/projects` | Create project |
| GET | `/projects` | List user's projects |
| GET | `/projects/:id` | Get project detail |
| DELETE | `/projects/:id` | Delete project (owner only) |
| POST | `/projects/:id/members` | Invite member |
| DELETE | `/projects/:id/members/:userId` | Remove member |

#### Prompts & Versions
| Method | Path | Description |
|--------|------|-------------|
| POST | `/projects/:id/prompts` | Create prompt |
| GET | `/projects/:id/prompts` | List prompts in project |
| GET | `/prompts/:id` | Get prompt detail |
| POST | `/prompts/:id/versions` | Commit new version |
| GET | `/prompts/:id/versions` | List version history |
| GET | `/prompts/:id/versions/:versionId` | Get specific version |
| POST | `/prompts/:id/versions/:versionId/rollback` | Rollback (creates new version) |
| GET | `/prompts/:id/diff?from=v1&to=v2` | Diff two versions |
| POST | `/prompts/:id/tags` | Create/move tag |

#### Responses
| Method | Path | Description |
|--------|------|-------------|
| POST | `/prompts/:id/versions/:versionId/responses` | Store response |
| GET | `/prompts/:id/versions/:versionId/responses` | List responses for version |
| GET | `/prompts/:id/responses/timeline` | All responses across versions |
| GET | `/prompts/:id/stats` | Aggregate stats |
| GET | `/responses/diff?a=r1&b=r2` | Compare two responses |

---

## 6. Frontend Routes

| Route | Component | Description |
|-------|-----------|-------------|
| `/` | `DashboardPage` | Project list |
| `/projects/new` | `NewProjectPage` | Create project form |
| `/projects/:id` | `PromptListPage` | Prompts in project |
| `/prompts/:id` | `VersionHistoryPage` | Git log style history |
| `/prompts/:id/versions/:versionId` | `VersionDetailPage` | Version + responses |
| `/prompts/:id/commit` | `CommitPage` | Author new version |
| `/prompts/:id/diff` | `DiffViewPage` | Compare two versions |
| `/login` | `LoginPage` | Auth |
| `/register` | `RegisterPage` | Auth |

---

## 7. Test Plan

### Unit Tests
- `diff.service.ts`: ≥15 cases covering empty, identical, partial, total replacement, multi-line, unicode
- Zod validation schemas: valid + invalid inputs for every route
- JWT utility: sign, verify, expiry, malformed token
- Role checking utility: all 3 roles × all permission types

### Integration Tests (Supertest)
- All CRUD routes: happy path + 401 unauthorized + 403 forbidden + 404 not found + 422 validation error
- Version immutability: PUT to a version ID returns 405
- Rollback: new version number increments, content matches target, parent_version_id set correctly
- Diff endpoint: verifies structured output shape for known fixture pairs

### E2E Tests (Playwright)
- **Critical path 1:** Register → create project → create prompt → commit v1 → commit v2 → view diff
- **Critical path 2:** Rollback v1 → verify new v3 has v1 content → check version list shows 3 entries
- **Critical path 3:** Invite collaborator → log in as collaborator → confirm editor can commit, viewer cannot

---

## 8. Open Questions

1. **Branching:** Should prompts support named branches (like `git branch`)? Scoped to parking lot for now — flat linear history is simpler to start.
2. **LLM integration:** Should the app call OpenAI/Anthropic directly to run a prompt and auto-store the response? Deferred — adds API key management complexity.
3. **Export format:** What's the canonical export format for prompt sets? JSON and YAML both viable — decide before Phase 8.
4. **Conflict resolution:** If two editors commit to the same prompt simultaneously, last-write-wins is the simplest model. Is optimistic locking needed?
5. **Soft delete:** Should deleted projects/prompts be soft-deleted with a `deleted_at` column, or hard-deleted? Recommend soft delete for audit trail.
