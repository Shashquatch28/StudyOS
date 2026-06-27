# StudyOS — Platform Engineer README
> Owner: Person A (Platform / Backend / Infrastructure)
>
> Your responsibility is to build and maintain the foundation that every other module depends on.
> Think like a Platform Engineer, not just a Backend Developer.
>
> Your goal is to make it possible for the Product Engineer to build features without worrying about infrastructure.

---

# Philosophy

You own the platform.

Your job is NOT to build every feature.

Your job is to build systems that make every feature possible.

Whenever possible:

- Build generic solutions.
- Avoid feature-specific implementations.
- Expose clean APIs.
- Keep modules independent.

---

# Core Responsibilities

You own:

- Project Architecture
- Infrastructure
- Authentication
- Database
- Backend APIs
- AI Infrastructure
- Search
- RAG
- Upload Pipeline
- Background Workers
- WebSockets
- DevOps
- Shared Backend Components

You DO NOT own feature UIs.

---

# Folder Ownership

## Backend

```
studyos-backend/
```

Everything inside this folder is yours.

Exception:

If frontend requires API changes,
coordinate before modifying contracts.

---

## Infrastructure

```
docker/
docker-compose.yml
.github/
studyos-infrastructure/
```

100% yours.

---

## Shared Documentation

```
studyos-docs/
```

Both developers contribute.

---

# Overall Roadmap

```
Foundation
↓

Authentication

↓

Database

↓

Infrastructure

↓

Knowledge Pipeline

↓

Search

↓

Tutor Backend

↓

AI Infrastructure

↓

Learning Engines

↓

Planner Backend

↓

Optimization
```

---

# Phase 0 — Foundation

Goal:

Get the project runnable.

---

## Checklist

### Repository

- [ ] Clone repository
- [ ] Setup backend folder
- [ ] Configure Poetry
- [ ] Configure Ruff
- [ ] Configure MyPy
- [ ] Configure pre-commit
- [ ] Docker setup

---

### FastAPI

- [ ] App factory
- [ ] Settings
- [ ] Dependency Injection
- [ ] Logging
- [ ] Exception Handlers
- [ ] Middleware

---

### Database

- [ ] PostgreSQL
- [ ] SQLAlchemy
- [ ] Alembic
- [ ] Base Models
- [ ] UUID Support

---

### Redis

- [ ] Redis connection
- [ ] Async client
- [ ] Cache helpers

---

### Celery

- [ ] Worker
- [ ] Beat
- [ ] Queues

---

### Docker

- [ ] API Container
- [ ] Worker Container
- [ ] Redis
- [ ] PostgreSQL
- [ ] OpenSearch

---

### CI

- [ ] Github Actions
- [ ] Ruff
- [ ] MyPy
- [ ] Tests

---

# Deliverable

Entire backend boots using

```
docker compose up
```

---

# Phase 1 — Authentication

## Checklist

### User

- [ ] User Model
- [ ] User Repository
- [ ] User Service

---

### Authentication

- [ ] JWT
- [ ] Refresh Tokens
- [ ] Google OAuth
- [ ] Password Auth

---

### APIs

- [ ] Register
- [ ] Login
- [ ] Refresh
- [ ] Logout
- [ ] Verify Email

---

### Security

- [ ] Password Hashing
- [ ] Middleware
- [ ] RBAC

---

# Deliverable

Authentication is production-ready.

---

# Phase 2 — Database

Implement every table.

Priority

```
Users

↓

Resources

↓

Chunks

↓

Embeddings

↓

Conversations

↓

Messages

↓

Notes

↓

Flashcards

↓

Planner

↓

Analytics
```

---

## Checklist

- [ ] Models
- [ ] Repositories
- [ ] Alembic
- [ ] Indexes
- [ ] Constraints

---

# Deliverable

Entire schema migrated.

---

# Phase 3 — Knowledge Pipeline

## Upload

- [ ] Presigned URLs
- [ ] Validation
- [ ] Virus Scan

---

