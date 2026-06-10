---
description: "FastReact development guide - Build your SaaS application with FastAPI and React Router v7. Learn workflows, testing, debugging, and best practices for fullstack development."
keywords: "fastreact development, fastapi development, react router v7 development, saas development, fullstack workflow, testing fastapi, debugging react, hot reload, api development"
---

# Development Guide

This guide shows you how to build your own SaaS application on top of [FastReact](https://fastreact.dev).

## Building Your Application

FastReact includes an AI Note Improver demo to showcase the platform's capabilities. Here's how to replace it with your own application:

### Step 1: Plan Your Application

Define your core business entities and features:

- What is your main domain model? (e.g., Projects, Tasks, Documents, etc.)
- What actions can users perform? (Create, edit, share, export, etc.)
- What billing model will you use? (Per-seat, usage-based, tiered, etc.)

### Step 2: Implement Your Features

Follow our comprehensive tutorial to add new entities and features to FastReact:

**[Adding a New Entity (End-to-End Tutorial)](tutorials.md#adding-a-new-entity-end-to-end)**

This tutorial walks you through building a complete "Projects" feature with:

- Database schema design with Sqitch migrations
- Backend implementation (models, repositories, services, API routes)
- Dependency injection setup
- Frontend components and API client generation
- Complete working code you can adapt for your own features

The tutorial demonstrates FastReact's full-stack architecture pattern that you can apply to any domain entity.

### Step 3: Remove Demo Code

Once you've implemented your own features, you can remove the AI Note Improver demo:

**Remove backend files:**

```bash
cd backend
rm app/model/note_model.py
rm app/service/note_service.py
rm app/service/note_organizer_service.py
rm app/api/route/note_route.py
rm app/data/repo/note_repo.py
```

**Remove frontend files:**

```bash
cd frontend
rm app/routes/protected/notes.tsx
rm app/routes/protected/notes-edit.tsx
```

**Remove the note table from the database:**

```bash
cd db
sqitch add remove_note_table -n "Remove note demo table"
```

Edit `deploy/remove_note_table.sql`:

```sql
-- Deploy fastreact:remove_note_table to pg

BEGIN;

DROP TABLE IF EXISTS fastreact.note CASCADE;

COMMIT;
```

Edit `revert/remove_note_table.sql` with the original CREATE TABLE statement from `deploy/001_schema.sql`, then deploy:

```bash
./sqitch.sh dev deploy
```

**Optional: Remove OpenAI dependency** (if you're not using AI features):

```bash
cd backend
uv remove openai
```

---

## Common Development Workflows

When making changes, follow the DB → Backend → Frontend flow:

### Database Changes

```bash
cd db
sqitch add feature_name -n "Description"
# Edit deploy/, revert/, verify/ SQL files
./sqitch.sh dev deploy
```

### Backend Changes

```bash
cd backend

# Add or remove dependencies
uv add package_name              # Add to production
uv add --dev package_name        # Add to development

# Run linter
uv run ruff check .

# Run tests
uv run pytest

# Start server (auto-reloads on code changes)
uv run uvicorn app.main:app --reload
```

### Frontend Changes

After backend API changes, regenerate the API client:

```bash
cd frontend

# Generate TypeScript client (backend must be running)
npm run generate

# Run linter and formatter
npm run format    # Prettier
npm run typecheck # Type checking

# Run linter
npm run lint

# Start dev server (hot-reloads on code changes)
npm run dev
```

---

## Keeping Dependencies Up-to-Date

Regular dependency upgrades keep your application secure and up-to-date. FastReact provides a structured upgrade process with automated tests to verify compatibility.

### Upgrade Policy

FastReact ships a Dependabot config (`.github/dependabot.yml`) so routine upgrades are automated and reviewable:

- **Automated, weekly:** Dependabot opens **grouped** PRs once a week per ecosystem — `npm` (frontend), `npm` (landing), and `uv` (backend). The backend uses a 7-day cooldown so brand-new releases settle before they're proposed.
- **Minor & patch upgrades:** handled by those weekly PRs. Review and merge once CI passes.
- **Major versions:** **excluded** from Dependabot (`version-update:semver-major` is ignored) and done deliberately, one component at a time, since they may require code changes.
- **Security updates:** fast-track immediately, outside the weekly cadence.
- **Version pinning:** `pyproject.toml` and `package.json` declare lower-bound (`>=`) ranges; the lockfiles (`uv.lock`, `package-lock.json`) pin exact versions and are committed — so installs are reproducible while ranges stay flexible.

### Backend Dependencies (Python)

FastReact uses `uv` to manage Python dependencies.

**Upgrade workflow:**

```bash
cd backend

# 1. Upgrade all dependencies
uv sync --upgrade

# Or upgrade a specific package
uv add --upgrade package_name

# 2. Run tests to verify compatibility
uv run pytest
```

**Note:** Smoke tests in `backend/test/smoke/` verify app startup, database connectivity, and API functionality. Database tests require PostgreSQL running (`docker compose up db -d`) but will skip gracefully if unavailable.

### Frontend Dependencies (npm)

The frontend and landing page use `package.json` and `package-lock.json` for dependency management.

**Upgrade workflow:**

```bash
# Frontend
cd frontend

# 1. Check what's outdated
npm outdated

# 2. Update dependencies (respects semver ranges)
npm update

# 3. Run checks to verify compatibility
npm run build
npm run typecheck
npm run lint

# Landing page (simpler, no tests by default)
cd ../landing
npm outdated
npm update
npm run build
```

**For major version upgrades:**

```bash
# Install npm-check-updates (one-time)
npm install -g npm-check-updates

# Frontend or landing - check what would be updated
cd frontend  # or cd landing
ncu

# Update package.json to latest versions
ncu -u

# Install and test
npm install
npm run build && npm run typecheck && npm run lint
```

**Important:**

- Commit `package-lock.json` after upgrades
- Test thoroughly after major version updates
- Update one component at a time (backend, frontend, landing) for easier troubleshooting

---

## Next Steps

- **[Tutorials](tutorials.md)** - Step-by-step guides for common tasks
- **[Architecture Overview](architecture-overview.md)** - Understand the codebase structure
- **[Troubleshooting](troubleshooting.md)** - Common issues and solutions
- **[Deployment](deployment/index.md)** - Deploy to production
