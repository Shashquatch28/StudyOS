# DECISIONS_LOG

> Permanent record of all important engineering and architectural decisions.
>
> **Purpose**
>
> This file answers:
>
> **"Why was something built this way?"**
>
> Every major technical decision should be documented here so future developers never have to rediscover the reasoning.
>
> **Rules**
>
> - Never delete entries.
> - Never modify old decisions unless they are factually incorrect.
> - If a decision changes, create a new entry referencing the previous one.
> - Entries are append-only.

---

# Decision Template

## ADR-XXX

**Date**

YYYY-MM-DD

**Status**

Proposed | Accepted | Superseded | Deprecated

**Decision**

Brief description.

**Context**

Why this decision was required.

**Alternatives Considered**

- Option A
- Option B
- Option C

**Chosen Solution**

Explain what was chosen.

**Consequences**

Positive:

-

Negative:

-

---

# ADR-001

**Date**

2026-06-26

**Status**

Accepted

**Decision**

Use a Modular Monolith architecture instead of Microservices.

**Context**

StudyOS is a greenfield project with a single development team. Microservices would introduce unnecessary complexity, deployment overhead, and operational costs during early development.

**Alternatives Considered**

- Microservices
- Serverless-only architecture
- Modular Monolith

**Chosen Solution**

Build the backend as a Modular Monolith with clearly isolated modules and well-defined service boundaries.

Each module should own:

- Routes
- Services
- Schemas
- Models
- Repositories
- Business logic

Communication should happen through services/events rather than tightly coupled imports.

**Consequences**

Positive

- Easier development
- Faster iteration
- Simple deployment
- Easier debugging
- Clear migration path to microservices

Negative

- Requires discipline to avoid module coupling.

---

# ADR-002

**Date**

2026-06-26

**Status**

Accepted

**Decision**

Use Next.js App Router for the frontend.

**Context**

The application requires:

- Server Components
- Streaming
- Route groups
- Layout nesting
- Future SSR and SEO support

**Alternatives Considered**

- React + Vite
- Next.js Pages Router
- Next.js App Router

**Chosen Solution**

Next.js App Router.

**Consequences**

Positive

- Better performance
- Better routing
- Native layouts
- Server Components
- Future scalability

Negative

- Slightly steeper learning curve.

---

# ADR-003

**Date**

2026-06-26

**Status**

Accepted

**Decision**

Adopt a Hybrid Frontend Architecture.

**Context**

Pure feature-based architecture causes duplicated UI components.

Pure layer-based architecture becomes difficult to scale.

**Chosen Solution**

Shared reusable UI remains inside:

components/

Feature-specific code lives inside:

features/

Shared logic remains inside:

lib/

**Consequences**

Positive

- High reusability
- Better scalability
- Easier onboarding
- Cleaner ownership

Negative

- Requires discipline when deciding component placement.

---

# ADR-004

**Date**

2026-06-26

**Status**

Accepted

**Decision**

Use FastAPI as the backend framework.

**Context**

StudyOS requires:

- Async APIs
- Type safety
- Automatic OpenAPI generation
- High performance
- AI workload compatibility

**Alternatives Considered**

- Django
- Flask
- FastAPI

**Chosen Solution**

FastAPI.

**Consequences**

Positive

- Excellent async support
- Strong typing
- High performance
- Modern ecosystem

Negative

- Smaller ecosystem than Django.

---

# ADR-005

**Date**

2026-06-26

**Status**

Accepted

**Decision**

Use PostgreSQL as the primary database.

**Context**

StudyOS stores highly relational educational data.

Needs:

- Transactions
- JSONB
- Full-text search
- Extensions
- pgvector

**Alternatives Considered**

- MongoDB
- MySQL
- PostgreSQL

**Chosen Solution**

PostgreSQL 16.

**Consequences**

Positive

- Mature
- Reliable
- Extensible
- Excellent AI ecosystem

Negative

- More operational complexity than NoSQL.

---

# ADR-006

**Date**

2026-06-26

**Status**

Accepted

**Decision**

Store vector embeddings inside PostgreSQL using pgvector.

**Context**

