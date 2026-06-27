# NEXT_STEPS

> Active development roadmap for StudyOS.
>
> **Purpose**
>
> This file answers:
>
> **"What should we work on next?"**
>
> Unlike `COMPLETED_WORK.md`, this document is a living task board and should be updated continuously.
>
> **Rules**
>
> - Mark completed tasks with ✅ before moving them to `COMPLETED_WORK.md`.
> - Add new work as soon as it is identified.
> - Keep tasks implementation-focused.
> - Link related GitHub Issues or PRs when available.
> - Review this file at the beginning and end of every development session.

---

# Current Milestone

## Phase 0 — Frontend Foundation

**Status**

🟡 In Progress

**Goal**

Establish a production-ready frontend architecture before implementing any product features.

---

# Current Sprint

## Sprint Objective

Build the frontend foundation that every future module will rely on.

Target outcomes:

- Scalable folder architecture
- Shared design system
- Global providers
- API layer
- State management
- Development tooling
- Theme system

---

# 🔴 In Progress

## Frontend Architecture

Priority: High

Tasks

- [ ] Create hybrid folder architecture
- [ ] Create feature directories
- [ ] Create shared component directories
- [ ] Configure project constants
- [ ] Configure environment variables
- [ ] Configure API services layer
- [ ] Configure Zustand stores
- [ ] Configure TanStack Query provider
- [ ] Configure Theme Provider
- [ ] Configure Sonner
- [ ] Configure Axios instance
- [ ] Create utility helpers
- [ ] Create shared hooks

Owner

Product Engineer

---

# 🟠 Up Next

## Design System

Priority: High

Tasks

- [ ] Build Button variants
- [ ] Build Input components
- [ ] Build Cards
- [ ] Build Modal/Dialog wrappers
- [ ] Build Empty State
- [ ] Build Loading State
- [ ] Build Error State
- [ ] Build Page Container
- [ ] Build Section component
- [ ] Build Typography system

---

## Application Shell

Priority: High

Tasks

- [ ] Dashboard layout
- [ ] Sidebar
- [ ] Navbar
- [ ] Breadcrumbs
- [ ] Mobile navigation
- [ ] Theme toggle
- [ ] Global search placeholder

---

# 🟡 Product Modules

Development order follows the architecture blueprint.

---

## Authentication

Status

Not Started

Tasks

- [ ] Login page
- [ ] Registration page
- [ ] Forgot password
- [ ] Reset password
- [ ] Session handling
- [ ] Route protection

---

## Dashboard

Status

Not Started

Tasks

- [ ] Dashboard overview
- [ ] Progress widgets
- [ ] Continue learning
- [ ] Recent activity
- [ ] Quick actions

---

## Knowledge Hub

Status

Not Started

Tasks

- [ ] Upload resources
- [ ] Resource library
- [ ] Resource viewer
- [ ] Metadata display
- [ ] Search
- [ ] Filters

---

## AI Tutor

Status

Not Started

Tasks

- [ ] Tutor chat UI
- [ ] Context panel
- [ ] Suggested questions
- [ ] Citation rendering
- [ ] Conversation history

---

## AI Notes

Status

Not Started

Tasks

- [ ] Notes viewer
- [ ] Rich editor
- [ ] Export
- [ ] Auto-generation UI

---

## Flashcards

Status

Not Started

Tasks

- [ ] Flashcard viewer
- [ ] Study mode
- [ ] Progress tracking
- [ ] Review scheduling UI

---

## Quizzes

Status

Not Started

Tasks

- [ ] Quiz interface
- [ ] Timer
- [ ] Results page
- [ ] Performance analytics

---

## Planner

Status

Not Started

Tasks

- [ ] Study planner
- [ ] Session creation
- [ ] Goals
- [ ] Calendar integration

---

## Analytics

Status

Not Started

Tasks

- [ ] Dashboard analytics
- [ ] Learning trends
- [ ] Study heatmap
- [ ] Performance graphs

---

# 🔵 Backend Dependencies

Waiting on Platform Engineer

- [ ] Authentication API
- [ ] User API
- [ ] Resource API
- [ ] Chat API
- [ ] Notes API
- [ ] Flashcards API
- [ ] Quiz API
- [ ] Planner API

Frontend should initially use mocked data until APIs become available.

---

# 🟣 Infrastructure Dependencies

Waiting on Platform Engineer

- [ ] FastAPI project
- [ ] PostgreSQL
- [ ] Redis
- [ ] Docker
- [ ] Celery
- [ ] OpenSearch
- [ ] pgvector

---

# ⚪ Technical Debt

Current

None.

Future items should be tracked here instead of leaving TODO comments throughout the codebase.

Example

- Replace temporary mock API
- Optimize bundle size
- Lazy load dashboard widgets

---

# 📚 Documentation Tasks

- [ ] Update CODEBASE_MAP after architecture changes
- [ ] Update PROJECT_STATE after each milestone
- [ ] Record architectural decisions
- [ ] Document backend APIs
- [ ] Maintain AI notes

---

# 🚀 Future Roadmap

## Phase 1

Foundation

## Phase 2

Core Learning Experience

- Knowledge Hub
- AI Tutor

## Phase 3

Learning Tools

- Notes
- Flashcards
- Quizzes

## Phase 4

Planning

- Planner
- Calendar
- Goals

## Phase 5

Analytics

- Learning Insights
- Knowledge Graph
- Recommendations

---

# Session Checklist

## Before Starting

- [ ] Read PROJECT_STATE.md
- [ ] Read BLOCKERS.md
- [ ] Read DECISIONS_LOG.md
- [ ] Select tasks from this document

---

## Before Ending

- [ ] Mark completed tasks
- [ ] Update PROJECT_STATE.md
- [ ] Move completed work to COMPLETED_WORK.md
- [ ] Record new architectural decisions
- [ ] Update CODEBASE_MAP if required

