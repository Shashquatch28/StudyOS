# StudyOS Engineering Blueprint
### A Complete System Design Document for Production-Scale AI Learning Platform
**Version:** 1.0 | **Prepared for:** CTO / Lead Engineering Team | **Scope:** 12–18 Month Build

---

## Table of Contents

1. [System Architecture](#1-system-architecture)
2. [Backend Architecture](#2-backend-architecture)
3. [Frontend Architecture](#3-frontend-architecture)
4. [Database Design](#4-database-design)
5. [AI Architecture](#5-ai-architecture)
6. [Knowledge Pipeline](#6-knowledge-pipeline)
7. [Search Architecture](#7-search-architecture)
8. [Module-by-Module Technical Design](#8-module-by-module-technical-design)
9. [API Design](#9-api-design)
10. [Authentication & Authorization](#10-authentication--authorization)
11. [Security Architecture](#11-security-architecture)
12. [DevOps & Infrastructure](#12-devops--infrastructure)
13. [Development Roadmap](#13-development-roadmap)
14. [Project Structure](#14-project-structure)
15. [Coding Standards](#15-coding-standards)
16. [Scaling Strategy](#16-scaling-strategy)
17. [Cost Optimization](#17-cost-optimization)
18. [Future Architecture](#18-future-architecture)

---

# 1. System Architecture

## 1.1 Architectural Decision: Modular Monolith First

**Decision:** Begin with a Modular Monolith, not Microservices.

**Rationale:**
- Microservices require distributed system expertise and infrastructure overhead that slows MVP velocity
- Modular Monolith enforces domain boundaries via Python packages with strict import rules
- When scale demands it (>100K users), individual modules can be extracted into microservices without rewriting business logic
- Database per module can be introduced incrementally by splitting PostgreSQL schemas
- The LangGraph AI agent layer is inherently distributed — it is the only "micro" component from day one

**Architecture Evolution:**
- **0 → 10K users:** Modular Monolith + LangGraph worker pool
- **10K → 100K users:** Extract Knowledge Pipeline and AI Processing into separate services
- **100K → 1M users:** Full service decomposition with event-driven communication

---

## 1.2 Overall System Diagram

```mermaid
graph TB
    subgraph "Client Layer"
        WEB[Next.js Web App]
        MOB[Mobile App - Future]
    end

    subgraph "Edge Layer"
        CDN[CloudFront CDN]
        WAF[AWS WAF]
    end

    subgraph "API Layer"
        GW[API Gateway - Kong / AWS ALB]
        WS[WebSocket Server - FastAPI + Starlette]
    end

    subgraph "Application Layer"
        AUTH[Auth Service]
        ORCH[AI Orchestrator]
        API[FastAPI Modular Monolith]
    end

    subgraph "AI Layer"
        LG[LangGraph Agent Runtime]
        LITELLM[LiteLLM Proxy]
        EMBED[Embedding Service]
    end

    subgraph "Worker Layer"
        CELERY[Celery Workers]
        BEAT[Celery Beat Scheduler]
    end

    subgraph "Data Layer"
        PG[(PostgreSQL + pgvector)]
        REDIS[(Redis Cluster)]
        S3[(AWS S3)]
        ES[(OpenSearch)]
    end

    subgraph "AI Providers"
        GPT[OpenAI GPT-4o]
        GEM[Google Gemini]
        CLA[Anthropic Claude]
    end

    subgraph "External Services"
        GCA[Google Calendar]
        GDOC[Google Document AI]
        YT[YouTube Data API]
    end

    subgraph "Observability"
        PROM[Prometheus]
        GRAF[Grafana]
        SENT[Sentry]
        LANG[LangSmith]
    end

    WEB --> CDN
    MOB --> CDN
    CDN --> WAF
    WAF --> GW
    GW --> AUTH
    GW --> API
    GW --> WS
    API --> ORCH
    ORCH --> LG
    LG --> LITELLM
    LITELLM --> GPT
    LITELLM --> GEM
    LITELLM --> CLA
    API --> CELERY
    CELERY --> EMBED
    CELERY --> PG
    CELERY --> S3
    API --> PG
    API --> REDIS
    API --> S3
    API --> ES
    BEAT --> CELERY
    LG --> PG
    LG --> REDIS
```

---

## 1.3 Service Boundaries

| Service | Responsibility | Technology | Owns |
|---------|---------------|------------|------|
| API Service | Business logic, CRUD, orchestration | FastAPI | PostgreSQL schemas |
| AI Orchestrator | Agent coordination, workflow management | LangGraph | Redis state |
| Knowledge Worker | File processing, OCR, embedding | Celery | S3, pgvector |
| Notification Service | Push, email, in-app alerts | Celery + SES | Notification queue |
| Search Service | Hybrid search, ranking | OpenSearch + pgvector | Search indices |
| Auth Service | JWT, OAuth, session | FastAPI + Redis | User sessions |
| WebSocket Server | Real-time AI streaming | FastAPI + Starlette | WS connections |

---

## 1.4 API Gateway Design

Use **AWS Application Load Balancer (ALB)** as the primary ingress with **Kong** for advanced API management.

Kong handles:
- Rate limiting (per user tier, per endpoint)
- JWT validation (offloaded from services)
- Request logging
- API key management for future public APIs
- Route-based load balancing

```mermaid
graph LR
    CLIENT[Client] --> ALB[AWS ALB]
    ALB --> KONG[Kong Gateway]
    KONG --> |/api/v1/*| MAIN[FastAPI Main]
    KONG --> |/ws/*| WS[WebSocket Server]
    KONG --> |/ai/*| ORCH[AI Orchestrator]
    KONG --> |/health| HEALTH[Health Check]
```

**ALB Target Groups:**
- `studyos-api` → FastAPI instances (ECS tasks)
- `studyos-ws` → WebSocket servers (ECS tasks, sticky sessions)
- `studyos-ai` → AI Orchestrator (ECS tasks)

---

## 1.5 AI Orchestrator Architecture

The AI Orchestrator is the brain of the platform. It receives structured requests from the API layer and routes them to the appropriate LangGraph workflow.

```mermaid
graph TD
    API[API Service] --> ORCH[Orchestrator Service]
    ORCH --> ROUTER[Intent Router]
    ROUTER --> |knowledge| KA[Knowledge Agent Workflow]
    ROUTER --> |tutor| TA[Tutor Agent Workflow]
    ROUTER --> |planner| PA[Planner Agent Workflow]
    ROUTER --> |flashcard| FA[Flashcard Agent Workflow]
    ROUTER --> |quiz| QA[Quiz Agent Workflow]
    ROUTER --> |research| RA[Research Agent Workflow]
    ROUTER --> |analytics| AA[Analytics Agent Workflow]

    KA --> LITELLM[LiteLLM Proxy]
    TA --> LITELLM
    PA --> LITELLM
    FA --> LITELLM
    QA --> LITELLM
    RA --> LITELLM
    AA --> LITELLM

    LITELLM --> |primary| GPT4O[GPT-4o]
    LITELLM --> |fallback| GEMINI[Gemini 1.5 Pro]
    LITELLM --> |long context| CLAUDE[Claude 3.5]

    KA --> STATE[Redis State Store]
    TA --> STATE
    PA --> STATE
```

---

## 1.6 Event-Driven Communication

**Technology:** Redis Streams (primary), with migration path to Apache Kafka at >100K users.

**Why Redis Streams over Kafka for MVP:**
- No additional infrastructure to manage
- Sufficient throughput for early scale (millions of events/day)
- Native Celery integration
- Consumer groups for parallel processing

```mermaid
graph LR
    subgraph "Producers"
        API[API Service]
        WORKER[Celery Workers]
        LG[LangGraph]
    end

    subgraph "Redis Streams"
        RS1[resource.uploaded]
        RS2[resource.processed]
        RS3[flashcard.generated]
        RS4[quiz.generated]
        RS5[mastery.updated]
        RS6[notification.queued]
        RS7[study.session.completed]
    end

    subgraph "Consumers"
        KW[Knowledge Worker]
        AW[Analytics Worker]
        NW[Notification Worker]
        SW[Search Indexer]
    end

    API --> RS1
    LG --> RS3
    LG --> RS4
    WORKER --> RS2
    RS1 --> KW
    RS2 --> SW
    RS2 --> AW
    RS3 --> SW
    RS5 --> NW
    RS7 --> AW
```

**Core Events Schema:**

```python
# events/schemas.py
from pydantic import BaseModel
from datetime import datetime
from uuid import UUID

class BaseEvent(BaseModel):
    event_id: UUID
    event_type: str
    user_id: UUID
    tenant_id: UUID
    timestamp: datetime
    version: str = "1.0"

class ResourceUploadedEvent(BaseEvent):
    event_type: str = "resource.uploaded"
    resource_id: UUID
    file_type: str
    file_size_bytes: int
    storage_key: str

class ResourceProcessedEvent(BaseEvent):
    event_type: str = "resource.processed"
    resource_id: UUID
    chunk_count: int
    embedding_count: int
    processing_duration_ms: int

class FlashcardGeneratedEvent(BaseEvent):
    event_type: str = "flashcard.generated"
    resource_id: UUID
    deck_id: UUID
    card_count: int
```

---

## 1.7 WebSocket Architecture

Real-time AI streaming is critical for the Tutor and AI Workspace modules.

**Technology:** FastAPI + Starlette WebSockets with Redis Pub/Sub for horizontal scaling.

```mermaid
sequenceDiagram
    participant C as Client
    participant WS as WS Server
    participant REDIS as Redis Pub/Sub
    participant ORCH as AI Orchestrator
    participant LLM as LiteLLM

    C->>WS: Connect (JWT in query param)
    WS->>WS: Authenticate, create channel_id
    C->>WS: Send message {type: "tutor_query", content: "...", context_ids: [...]}
    WS->>ORCH: Dispatch to Tutor Agent workflow
    ORCH->>LLM: Stream tokens
    LLM-->>ORCH: Token stream
    ORCH-->>REDIS: Publish to channel_{user_id}
    REDIS-->>WS: Receive token
    WS-->>C: Stream token to client
    ORCH-->>REDIS: Publish {type: "done", citations: [...]}
    REDIS-->>WS: Done signal
    WS-->>C: Send citations + close stream
```

**WebSocket Message Protocol:**

```typescript
// Outgoing (Client → Server)
interface WsClientMessage {
  type: 'tutor_query' | 'workspace_chat' | 'ping';
  request_id: string;
  payload: {
    content: string;
    context_ids?: string[];    // resource IDs for context
    conversation_id?: string;  // for continuations
    stream: boolean;
  };
}

// Incoming (Server → Client)
interface WsServerMessage {
  type: 'token' | 'done' | 'error' | 'pong';
  request_id: string;
  payload: {
    token?: string;
    citations?: Citation[];
    error?: string;
    usage?: TokenUsage;
  };
}
```

---

## 1.8 Notification Pipeline

```mermaid
graph TD
    TRIGGER[Trigger Sources] --> NQ[Notification Queue - Redis]
    NQ --> NW[Notification Worker - Celery]
    NW --> CHANNEL{Channel Router}
    CHANNEL --> |email| SES[AWS SES]
    CHANNEL --> |push| FCM[Firebase FCM]
    CHANNEL --> |in-app| WS[WebSocket]
    CHANNEL --> |sms - future| SNS[AWS SNS]

    subgraph "Trigger Sources"
        BEAT[Celery Beat - Scheduled]
        API_T[API Events]
        AI_T[AI Agent Triggers]
    end
```

---

## 1.9 File Processing Pipeline

```mermaid
graph TD
    UPLOAD[Client Upload] --> PRE[Presigned URL Request]
    PRE --> S3[Direct Upload to S3]
    S3 --> S3E[S3 Event Notification]
    S3E --> SQS[AWS SQS]
    SQS --> CELERY[Celery Worker]
    CELERY --> VALID[File Validation]
    VALID --> MIME[MIME Type Check]
    MIME --> VIRUS[Virus Scan - ClamAV]
    VIRUS --> EXTRACT[Content Extraction]
    EXTRACT --> OCR{Needs OCR?}
    OCR --> |Yes| GDOCAI[Google Document AI]
    OCR --> |No| PARSE[Direct Parse]
    GDOCAI --> CLEAN[Text Cleaning]
    PARSE --> CLEAN
    CLEAN --> CHUNK[Chunking Service]
    CHUNK --> META[Metadata Extraction]
    META --> EMBED[Embedding Generation]
    EMBED --> PGVEC[(pgvector)]
    EMBED --> INDEX[OpenSearch Index]
    INDEX --> KG[Knowledge Graph Update]
    KG --> DONE[Mark Resource Ready]
    DONE --> EVENT[Publish resource.processed event]
```

---

## 1.10 Authentication Flow

```mermaid
sequenceDiagram
    participant C as Client
    participant GW as API Gateway
    participant AUTH as Auth Service
    participant REDIS as Redis
    participant DB as PostgreSQL

    C->>GW: POST /api/v1/auth/google
    GW->>AUTH: Forward request
    AUTH->>AUTH: Verify Google token
    AUTH->>DB: Upsert user record
    AUTH->>REDIS: Store refresh token (TTL 30d)
    AUTH-->>C: {access_token (15min), refresh_token (30d)}

    Note over C,AUTH: Subsequent requests

    C->>GW: GET /api/v1/resources (Bearer: access_token)
    GW->>AUTH: Validate JWT (signature + expiry)
    AUTH-->>GW: {user_id, role, tenant_id}
    GW->>API: Forward with user context headers
    API-->>C: Response

    Note over C,AUTH: Token refresh

    C->>AUTH: POST /api/v1/auth/refresh (refresh_token)
    AUTH->>REDIS: Validate refresh token
    AUTH->>AUTH: Issue new access token
    AUTH-->>C: {access_token (15min)}
```

---

## 1.11 RAG Architecture

```mermaid
graph TD
    QUERY[User Query] --> QE[Query Embedding]
    QE --> HYD[HyDE - Hypothetical Doc Embedding]
    HYD --> HYBRID[Hybrid Retrieval]
    HYBRID --> VEC[Vector Search - pgvector]
    HYBRID --> BM25[BM25 - OpenSearch]
    VEC --> MERGE[RRF Merge - Reciprocal Rank Fusion]
    BM25 --> MERGE
    MERGE --> RERANK[Cross-Encoder Re-ranking]
    RERANK --> FILTER[Metadata Filter]
    FILTER --> |user_id + resource_ids| CHUNKS[Top-K Chunks]
    CHUNKS --> COMPRESS[Context Compression - LLMLingua]
    COMPRESS --> PROMPT[Prompt Assembly]
    PROMPT --> LLM[LiteLLM]
    LLM --> STREAM[Streaming Response]
    STREAM --> CITE[Citation Extraction]
    CITE --> CLIENT[Client]
```

---

## 1.12 Analytics Pipeline

```mermaid
graph LR
    subgraph "Event Sources"
        API[API Events]
        WS2[WebSocket Events]
        CELERY2[Worker Events]
    end

    subgraph "Ingestion"
        REDIS_S[Redis Streams]
    end

    subgraph "Processing"
        AW2[Analytics Worker - Celery]
        AGG[Aggregator]
    end

    subgraph "Storage"
        PG_A[(PostgreSQL - analytics schema)]
        TS[TimescaleDB - future]
    end

    subgraph "Serving"
        API_A[Analytics API]
        GRAF2[Grafana Dashboard]
    end

    API --> REDIS_S
    WS2 --> REDIS_S
    CELERY2 --> REDIS_S
    REDIS_S --> AW2
    AW2 --> AGG
    AGG --> PG_A
    PG_A --> API_A
    PG_A --> GRAF2
```

---

# 2. Backend Architecture

## 2.1 Technology Decisions

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Framework | FastAPI | Async-first, Pydantic native, OpenAPI auto-generation, battle-tested at scale |
| ORM | SQLAlchemy 2.0 (async) | Industry standard, excellent PostgreSQL support, type-safe queries |
| Migration | Alembic | SQLAlchemy's native migration tool, revision-based, deterministic |
| DI Container | `dependency-injector` | Explicit DI wiring, testable, production-proven |
| Task Queue | Celery 5 + Redis | Mature, horizontally scalable, excellent monitoring with Flower |
| Schema Validation | Pydantic v2 | 5-10x faster than v1, Rust-backed validation, native FastAPI integration |
| HTTP Client | `httpx` | Async-first, mirrors `requests` API, connection pooling |
| Testing | pytest + pytest-asyncio | Async test support, fixture system superior to unittest |

---

## 2.2 Folder Structure (Complete)

```
studyos-backend/
├── app/
│   ├── __init__.py
│   ├── main.py                     # FastAPI app factory
│   ├── config.py                   # Pydantic Settings
│   ├── dependencies.py             # Shared FastAPI dependencies
│   │
│   ├── core/                       # Cross-cutting infrastructure
│   │   ├── __init__.py
│   │   ├── database.py             # SQLAlchemy engine + session factory
│   │   ├── redis.py                # Redis client factory
│   │   ├── s3.py                   # S3/Boto3 client factory
│   │   ├── security.py             # JWT, password hashing
│   │   ├── exceptions.py           # Base exception hierarchy
│   │   ├── middleware.py           # CORS, request ID, logging
│   │   ├── events.py               # App startup/shutdown events
│   │   └── logging.py              # Structured logging setup
│   │
│   ├── shared/                     # Shared domain primitives
│   │   ├── __init__.py
│   │   ├── base_model.py           # SQLAlchemy Base + audit mixin
│   │   ├── base_repository.py      # Generic CRUD repository
│   │   ├── base_service.py         # Base service class
│   │   ├── pagination.py           # Cursor + offset pagination
│   │   ├── filters.py              # Filter base classes
│   │   └── schemas.py              # Shared Pydantic schemas
│   │
│   ├── modules/                    # Domain modules (bounded contexts)
│   │   │
│   │   ├── auth/
│   │   │   ├── __init__.py
│   │   │   ├── router.py           # FastAPI router
│   │   │   ├── service.py          # Auth business logic
│   │   │   ├── schemas.py          # Request/response Pydantic models
│   │   │   ├── models.py           # SQLAlchemy models
│   │   │   ├── repository.py       # Database access
│   │   │   ├── dependencies.py     # Module-level DI
│   │   │   └── exceptions.py       # Module exceptions
│   │   │
│   │   ├── users/
│   │   │   ├── router.py
│   │   │   ├── service.py
│   │   │   ├── schemas.py
│   │   │   ├── models.py
│   │   │   └── repository.py
│   │   │
│   │   ├── knowledge/              # Knowledge Hub module
│   │   │   ├── router.py
│   │   │   ├── service.py
│   │   │   ├── schemas.py
│   │   │   ├── models.py
│   │   │   ├── repository.py
│   │   │   ├── processors/         # File type processors
│   │   │   │   ├── pdf.py
│   │   │   │   ├── docx.py
│   │   │   │   ├── pptx.py
│   │   │   │   ├── image.py
│   │   │   │   ├── audio.py
│   │   │   │   ├── video.py
│   │   │   │   └── web.py
│   │   │   └── chunkers/
│   │   │       ├── semantic.py
│   │   │       ├── fixed.py
│   │   │       └── hierarchical.py
│   │   │
│   │   ├── notes/
│   │   ├── flashcards/
│   │   ├── quizzes/
│   │   ├── planner/
│   │   ├── calendar/
│   │   ├── tasks/
│   │   ├── mastery/
│   │   ├── analytics/
│   │   ├── notifications/
│   │   ├── research/
│   │   ├── search/
│   │   └── workspace/
│   │
│   ├── ai/                         # AI subsystem
│   │   ├── __init__.py
│   │   ├── orchestrator.py         # Request routing to agents
│   │   ├── litellm_client.py       # LiteLLM wrapper
│   │   ├── embedding.py            # Embedding service
│   │   ├── rag/
│   │   │   ├── retriever.py        # Hybrid retrieval
│   │   │   ├── reranker.py         # Cross-encoder reranking
│   │   │   ├── compressor.py       # Context compression
│   │   │   └── citation.py         # Citation extraction
│   │   ├── agents/
│   │   │   ├── knowledge_agent.py
│   │   │   ├── tutor_agent.py
│   │   │   ├── planner_agent.py
│   │   │   ├── flashcard_agent.py
│   │   │   ├── quiz_agent.py
│   │   │   ├── memory_agent.py
│   │   │   ├── research_agent.py
│   │   │   ├── analytics_agent.py
│   │   │   └── notification_agent.py
│   │   ├── prompts/
│   │   │   ├── tutor.py
│   │   │   ├── flashcard.py
│   │   │   ├── quiz.py
│   │   │   ├── notes.py
│   │   │   ├── planner.py
│   │   │   └── base.py
│   │   └── memory/
│   │       ├── short_term.py       # Conversation buffer
│   │       ├── long_term.py        # PostgreSQL-backed
│   │       └── episodic.py         # Session summaries
│   │
│   ├── workers/                    # Celery tasks
│   │   ├── __init__.py
│   │   ├── celery_app.py           # Celery application factory
│   │   ├── knowledge_tasks.py      # File processing tasks
│   │   ├── embedding_tasks.py      # Embedding generation
│   │   ├── notification_tasks.py   # Notification dispatch
│   │   ├── analytics_tasks.py      # Metrics aggregation
│   │   ├── maintenance_tasks.py    # DB cleanup, cache warming
│   │   └── scheduler.py            # Celery Beat schedule
│   │
│   └── api/
│       ├── __init__.py
│       └── v1/
│           ├── __init__.py
│           └── router.py           # Aggregates all module routers
│
├── tests/
│   ├── conftest.py
│   ├── unit/
│   │   ├── test_auth_service.py
│   │   ├── test_knowledge_service.py
│   │   └── ...
│   ├── integration/
│   │   ├── test_auth_api.py
│   │   ├── test_knowledge_api.py
│   │   └── ...
│   └── e2e/
│       └── test_learning_flow.py
│
├── alembic/
│   ├── env.py
│   ├── script.py.mako
│   └── versions/
│
├── docker/
│   ├── Dockerfile.api
│   ├── Dockerfile.worker
│   └── Dockerfile.beat
│
├── scripts/
│   ├── seed.py
│   └── migrate.sh
│
├── pyproject.toml
├── poetry.lock
├── .env.example
└── docker-compose.yml
```

---

## 2.3 Domain-Driven Design Implementation

StudyOS uses a simplified DDD approach — Bounded Contexts mapped to Python modules with clear anti-corruption layers at module boundaries.

**Bounded Contexts:**

| Context | Module | Responsibility |
|---------|--------|----------------|
| Identity | `auth`, `users` | Authentication, profiles |
| Knowledge | `knowledge` | Resource management, processing |
| Learning | `notes`, `flashcards`, `quizzes` | Study material generation |
| Planning | `planner`, `calendar`, `tasks` | Schedule management |
| Intelligence | `mastery`, `analytics` | Performance tracking |
| Communication | `notifications` | Alerts and reminders |
| Discovery | `research`, `search` | Resource discovery |

**Module Interaction Rule:** Modules communicate via service interfaces, never via direct ORM model imports. Module A cannot import Module B's SQLAlchemy model — it must call Module B's service.

---

## 2.4 Layered Architecture per Module

```
Router (FastAPI)           ← HTTP boundary, schema validation
    ↓
Service (Business Logic)   ← Domain rules, orchestration
    ↓
Repository (Data Access)   ← Database queries only
    ↓
Model (SQLAlchemy)         ← Table mapping only
```

**Example — Knowledge Module:**

```python
# modules/knowledge/models.py
from app.shared.base_model import BaseModel
from sqlalchemy import Column, String, BigInteger, Enum
from sqlalchemy.dialects.postgresql import UUID, JSONB
import uuid

class Resource(BaseModel):
    __tablename__ = "resources"

    id = Column(UUID(as_uuid=True), primary_key=True, default=uuid.uuid4)
    user_id = Column(UUID(as_uuid=True), ForeignKey("users.id"), nullable=False, index=True)
    title = Column(String(500), nullable=False)
    file_type = Column(Enum("pdf","docx","pptx","image","audio","video","url","youtube"), nullable=False)
    storage_key = Column(String(1000), nullable=True)
    processing_status = Column(Enum("pending","processing","ready","failed"), default="pending", index=True)
    chunk_count = Column(BigInteger, default=0)
    metadata = Column(JSONB, default=dict)


# modules/knowledge/repository.py
from app.shared.base_repository import BaseRepository
from .models import Resource

class ResourceRepository(BaseRepository[Resource]):
    async def get_by_user(self, user_id: UUID, status: str | None = None) -> list[Resource]:
        stmt = select(Resource).where(Resource.user_id == user_id)
        if status:
            stmt = stmt.where(Resource.processing_status == status)
        return await self.session.scalars(stmt)

    async def get_ready_for_user(self, user_id: UUID) -> list[Resource]:
        return await self.get_by_user(user_id, status="ready")


# modules/knowledge/service.py
class KnowledgeService:
    def __init__(
        self,
        repo: ResourceRepository,
        s3_client: S3Client,
        event_publisher: EventPublisher,
    ):
        self.repo = repo
        self.s3 = s3_client
        self.events = event_publisher

    async def initiate_upload(self, user_id: UUID, filename: str, content_type: str) -> UploadInitiatedSchema:
        resource = Resource(user_id=user_id, title=filename, file_type=_detect_type(content_type))
        await self.repo.create(resource)
        presigned = await self.s3.generate_presigned_upload(
            key=f"resources/{resource.id}/{filename}",
            content_type=content_type,
            expires_in=3600,
        )
        return UploadInitiatedSchema(resource_id=resource.id, upload_url=presigned.url, fields=presigned.fields)

    async def confirm_upload(self, resource_id: UUID, user_id: UUID) -> Resource:
        resource = await self.repo.get_or_raise(resource_id, user_id)
        resource.processing_status = "processing"
        await self.repo.update(resource)
        await self.events.publish(ResourceUploadedEvent(
            resource_id=resource.id,
            user_id=user_id,
            storage_key=resource.storage_key,
        ))
        return resource
```

---

## 2.5 Base Repository Pattern

```python
# shared/base_repository.py
from typing import TypeVar, Generic, Type
from uuid import UUID
from sqlalchemy.ext.asyncio import AsyncSession
from sqlalchemy import select
from .exceptions import NotFoundError

T = TypeVar("T")

class BaseRepository(Generic[T]):
    def __init__(self, session: AsyncSession, model: Type[T]):
        self.session = session
        self.model = model

    async def get_by_id(self, id: UUID) -> T | None:
        return await self.session.get(self.model, id)

    async def get_or_raise(self, id: UUID, user_id: UUID | None = None) -> T:
        obj = await self.get_by_id(id)
        if not obj:
            raise NotFoundError(f"{self.model.__name__} {id} not found")
        if user_id and obj.user_id != user_id:
            raise NotFoundError(f"{self.model.__name__} {id} not found")  # Mask as 404 for security
        return obj

    async def create(self, obj: T) -> T:
        self.session.add(obj)
        await self.session.flush()
        await self.session.refresh(obj)
        return obj

    async def update(self, obj: T) -> T:
        await self.session.flush()
        await self.session.refresh(obj)
        return obj

    async def soft_delete(self, obj: T) -> None:
        obj.deleted_at = datetime.utcnow()
        await self.session.flush()

    async def paginate(
        self,
        stmt,
        page: int = 1,
        size: int = 20
    ) -> tuple[list[T], int]:
        count_stmt = select(func.count()).select_from(stmt.subquery())
        total = await self.session.scalar(count_stmt)
        results = await self.session.scalars(stmt.offset((page-1)*size).limit(size))
        return list(results), total
```

---

## 2.6 Dependency Injection

Use FastAPI's built-in DI system with a `dependency-injector` container for complex wiring.

```python
# core/container.py
from dependency_injector import containers, providers
from app.core.database import get_async_session
from app.modules.knowledge.repository import ResourceRepository
from app.modules.knowledge.service import KnowledgeService

class Container(containers.DeclarativeContainer):
    # Core
    db_session = providers.Resource(get_async_session)
    redis = providers.Singleton(create_redis_client)
    s3_client = providers.Singleton(create_s3_client)
    event_publisher = providers.Singleton(RedisEventPublisher, redis=redis)

    # Knowledge
    resource_repo = providers.Factory(ResourceRepository, session=db_session, model=Resource)
    knowledge_service = providers.Factory(
        KnowledgeService,
        repo=resource_repo,
        s3_client=s3_client,
        event_publisher=event_publisher,
    )

# Usage in router
@router.post("/resources/upload-initiate")
async def initiate_upload(
    request: UploadInitiateRequest,
    service: KnowledgeService = Depends(Provide[Container.knowledge_service]),
    current_user: User = Depends(get_current_user),
):
    return await service.initiate_upload(current_user.id, request.filename, request.content_type)
```

---

## 2.7 Configuration Management

```python
# config.py
from pydantic_settings import BaseSettings, SettingsConfigDict
from functools import lru_cache

class Settings(BaseSettings):
    model_config = SettingsConfigDict(env_file=".env", env_file_encoding="utf-8", case_sensitive=False)

    # App
    APP_NAME: str = "StudyOS"
    DEBUG: bool = False
    ENVIRONMENT: str = "production"
    SECRET_KEY: str
    API_V1_PREFIX: str = "/api/v1"

    # Database
    DATABASE_URL: str
    DATABASE_POOL_SIZE: int = 20
    DATABASE_MAX_OVERFLOW: int = 10
    DATABASE_ECHO: bool = False

    # Redis
    REDIS_URL: str
    REDIS_MAX_CONNECTIONS: int = 100

    # AWS
    AWS_REGION: str = "us-east-1"
    AWS_ACCESS_KEY_ID: str
    AWS_SECRET_ACCESS_KEY: str
    S3_BUCKET_NAME: str
    S3_PRESIGNED_URL_EXPIRY: int = 3600

    # Auth
    JWT_SECRET_KEY: str
    JWT_ALGORITHM: str = "HS256"
    ACCESS_TOKEN_EXPIRE_MINUTES: int = 15
    REFRESH_TOKEN_EXPIRE_DAYS: int = 30
    GOOGLE_CLIENT_ID: str
    GOOGLE_CLIENT_SECRET: str

    # AI
    OPENAI_API_KEY: str
    ANTHROPIC_API_KEY: str
    GOOGLE_AI_API_KEY: str
    LITELLM_MASTER_KEY: str
    LANGSMITH_API_KEY: str | None = None

    # Celery
    CELERY_BROKER_URL: str
    CELERY_RESULT_BACKEND: str

    # Embedding
    EMBEDDING_MODEL: str = "text-embedding-3-large"
    EMBEDDING_DIMENSIONS: int = 3072
    EMBEDDING_BATCH_SIZE: int = 100

    # Search
    OPENSEARCH_URL: str
    OPENSEARCH_INDEX_PREFIX: str = "studyos"

    # Rate Limiting
    RATE_LIMIT_PER_MINUTE: int = 60
    AI_RATE_LIMIT_PER_HOUR: int = 100

@lru_cache
def get_settings() -> Settings:
    return Settings()

settings = get_settings()
```

---

## 2.8 Exception Handling

```python
# core/exceptions.py
class StudyOSError(Exception):
    """Base exception"""
    def __init__(self, message: str, code: str = "INTERNAL_ERROR"):
        self.message = message
        self.code = code
        super().__init__(message)

class NotFoundError(StudyOSError):
    def __init__(self, message: str):
        super().__init__(message, "NOT_FOUND")

class ForbiddenError(StudyOSError):
    def __init__(self, message: str = "Access denied"):
        super().__init__(message, "FORBIDDEN")

class ValidationError(StudyOSError):
    def __init__(self, message: str, field: str | None = None):
        self.field = field
        super().__init__(message, "VALIDATION_ERROR")

class RateLimitError(StudyOSError):
    def __init__(self, retry_after: int):
        self.retry_after = retry_after
        super().__init__(f"Rate limit exceeded. Retry after {retry_after}s", "RATE_LIMIT_EXCEEDED")

class AIProcessingError(StudyOSError):
    def __init__(self, message: str):
        super().__init__(message, "AI_PROCESSING_FAILED")


# core/middleware.py - Global exception handler
from fastapi import Request
from fastapi.responses import JSONResponse

async def global_exception_handler(request: Request, exc: Exception) -> JSONResponse:
    if isinstance(exc, NotFoundError):
        return JSONResponse(status_code=404, content={"error": {"code": exc.code, "message": exc.message}})
    if isinstance(exc, ForbiddenError):
        return JSONResponse(status_code=403, content={"error": {"code": exc.code, "message": exc.message}})
    if isinstance(exc, ValidationError):
        return JSONResponse(status_code=422, content={"error": {"code": exc.code, "message": exc.message, "field": exc.field}})
    if isinstance(exc, RateLimitError):
        return JSONResponse(
            status_code=429,
            headers={"Retry-After": str(exc.retry_after)},
            content={"error": {"code": exc.code, "message": exc.message}}
        )

    # Unexpected - log and return 500
    logger.exception("Unhandled exception", request_id=request.state.request_id)
    return JSONResponse(status_code=500, content={"error": {"code": "INTERNAL_ERROR", "message": "An unexpected error occurred"}})
```

---

## 2.9 Caching Strategy

**Three-tier caching:**

```python
# core/cache.py
from enum import IntEnum
from functools import wraps
import json

class CacheTTL(IntEnum):
    SHORT = 300       # 5 minutes - search results, user sessions
    MEDIUM = 3600     # 1 hour - processed resources, notes
    LONG = 86400      # 24 hours - embeddings, flashcard decks
    WEEK = 604800     # 7 days - static reference data

class CacheKey:
    @staticmethod
    def user_profile(user_id: str) -> str:
        return f"user:profile:{user_id}"

    @staticmethod
    def resource(resource_id: str) -> str:
        return f"resource:{resource_id}"

    @staticmethod
    def search_results(user_id: str, query_hash: str) -> str:
        return f"search:{user_id}:{query_hash}"

    @staticmethod
    def flashcard_deck(deck_id: str) -> str:
        return f"flashcards:deck:{deck_id}"

    @staticmethod
    def ai_response(prompt_hash: str) -> str:
        return f"ai:response:{prompt_hash}"

    @staticmethod
    def daily_plan(user_id: str, date: str) -> str:
        return f"planner:daily:{user_id}:{date}"


def cache_response(key_fn, ttl: int = CacheTTL.MEDIUM):
    """Decorator for caching service method responses"""
    def decorator(func):
        @wraps(func)
        async def wrapper(self, *args, **kwargs):
            key = key_fn(*args, **kwargs)
            cached = await self.redis.get(key)
            if cached:
                return json.loads(cached)
            result = await func(self, *args, **kwargs)
            await self.redis.setex(key, ttl, json.dumps(result, default=str))
            return result
        return wrapper
    return decorator
```

---

## 2.10 Celery Architecture

```python
# workers/celery_app.py
from celery import Celery
from kombu import Exchange, Queue
from app.config import settings

celery_app = Celery("studyos")

celery_app.conf.update(
    broker_url=settings.CELERY_BROKER_URL,
    result_backend=settings.CELERY_RESULT_BACKEND,
    task_serializer="json",
    accept_content=["json"],
    result_serializer="json",
    timezone="UTC",
    task_track_started=True,
    task_acks_late=True,  # Ack after completion, not on receipt (safer)
    worker_prefetch_multiplier=1,  # Process one task at a time per worker
    task_routes={
        "workers.knowledge_tasks.*": {"queue": "knowledge"},
        "workers.embedding_tasks.*": {"queue": "embedding"},
        "workers.notification_tasks.*": {"queue": "notifications"},
        "workers.analytics_tasks.*": {"queue": "analytics"},
        "workers.maintenance_tasks.*": {"queue": "maintenance"},
    },
    task_queues=[
        Queue("knowledge", Exchange("knowledge"), routing_key="knowledge", max_priority=10),
        Queue("embedding", Exchange("embedding"), routing_key="embedding"),
        Queue("notifications", Exchange("notifications"), routing_key="notifications"),
        Queue("analytics", Exchange("analytics"), routing_key="analytics"),
        Queue("maintenance", Exchange("maintenance"), routing_key="maintenance"),
    ],
    beat_schedule={
        "daily-revision-reminders": {
            "task": "workers.notification_tasks.send_daily_reminders",
            "schedule": crontab(hour=7, minute=0),
        },
        "aggregate-daily-analytics": {
            "task": "workers.analytics_tasks.aggregate_daily_stats",
            "schedule": crontab(hour=0, minute=30),
        },
        "refresh-mastery-scores": {
            "task": "workers.maintenance_tasks.refresh_mastery_scores",
            "schedule": crontab(hour=2, minute=0),
        },
        "cleanup-expired-sessions": {
            "task": "workers.maintenance_tasks.cleanup_expired_sessions",
            "schedule": crontab(hour=3, minute=0),
        },
    }
)
```

---

## 2.11 Rate Limiting

```python
# core/rate_limiter.py
from fastapi import Request, HTTPException
import redis.asyncio as redis

class RateLimiter:
    def __init__(self, redis_client, calls: int, period: int):
        self.redis = redis_client
        self.calls = calls
        self.period = period

    async def check(self, identifier: str) -> None:
        key = f"ratelimit:{identifier}"
        pipe = self.redis.pipeline()
        pipe.incr(key)
        pipe.expire(key, self.period)
        count, _ = await pipe.execute()
        if count > self.calls:
            ttl = await self.redis.ttl(key)
            raise RateLimitError(retry_after=ttl)

# Usage
api_limiter = RateLimiter(redis_client, calls=60, period=60)
ai_limiter = RateLimiter(redis_client, calls=100, period=3600)

async def rate_limit_api(request: Request):
    user_id = request.state.user_id
    await api_limiter.check(f"api:{user_id}")

async def rate_limit_ai(request: Request):
    user_id = request.state.user_id
    await ai_limiter.check(f"ai:{user_id}")
```

---

## 2.12 API Versioning

All APIs are versioned at the URL level: `/api/v1/`, `/api/v2/`.

```python
# api/v1/router.py
from fastapi import APIRouter
from app.modules.auth.router import router as auth_router
from app.modules.knowledge.router import router as knowledge_router
# ... etc

v1_router = APIRouter(prefix="/api/v1")
v1_router.include_router(auth_router, prefix="/auth", tags=["Auth"])
v1_router.include_router(knowledge_router, prefix="/knowledge", tags=["Knowledge"])

# main.py
app.include_router(v1_router)
```

When introducing breaking changes, create `/api/v2/` with new routers while maintaining `/api/v1/` for 6 months.

---

## 2.13 Testing Strategy

```python
# tests/conftest.py
import pytest
import asyncio
from httpx import AsyncClient
from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession
from app.main import create_app
from app.core.database import Base

@pytest.fixture(scope="session")
def event_loop():
    return asyncio.get_event_loop()

@pytest.fixture(scope="session")
async def engine():
    engine = create_async_engine("postgresql+asyncpg://test:test@localhost/studyos_test")
    async with engine.begin() as conn:
        await conn.run_sync(Base.metadata.create_all)
    yield engine
    async with engine.begin() as conn:
        await conn.run_sync(Base.metadata.drop_all)

@pytest.fixture
async def session(engine):
    async with AsyncSession(engine) as session:
        yield session
        await session.rollback()

@pytest.fixture
async def client(session):
    app = create_app()
    async with AsyncClient(app=app, base_url="http://test") as client:
        yield client

@pytest.fixture
async def auth_headers(client, session):
    """Returns headers with valid JWT"""
    user = await create_test_user(session)
    token = create_access_token(user.id)
    return {"Authorization": f"Bearer {token}"}
```

**Testing pyramid:**
- **Unit tests (70%):** Pure Python functions, service logic, no DB
- **Integration tests (25%):** API endpoints with real DB (test PostgreSQL container)
- **E2E tests (5%):** Full learning flows with real AI (gated, run on release)

---

# 3. Frontend Architecture

## 3.1 Technology Decisions

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Framework | Next.js 14 (App Router) | RSC for fast initial load, streaming, SEO, colocated data fetching |
| Language | TypeScript 5 | Strict mode, eliminates entire categories of bugs |
| Styling | Tailwind CSS + Shadcn/UI | Utility-first, zero runtime, accessible components |
| State | Zustand (global) + TanStack Query (server) | Minimal boilerplate, devtools, separation of concerns |
| Forms | React Hook Form + Zod | Performant validation, TS inference, minimal re-renders |
| HTTP | TanStack Query + Axios | Caching, background refetch, optimistic updates |
| Icons | Lucide React | Tree-shakeable, consistent design |
| Charts | Recharts | React-native, composable, accessible |
| Rich Text | Tiptap | ProseMirror-based, extensible, collaborative-ready |

---

## 3.2 Next.js Folder Structure

```
studyos-frontend/
├── app/
│   ├── (auth)/
│   │   ├── login/
│   │   │   └── page.tsx
│   │   ├── signup/
│   │   │   └── page.tsx
│   │   └── layout.tsx               # Minimal auth layout
│   │
│   ├── (app)/                       # Authenticated app shell
│   │   ├── layout.tsx               # App shell with sidebar + header
│   │   ├── dashboard/
│   │   │   ├── page.tsx             # Server Component - fetch dashboard data
│   │   │   └── _components/
│   │   │       ├── StudySchedule.tsx
│   │   │       ├── ProgressWidget.tsx
│   │   │       ├── RecentResources.tsx
│   │   │       └── AIRecommendations.tsx
│   │   │
│   │   ├── knowledge/
│   │   │   ├── page.tsx             # Resource library
│   │   │   ├── upload/
│   │   │   │   └── page.tsx
│   │   │   ├── [resourceId]/
│   │   │   │   ├── page.tsx         # Resource detail
│   │   │   │   ├── notes/
│   │   │   │   │   └── page.tsx
│   │   │   │   └── flashcards/
│   │   │   │       └── page.tsx
│   │   │   └── _components/
│   │   │       ├── ResourceGrid.tsx
│   │   │       ├── UploadDropzone.tsx
│   │   │       ├── ResourceCard.tsx
│   │   │       └── FolderTree.tsx
│   │   │
│   │   ├── tutor/
│   │   │   ├── page.tsx             # AI Tutor chat
│   │   │   └── [conversationId]/
│   │   │       └── page.tsx
│   │   │
│   │   ├── flashcards/
│   │   │   ├── page.tsx             # Deck library
│   │   │   ├── [deckId]/
│   │   │   │   ├── page.tsx         # Review mode
│   │   │   │   └── edit/
│   │   │   │       └── page.tsx
│   │   │   └── _components/
│   │   │       ├── FlashcardReviewer.tsx
│   │   │       └── DeckCard.tsx
│   │   │
│   │   ├── quiz/
│   │   ├── planner/
│   │   ├── calendar/
│   │   ├── tasks/
│   │   ├── analytics/
│   │   ├── knowledge-graph/
│   │   ├── research/
│   │   ├── workspace/
│   │   ├── notifications/
│   │   └── settings/
│   │       ├── profile/
│   │       │   └── page.tsx
│   │       ├── preferences/
│   │       │   └── page.tsx
│   │       └── billing/
│   │           └── page.tsx
│   │
│   ├── api/                         # Next.js API routes (thin proxies)
│   │   └── auth/
│   │       └── [...nextauth]/
│   │           └── route.ts
│   │
│   ├── globals.css
│   ├── layout.tsx                   # Root layout - fonts, providers
│   └── not-found.tsx
│
├── components/
│   ├── ui/                          # Shadcn components (auto-generated)
│   │   ├── button.tsx
│   │   ├── dialog.tsx
│   │   └── ...
│   │
│   ├── shared/                      # App-wide shared components
│   │   ├── AppSidebar.tsx
│   │   ├── AppHeader.tsx
│   │   ├── NotificationBell.tsx
│   │   ├── SearchBar.tsx
│   │   ├── UserAvatar.tsx
│   │   ├── EmptyState.tsx
│   │   ├── ErrorBoundary.tsx
│   │   ├── LoadingSkeleton.tsx
│   │   └── PageHeader.tsx
│   │
│   └── ai/                          # AI-specific components
│       ├── ChatMessage.tsx
│       ├── StreamingText.tsx
│       ├── CitationCard.tsx
│       └── AITypingIndicator.tsx
│
├── hooks/
│   ├── useWebSocket.ts              # WS connection hook
│   ├── useAIStream.ts               # Streaming AI response hook
│   ├── useAuth.ts                   # Auth state hook
│   ├── useDebounce.ts
│   └── useLocalStorage.ts
│
├── lib/
│   ├── api.ts                       # Axios instance + interceptors
│   ├── auth.ts                      # NextAuth config
│   ├── queryClient.ts               # TanStack Query client
│   ├── utils.ts                     # cn(), formatters
│   └── validators.ts                # Shared Zod schemas
│
├── stores/
│   ├── useAppStore.ts               # Global app state (sidebar, theme)
│   ├── useChatStore.ts              # Active chat state
│   ├── useFlashcardStore.ts         # Current review session
│   └── useNotificationStore.ts      # Real-time notification count
│
├── services/                        # API call functions (used by TanStack Query)
│   ├── knowledge.service.ts
│   ├── notes.service.ts
│   ├── flashcard.service.ts
│   ├── quiz.service.ts
│   ├── tutor.service.ts
│   ├── planner.service.ts
│   ├── analytics.service.ts
│   └── auth.service.ts
│
├── types/
│   ├── api.types.ts                 # API response types
│   ├── domain.types.ts              # Domain entity types
│   └── ui.types.ts                  # UI-specific types
│
├── public/
│   ├── fonts/
│   └── images/
│
├── next.config.ts
├── tailwind.config.ts
├── tsconfig.json
└── package.json
```

---

## 3.3 Server vs Client Component Strategy

**Rule:** Default to Server Components. Add `"use client"` only when you need:
- `useState`, `useEffect`, `useReducer`
- Browser APIs (localStorage, window)
- Event listeners
- TanStack Query hooks
- Zustand stores
- WebSocket connections

```typescript
// ✅ Server Component - knowledge/page.tsx
// Fetches data on server, no client JS bundle cost
export default async function KnowledgePage() {
    const resources = await getResources(); // Direct service call
    return (
        <div>
            <PageHeader title="Knowledge Hub" />
            <ResourceGrid resources={resources} />  {/* SC */}
            <UploadButton />                          {/* CC - needs interactivity */}
        </div>
    );
}

// ✅ Client Component - _components/UploadDropzone.tsx
"use client"
import { useState, useCallback } from "react";
import { useUploadResource } from "@/services/knowledge.service";

export function UploadDropzone() {
    const [isDragging, setIsDragging] = useState(false);
    const uploadMutation = useUploadResource();
    // ... dropzone logic
}
```

---

## 3.4 State Management Architecture

**TanStack Query** manages all server state (API data). **Zustand** manages ephemeral UI state.

```typescript
// services/knowledge.service.ts
import { useQuery, useMutation, useQueryClient } from "@tanstack/react-query";
import api from "@/lib/api";

export const knowledgeKeys = {
    all: ["knowledge"] as const,
    resources: () => [...knowledgeKeys.all, "resources"] as const,
    resource: (id: string) => [...knowledgeKeys.resources(), id] as const,
};

export function useResources() {
    return useQuery({
        queryKey: knowledgeKeys.resources(),
        queryFn: () => api.get<Resource[]>("/knowledge/resources").then(r => r.data),
        staleTime: 1000 * 60 * 5,  // 5 minutes
    });
}

export function useUploadResource() {
    const qc = useQueryClient();
    return useMutation({
        mutationFn: (file: File) => uploadResourceFile(file),
        onSuccess: () => {
            qc.invalidateQueries({ queryKey: knowledgeKeys.resources() });
        },
        // Optimistic update
        onMutate: async (file) => {
            await qc.cancelQueries({ queryKey: knowledgeKeys.resources() });
            const snapshot = qc.getQueryData(knowledgeKeys.resources());
            qc.setQueryData(knowledgeKeys.resources(), (old: Resource[]) => [
                ...old,
                { id: "temp", title: file.name, processing_status: "pending" }
            ]);
            return { snapshot };
        },
        onError: (_, __, ctx) => {
            qc.setQueryData(knowledgeKeys.resources(), ctx?.snapshot);
        },
    });
}

// stores/useAppStore.ts
import { create } from "zustand";
import { persist } from "zustand/middleware";

interface AppStore {
    sidebarOpen: boolean;
    theme: "light" | "dark" | "system";
    setSidebarOpen: (open: boolean) => void;
    setTheme: (theme: "light" | "dark" | "system") => void;
}

export const useAppStore = create<AppStore>()(
    persist(
        (set) => ({
            sidebarOpen: true,
            theme: "system",
            setSidebarOpen: (open) => set({ sidebarOpen: open }),
            setTheme: (theme) => set({ theme }),
        }),
        { name: "studyos-app-store" }
    )
);
```

---

## 3.5 AI Streaming Hook

```typescript
// hooks/useAIStream.ts
"use client"
import { useState, useRef, useCallback } from "react";

interface StreamOptions {
    onToken?: (token: string) => void;
    onDone?: (citations: Citation[]) => void;
    onError?: (error: string) => void;
}

export function useAIStream(options: StreamOptions = {}) {
    const [content, setContent] = useState("");
    const [isStreaming, setIsStreaming] = useState(false);
    const [citations, setCitations] = useState<Citation[]>([]);
    const wsRef = useRef<WebSocket | null>(null);
    const requestIdRef = useRef<string>("");

    const startStream = useCallback((query: string, contextIds: string[]) => {
        const requestId = crypto.randomUUID();
        requestIdRef.current = requestId;
        setContent("");
        setIsStreaming(true);
        setCitations([]);

        const ws = getWebSocketConnection(); // Singleton WS connection
        wsRef.current = ws;

        ws.send(JSON.stringify({
            type: "tutor_query",
            request_id: requestId,
            payload: { content: query, context_ids: contextIds, stream: true }
        }));

        ws.onmessage = (event) => {
            const msg = JSON.parse(event.data);
            if (msg.request_id !== requestId) return;

            if (msg.type === "token") {
                setContent(prev => prev + msg.payload.token);
                options.onToken?.(msg.payload.token);
            } else if (msg.type === "done") {
                setIsStreaming(false);
                setCitations(msg.payload.citations || []);
                options.onDone?.(msg.payload.citations || []);
            } else if (msg.type === "error") {
                setIsStreaming(false);
                options.onError?.(msg.payload.error);
            }
        };
    }, []);

    const stopStream = useCallback(() => {
        setIsStreaming(false);
        wsRef.current?.send(JSON.stringify({ type: "cancel", request_id: requestIdRef.current }));
    }, []);

    return { content, isStreaming, citations, startStream, stopStream };
}
```

---

## 3.6 Form Management

```typescript
// Example: Zod schema + React Hook Form
import { z } from "zod";
import { useForm } from "react-hook-form";
import { zodResolver } from "@hookform/resolvers/zod";

const CreateFlashcardSchema = z.object({
    front: z.string().min(1, "Front is required").max(500),
    back: z.string().min(1, "Back is required").max(2000),
    deck_id: z.string().uuid(),
    tags: z.array(z.string()).max(10).optional(),
    image_url: z.string().url().optional(),
});

type CreateFlashcardForm = z.infer<typeof CreateFlashcardSchema>;

export function CreateFlashcardDialog({ deckId }: { deckId: string }) {
    const form = useForm<CreateFlashcardForm>({
        resolver: zodResolver(CreateFlashcardSchema),
        defaultValues: { deck_id: deckId },
    });

    const { mutate, isPending } = useCreateFlashcard();

    return (
        <Form {...form}>
            <form onSubmit={form.handleSubmit(data => mutate(data))}>
                <FormField name="front" render={({ field }) => (
                    <FormItem>
                        <FormLabel>Front</FormLabel>
                        <FormControl><Textarea {...field} /></FormControl>
                        <FormMessage />
                    </FormItem>
                )} />
                {/* ... more fields */}
                <Button type="submit" disabled={isPending}>
                    {isPending ? <Spinner /> : "Create Card"}
                </Button>
            </form>
        </Form>
    );
}
```

---

## 3.7 Theme System

```typescript
// tailwind.config.ts
export default {
    darkMode: ["class"],
    theme: {
        extend: {
            colors: {
                background: "hsl(var(--background))",
                foreground: "hsl(var(--foreground))",
                primary: {
                    DEFAULT: "hsl(var(--primary))",
                    foreground: "hsl(var(--primary-foreground))",
                },
                // StudyOS brand: indigo primary, emerald accent
                brand: {
                    50: "#eef2ff",
                    500: "#6366f1",
                    600: "#4f46e5",
                    700: "#4338ca",
                },
                accent: {
                    500: "#10b981",
                    600: "#059669",
                },
            },
            fontFamily: {
                sans: ["Inter var", "system-ui", "sans-serif"],
                mono: ["JetBrains Mono", "monospace"],
            },
        },
    },
}
```