## Extraction

- [ ] PDF
- [ ] DOCX
- [ ] PPTX
- [ ] OCR
- [ ] Whisper
- [ ] Crawl4AI

---

## Processing

- [ ] Cleaning
- [ ] Metadata
- [ ] Chunking
- [ ] Embeddings
- [ ] OpenSearch

---

## Background Jobs

- [ ] Celery Queue
- [ ] Retry Logic
- [ ] Failure Recovery

---

# Deliverable

Uploaded files become searchable.

---

# Phase 4 — Search

## Checklist

- [ ] pgvector
- [ ] OpenSearch
- [ ] BM25
- [ ] Hybrid Retrieval
- [ ] Re-ranking
- [ ] Search API

---

# Deliverable

Hybrid Search operational.

---

# Phase 5 — AI Infrastructure

## LiteLLM

- [ ] Routing
- [ ] Fallbacks
- [ ] Caching

---

## LangGraph

- [ ] Tutor Graph
- [ ] Planner Graph
- [ ] Knowledge Agent

---

## Memory

- [ ] Short Term
- [ ] Long Term
- [ ] Episodic

---

## WebSockets

- [ ] Streaming
- [ ] Token Publishing

---

# Deliverable

AI backend complete.

---

# Phase 6 — Learning Engines

Implement backend for

- [ ] Notes
- [ ] Flashcards
- [ ] Quiz
- [ ] Planner
- [ ] Analytics
- [ ] Knowledge Graph

No frontend.

---

# Backend API Checklist

Every feature requires

- [ ] Service
- [ ] Repository
- [ ] Schemas
- [ ] API
- [ ] Tests
- [ ] Documentation

Never skip.

---

# Code Ownership

You own

```
Database

Authentication

Infrastructure

Redis

Celery

Search

Workers

Repositories

Services

AI

Security

WebSockets
```

---

# Communication Points

These require discussion BEFORE implementation.

## 1.

Authentication Flow

Need agreement because frontend depends on it.

---

## 2.

API Contracts

Every endpoint must be finalized together.

Never change after frontend starts.

---

## 3.

Database Changes

Changing

- IDs
- Relationships
- Names

requires discussion.

Adding tables usually doesn't.

---

## 4.

WebSocket Messages

Frontend consumes these.

Never modify format without agreement.

---

## 5.

Shared Types

Response JSON

Status enums

Constants

Need synchronization.

---

# Areas With Full Freedom

You DO NOT need permission for

- Internal architecture
- Repository pattern
- Service pattern
- Celery structure
- Logging
- Folder organization
- Dependency Injection
- Internal AI orchestration
- Background jobs
- Testing
- Docker
- Monitoring
- Performance improvements

As long as APIs remain unchanged.

---

# Before Starting Any Module

Complete

```
Backend Design

↓

Database

↓

Repository

↓

Service

↓

API

↓

Tests

↓

Documentation
```

Only then begin implementation.

---

# Pull Request Checklist

- [ ] Tests pass
- [ ] Ruff passes
- [ ] MyPy passes
- [ ] API documented
- [ ] Migration included
- [ ] No breaking API changes
- [ ] Changelog updated

---

# Daily Workflow

Morning

- Pull latest
- Review issues
- Pick one feature

During Work

- Commit often
- Push regularly
- Keep branches small

Evening

- Open PR
- Update docs
- Review teammate's PR

---

# Documentation Requirements

Every completed module updates

```
decisions.md

next_steps.md

mistakes.md

implementation_notes.md

technical_debt.md

CHANGELOG.md
```

---

# Branch Naming

```
feature/auth

feature/upload

feature/rag

feature/search

feature/langgraph

feature/embeddings

feature/planner

fix/auth

fix/search

refactor/database
```

---

# Definition of Done

A backend feature is complete only if

- Database exists
- APIs work
- Tests pass
- Swagger updated
- Documentation updated
- Frontend can consume it without backend modifications
- CI passes

If any of these are missing,

**the feature is not complete.**