The project requires semantic search without introducing another database early.

**Alternatives Considered**

- Pinecone
- Weaviate
- Qdrant
- pgvector

**Chosen Solution**

pgvector.

**Consequences**

Positive

- Single database
- Lower cost
- Simpler deployment
- Easier backups

Negative

- May require migration for extremely large datasets.

---

# ADR-007

**Date**

2026-06-26

**Status**

Accepted

**Decision**

Use OpenSearch alongside pgvector for Hybrid Search.

**Context**

Semantic search alone performs poorly on exact keyword lookups.

BM25 alone misses semantic similarity.

**Chosen Solution**

Hybrid Search:

- pgvector
- OpenSearch BM25
- Reciprocal Rank Fusion (RRF)

**Consequences**

Positive

- Better retrieval quality
- Better precision
- Better recall

Negative

- Additional infrastructure.

---

# ADR-008

**Date**

2026-06-26

**Status**

Accepted

**Decision**

Use LangGraph for AI workflows.

**Context**

StudyOS contains multiple AI agents requiring stateful, multi-step execution.

Examples:

- Tutor
- Notes
- Quiz
- Planner
- Knowledge Graph

**Alternatives Considered**

- Manual orchestration
- LangChain Chains
- LangGraph

**Chosen Solution**

LangGraph.

**Consequences**

Positive

- Stateful execution
- Retry support
- Easier debugging
- Better workflow composition

Negative

- Additional abstraction layer.

---

# ADR-009

**Date**

2026-06-26

**Status**

Accepted

**Decision**

Use LiteLLM as the LLM gateway.

**Context**

StudyOS should remain model-agnostic.

Need support for:

- OpenAI
- Claude
- Gemini
- Future providers

**Chosen Solution**

LiteLLM.

**Consequences**

Positive

- Easy model switching
- Routing
- Fallbacks
- Cost tracking
- Caching

Negative

- Extra dependency.

---

# ADR-010

**Date**

2026-06-26

**Status**

Accepted

**Decision**

Use UUIDv4 as primary keys across the database.

**Context**

The system may eventually support distributed systems and sharding.

**Alternatives Considered**

- Auto Increment IDs
- UUIDv4

**Chosen Solution**

UUIDv4.

**Consequences**

Positive

- Globally unique
- Easier future scaling
- Safer APIs

Negative

- Slightly larger indexes.

---

# ADR-011

**Date**

2026-06-26

**Status**

Accepted

**Decision**

Adopt asynchronous document processing.

**Context**

OCR, embeddings, metadata extraction, indexing, and AI processing are expensive operations.

Running them synchronously would degrade user experience.

**Chosen Solution**

Upload → Queue → Worker → Processing Pipeline → Ready.

**Consequences**

Positive

- Responsive uploads
- Better scalability
- Retry support

Negative

- More infrastructure.

---

# ADR-012

**Date**

2026-06-26

**Status**

Accepted

**Decision**

Build the platform module-first rather than feature-first.

**Implementation Order**

1. Foundation
2. Knowledge Hub
3. AI Tutor
4. Notes
5. Flashcards
6. Quizzes
7. Planner
8. Calendar
9. Analytics
10. Knowledge Graph

**Reason**

Each later module depends on previous modules.

This minimizes rework.

---

# ADR-013

**Date**

2026-06-26

**Status**

Accepted

**Decision**

Maintain an AI Memory System alongside the codebase.

**Context**

Development frequently switches between developers and AI assistants. Important architectural knowledge can easily be lost if it only exists in conversation history.

**Chosen Solution**

Create a dedicated `AI_MEMORY/` directory containing project state, decisions, next steps, completed work, blockers, mistakes, database notes, AI notes, and a codebase map.

**Consequences**

Positive

- Faster onboarding
- Better continuity across sessions
- Preserves architectural reasoning
- Reduces repeated discussions

Negative

- Requires developers to keep documentation updated.

---

# Future Decisions

Document all future decisions here, including:

- Database schema changes
- Authentication changes
- Infrastructure changes
- AI architecture updates
- Major dependency changes
- Performance optimizations
- Deployment strategy changes
- Breaking architectural changes
