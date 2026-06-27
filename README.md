# StudyOS

> **An AI-native Learning Operating System**
>
> StudyOS is a modular learning platform that combines AI tutoring, knowledge management, planning, analytics, and active recall into a unified learning experience.

---

# Repository Structure

```text
StudyOS/
│
├── studyos-frontend/        # Next.js Frontend (Product Engineer)
├── studyos-backend/         # FastAPI Backend (Platform Engineer) [Planned]
├── studyos-docs/            # Architecture & Project Documentation
│
└── AI_MEMORY/               # Long-term project memory
```

---

# Tech Stack

## Frontend

* Next.js (App Router)
* React
* TypeScript
* Tailwind CSS
* shadcn/ui
* Zustand
* TanStack Query
* Axios
* React Hook Form
* Zod
* Framer Motion
* Sonner

---

## Backend (Planned)

* FastAPI
* PostgreSQL
* SQLAlchemy
* Redis
* Celery
* OpenSearch
* pgvector

---

## AI Stack (Planned)

* LangGraph
* LiteLLM
* OpenAI
* Claude
* Hybrid Search (OpenSearch + pgvector)

---

# First-Time Setup

## Prerequisites

Install:

* Git
* Node.js 22+
* npm 10+

Verify:

```bash
node -v
npm -v
git --version
```

---

# Clone the Repository

```bash
git clone git@github.com:Shashquatch28/studyos.git
```

Enter the project:

```bash
cd studyos
```

---

# Frontend Setup

Navigate into the frontend:

```bash
cd studyos-frontend
```

Install all project dependencies:

```bash
npm install
```

Start the development server:

```bash
npm run dev
```

Open:

```
http://localhost:3000
```

---

# Backend Setup

> Backend setup will be added once the Platform Engineer initializes the FastAPI project.

---

# IMPORTANT

The project has **already been bootstrapped**.

**Do NOT run:**

```bash
npx create-next-app
```

or

```bash
npx shadcn@latest init
```

These steps have already been completed and committed.

Simply clone the repository and run:

```bash
npm install
```

---

# Development Workflow

Every development session should follow this workflow.

---

## 1. Pull Latest Changes

```bash
git checkout main
git pull origin main
```

---

## 2. Create a Feature Branch

Never work directly on `main`.

Use descriptive branch names.

Examples:

```bash
git checkout -b feature/auth-ui
```

```bash
git checkout -b feature/dashboard-layout
```

```bash
git checkout -b feature/knowledge-upload
```

```bash
git checkout -b fix/sidebar-overflow
```

---

## 3. Read the AI Memory

Before writing code, read:

```
AI_MEMORY/

README.md

↓

PROJECT_STATE.md

↓

NEXT_STEPS.md

↓

BLOCKERS.md

↓

CODEBASE_MAP.md (if unfamiliar)
```

This should take less than 10 minutes.

---

## 4. Implement the Feature

Follow the architecture blueprint.

Guidelines:

* Keep components small.
* Build reusable UI.
* Prefer feature-based organization.
* Do not bypass the service layer.
* Do not directly call APIs from UI components.

---

## 5. Test

Before committing:

```bash
npm run lint
```

```bash
npm run dev
```

Verify:

* No build errors
* No lint errors
* UI behaves correctly
* Responsive layout
* No console errors

---

## 6. Commit

Use Conventional Commits.

Examples:

```
feat(auth): add login page

feat(notes): implement rich text editor

fix(upload): resolve drag-and-drop bug

refactor(layout): simplify dashboard shell

docs(memory): update project state

chore(frontend): configure providers
```

Keep commits focused.

Avoid giant commits covering multiple unrelated changes.

---

## 7. Push

```bash
git push origin feature/<branch-name>
```

---

## 8. Open Pull Request

Before merging:

* Review your changes.
* Ensure documentation is updated.
* Resolve merge conflicts.
* Wait for approval if required.

---

# AI Memory Workflow

Every developer is responsible for maintaining the project memory.

Whenever significant work is completed, update:

```
PROJECT_STATE.md

↓

NEXT_STEPS.md

↓

COMPLETED_WORK.md

↓

Relevant notes file
```

If a major engineering decision is made:

Update:

```
DECISIONS_LOG.md
```

If something blocks progress:

Update:

```
BLOCKERS.md
```

If an important lesson is learned:

Update:

```
MISTAKES.md
```

---

# Coding Standards

## Frontend

* TypeScript only
* Functional components
* Hooks over classes
* Feature-based architecture
* Shared UI inside `components/`
* Feature logic inside `features/`

---

## Git

* Never commit directly to `main`.
* Keep commits atomic.
* Use descriptive commit messages.
* Pull before starting work.
* Resolve conflicts before opening PRs.

---

## Documentation

Whenever architecture changes:

Update:

* CODEBASE_MAP.md
* DECISIONS_LOG.md

Documentation should evolve alongside the codebase.

---

# Current Project Status

Current phase:

```
Frontend Foundation
```

Current objective:

```
Establish production-ready frontend architecture.
```

After completion:

```
Authentication

↓

Dashboard

↓

Knowledge Hub

↓

AI Tutor

↓

Learning Tools

↓

Planner

↓

Analytics
```

---

# Contributing Guidelines

Every contribution should follow this checklist:

* [ ] Pull latest changes
* [ ] Create a feature branch
* [ ] Read the AI Memory
* [ ] Implement the feature
* [ ] Test locally
* [ ] Update documentation
* [ ] Update AI Memory (if applicable)
* [ ] Commit using Conventional Commits
* [ ] Push branch
* [ ] Open Pull Request

---

# Repository Principles

* Build incrementally.
* Prefer clarity over cleverness.
* Keep modules independent.
* Reuse components whenever possible.
* Document important decisions.
* Keep the AI Memory System up to date.
* Treat architecture as a first-class artifact.

---

# Need Help?

If you're unsure where to begin:

1. Read the AI Memory System.
2. Read the architecture documentation.
3. Check `NEXT_STEPS.md`.
4. Continue from the current milestone.

The goal is that any developer or AI assistant should be able to onboard and contribute with minimal context switching.
