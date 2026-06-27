
---

# 9. API Design

## 9.1 API Conventions

- Base URL: `https://api.studyos.io/api/v1`
- Response format: `{"data": {...}, "meta": {...}}` for success, `{"error": {"code": "...", "message": "..."}}` for errors
- Pagination: cursor-based for lists (prefer over offset for large datasets)
- All timestamps: ISO 8601 UTC (`2024-01-15T10:30:00Z`)
- IDs: UUID v4 strings
- Soft-deleted resources return 404 (not 410)

---

## 9.2 Complete API Reference

### Authentication

| Method | Route | Auth | Description |
|--------|-------|------|-------------|
| POST | `/auth/google` | None | Google OAuth sign-in/sign-up |
| POST | `/auth/email/register` | None | Email registration |
| POST | `/auth/email/login` | None | Email login |
| POST | `/auth/refresh` | Refresh Token | Issue new access token |
| POST | `/auth/logout` | Bearer | Revoke refresh token |
| POST | `/auth/email/verify` | None | Verify email OTP |
| POST | `/auth/password/reset-request` | None | Send reset email |
| POST | `/auth/password/reset` | Reset Token | Set new password |

**POST /auth/google**
```json
// Request
{ "google_id_token": "eyJ..." }

// Response 200
{
  "data": {
    "access_token": "eyJ...",
    "refresh_token": "ref_...",
    "expires_in": 900,
    "user": {
      "id": "uuid",
      "email": "user@example.com",
      "display_name": "Jane Doe",
      "onboarding_done": false
    }
  }
}

// Response 400
{
  "error": {
    "code": "INVALID_GOOGLE_TOKEN",
    "message": "The provided Google ID token is invalid or expired"
  }
}
```

---

### Knowledge Hub

| Method | Route | Auth | Description |
|--------|-------|------|-------------|
| POST | `/knowledge/resources/upload-initiate` | Bearer | Get S3 presigned URL |
| POST | `/knowledge/resources/{id}/upload-confirm` | Bearer | Trigger processing |
| GET | `/knowledge/resources` | Bearer | List resources |
| GET | `/knowledge/resources/{id}` | Bearer | Resource detail |
| PATCH | `/knowledge/resources/{id}` | Bearer | Update metadata |
| DELETE | `/knowledge/resources/{id}` | Bearer | Soft delete |
| POST | `/knowledge/resources/{id}/move` | Bearer | Move to folder |
| GET | `/knowledge/folders` | Bearer | Folder tree |
| POST | `/knowledge/folders` | Bearer | Create folder |
| PATCH | `/knowledge/folders/{id}` | Bearer | Rename/recolor |
| DELETE | `/knowledge/folders/{id}` | Bearer | Delete folder |

**POST /knowledge/resources/upload-initiate**
```json
// Request
{
  "filename": "chemistry_notes.pdf",
  "content_type": "application/pdf",
  "file_size_bytes": 5242880,
  "folder_id": "uuid-optional"
}

// Response 200
{
  "data": {
    "resource_id": "uuid",
    "upload_url": "https://s3.amazonaws.com/...",
    "upload_fields": {
      "key": "resources/uuid/chemistry_notes.pdf",
      "Content-Type": "application/pdf",
      "X-Amz-Signature": "..."
    },
    "expires_at": "2024-01-15T11:30:00Z"
  }
}
```

**GET /knowledge/resources**
```
Query params:
  - folder_id: UUID (optional)
  - file_type: pdf|docx|... (optional)
  - status: pending|processing|ready|failed (optional)
  - search: string (optional, fulltext)
  - page: int (default: 1)
  - size: int (default: 20, max: 100)
  - sort: created_at|title|updated_at (default: created_at)
  - order: asc|desc (default: desc)
  - cursor: string (cursor-based pagination)
```

```json
// Response 200
{
  "data": [
    {
      "id": "uuid",
      "title": "Chemistry Notes",
      "file_type": "pdf",
      "processing_status": "ready",
      "chunk_count": 45,
      "word_count": 12500,
      "topics": ["Organic Chemistry", "Reactions"],
      "created_at": "2024-01-15T10:00:00Z",
      "updated_at": "2024-01-15T10:05:00Z"
    }
  ],
  "meta": {
    "total": 142,
    "page": 1,
    "size": 20,
    "next_cursor": "eyJ..."
  }
}
```

---

### Notes

| Method | Route | Auth | Description |
|--------|-------|------|-------------|
| POST | `/notes/generate` | Bearer | Generate note (async) |
| GET | `/notes/jobs/{job_id}` | Bearer | Check generation status |
| GET | `/notes` | Bearer | List notes |
| GET | `/notes/{id}` | Bearer | Note detail |
| PUT | `/notes/{id}` | Bearer | Update note content |
| DELETE | `/notes/{id}` | Bearer | Soft delete |
| POST | `/notes/{id}/export` | Bearer | Export to PDF/DOCX |

**POST /notes/generate**
```json
// Request
{
  "resource_id": "uuid",
  "note_type": "summary",
  // Options: summary|chapter_summary|cheat_sheet|formula_sheet|
  //          key_takeaways|mind_map|timeline|beginner|exam|interview
  "chapter_range": [1, 3],      // Optional: specific chapters
  "additional_instructions": "Focus on reaction mechanisms"
}

// Response 202 Accepted
{
  "data": {
    "job_id": "uuid",
    "status": "queued",
    "estimated_time_secs": 30
  }
}

// GET /notes/jobs/{job_id} Response - in progress
{
  "data": {
    "job_id": "uuid",
    "status": "processing",
    "progress_percent": 45
  }
}

// GET /notes/jobs/{job_id} Response - done
{
  "data": {
    "job_id": "uuid",
    "status": "completed",
    "note_id": "uuid"
  }
}
```

---

### Flashcards

| Method | Route | Auth | Description |
|--------|-------|------|-------------|
| POST | `/flashcards/decks/generate` | Bearer | AI generate deck |
| POST | `/flashcards/decks` | Bearer | Create manual deck |
| GET | `/flashcards/decks` | Bearer | List decks |
| GET | `/flashcards/decks/{id}` | Bearer | Deck with cards |
| DELETE | `/flashcards/decks/{id}` | Bearer | Delete deck |
| POST | `/flashcards/decks/{id}/cards` | Bearer | Add card to deck |
| PUT | `/flashcards/cards/{id}` | Bearer | Edit card |
| DELETE | `/flashcards/cards/{id}` | Bearer | Delete card |
| GET | `/flashcards/review/due` | Bearer | Cards due today |
| POST | `/flashcards/cards/{id}/review` | Bearer | Submit review |

**POST /flashcards/cards/{id}/review**
```json
// Request
{
  "rating": 3,          // 1=Again, 2=Hard, 3=Good, 4=Easy
  "response_time_ms": 4500
}

// Response 200
{
  "data": {
    "card_id": "uuid",
    "new_interval_days": 6.0,
    "new_ease_factor": 2.5,
    "next_review_at": "2024-01-21T10:00:00Z"
  }
}
```

---

### Quizzes

| Method | Route | Auth | Description |
|--------|-------|------|-------------|
| POST | `/quizzes/generate` | Bearer | AI generate quiz |
| GET | `/quizzes` | Bearer | List quizzes |
| GET | `/quizzes/{id}` | Bearer | Quiz + questions |
| POST | `/quizzes/{id}/attempts` | Bearer | Start attempt |
| GET | `/quizzes/attempts/{id}` | Bearer | Current attempt |
| POST | `/quizzes/attempts/{id}/answer` | Bearer | Submit single answer |
| POST | `/quizzes/attempts/{id}/submit` | Bearer | Finish + get score |
| GET | `/quizzes/attempts/{id}/review` | Bearer | Detailed review |

---

### AI Tutor

| Method | Route | Auth | Description |
|--------|-------|------|-------------|
| POST | `/tutor/conversations` | Bearer | Create conversation |
| GET | `/tutor/conversations` | Bearer | List conversations |
| GET | `/tutor/conversations/{id}` | Bearer | Conversation detail |
| GET | `/tutor/conversations/{id}/messages` | Bearer | Message history |
| DELETE | `/tutor/conversations/{id}` | Bearer | Delete conversation |

**Note:** Actual messaging happens via WebSocket, not REST.

---

### Analytics

| Method | Route | Auth | Description |
|--------|-------|------|-------------|
| GET | `/analytics/summary` | Bearer | Overall stats |
| GET | `/analytics/study-time` | Bearer | Time trend |
| GET | `/analytics/mastery` | Bearer | Mastery by topic |
| GET | `/analytics/quiz-performance` | Bearer | Quiz score trends |
| GET | `/analytics/streak` | Bearer | Streak data |

**GET /analytics/summary**
```json
{
  "data": {
    "total_study_minutes": 1240,
    "current_streak_days": 14,
    "longest_streak_days": 21,
    "resources_count": 42,
    "notes_generated": 18,
    "cards_reviewed": 324,
    "quizzes_completed": 7,
    "avg_quiz_score": 0.78,
    "mastery_overview": {
      "mastered": 15,
      "learning": 28,
      "needs_review": 9
    }
  }
}
```

---

### Search

| Method | Route | Auth | Description |
|--------|-------|------|-------------|
| POST | `/search` | Bearer | Hybrid search |
| GET | `/search/suggest` | Bearer | Autocomplete |

**POST /search**
```json
// Request
{
  "query": "how does gradient descent work",
  "filters": {
    "resource_ids": ["uuid1", "uuid2"],
    "file_types": ["pdf", "docx"],
    "topics": ["Machine Learning"]
  },
  "page": 1,
  "size": 10
}

// Response 200
{
  "data": {
    "results": [
      {
        "chunk_id": "uuid",
        "resource_id": "uuid",
        "resource_title": "ML Fundamentals",
        "content": "Gradient descent is an optimization algorithm...",
        "section_title": "Chapter 3: Optimization",
        "page_number": 47,
        "relevance_score": 0.94,
        "highlight": "...optimization algorithm that iteratively moves toward the <mark>minimum</mark>..."
      }
    ],
    "total": 23,
    "page": 1,
    "size": 10
  },
  "meta": {
    "search_id": "uuid",
    "latency_ms": 145
  }
}
```

---

### Planner

| Method | Route | Auth | Description |
|--------|-------|------|-------------|
| POST | `/planner/plans/generate` | Bearer | AI generate plan |
| GET | `/planner/plans` | Bearer | List plans |
| GET | `/planner/plans/{id}` | Bearer | Plan detail |
| GET | `/planner/sessions` | Bearer | Sessions (date range) |
| POST | `/planner/sessions/{id}/start` | Bearer | Start session |
| POST | `/planner/sessions/{id}/complete` | Bearer | Complete session |
| POST | `/planner/plans/{id}/reschedule` | Bearer | AI reschedule |

---

### Users & Profile

| Method | Route | Auth | Description |
|--------|-------|------|-------------|
| GET | `/users/me` | Bearer | Current user |
| PATCH | `/users/me` | Bearer | Update profile |
| GET | `/users/me/profile` | Bearer | Learning profile |
| PUT | `/users/me/profile` | Bearer | Update learning profile |
| DELETE | `/users/me` | Bearer | Delete account |

---

# 10. Authentication & Authorization

## 10.1 JWT Strategy

**Access Token:** HS256, 15-minute TTL, stored in memory (not localStorage).
**Refresh Token:** Opaque string (UUID), hashed in DB, 30-day TTL, stored in httpOnly cookie.

```python
# core/security.py
from jose import jwt, JWTError
from passlib.context import CryptContext
from datetime import datetime, timedelta
import secrets, hashlib

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

def create_access_token(user_id: str, role: str) -> str:
    payload = {
        "sub": user_id,
        "role": role,
        "type": "access",
        "iat": datetime.utcnow(),
        "exp": datetime.utcnow() + timedelta(minutes=settings.ACCESS_TOKEN_EXPIRE_MINUTES),
        "jti": secrets.token_hex(16),  # JWT ID for revocation
    }
    return jwt.encode(payload, settings.JWT_SECRET_KEY, algorithm=settings.JWT_ALGORITHM)

def create_refresh_token() -> tuple[str, str]:
    """Returns (raw_token, hashed_token)"""
    raw = secrets.token_urlsafe(64)
    hashed = hashlib.sha256(raw.encode()).hexdigest()
    return raw, hashed

def verify_access_token(token: str) -> dict:
    try:
        payload = jwt.decode(token, settings.JWT_SECRET_KEY, algorithms=[settings.JWT_ALGORITHM])
        if payload.get("type") != "access":
            raise InvalidTokenError()
        return payload
    except JWTError:
        raise InvalidTokenError()
```

---

## 10.2 RBAC Design

**Roles:**

| Role | Description | Access |
|------|-------------|--------|
| `student` | Default user role | Own resources only |
| `pro` | Paid user | Advanced AI features, more storage |
| `admin` | Platform admin | All users, analytics, config |
| `institution_admin` | Future: school admin | Institution users |

```python
# core/permissions.py
from enum import Enum

class Permission(str, Enum):
    # Knowledge
    RESOURCE_CREATE = "resource:create"
    RESOURCE_READ = "resource:read"
    RESOURCE_DELETE = "resource:delete"

    # AI
    AI_TUTOR = "ai:tutor"
    AI_NOTES = "ai:notes"
    AI_UNLIMITED = "ai:unlimited"

    # Admin
    USER_MANAGE = "user:manage"
    ANALYTICS_VIEW_ALL = "analytics:view_all"

ROLE_PERMISSIONS = {
    "student": [
        Permission.RESOURCE_CREATE, Permission.RESOURCE_READ, Permission.RESOURCE_DELETE,
        Permission.AI_TUTOR, Permission.AI_NOTES,
    ],
    "pro": [
        # All student permissions +
        Permission.AI_UNLIMITED,
    ],
    "admin": list(Permission),  # All permissions
}

def require_permission(permission: Permission):
    async def dependency(current_user: User = Depends(get_current_user)):
        user_permissions = ROLE_PERMISSIONS.get(current_user.role, [])
        if permission not in user_permissions:
            raise ForbiddenError(f"Permission '{permission}' required")
        return current_user
    return dependency
```

---

## 10.3 Google OAuth Flow

```python
# modules/auth/service.py
from google.oauth2 import id_token
from google.auth.transport import requests as google_requests

async def google_sign_in(self, google_id_token: str) -> AuthResult:
    # Verify token with Google
    try:
        idinfo = id_token.verify_oauth2_token(
            google_id_token,
            google_requests.Request(),
            settings.GOOGLE_CLIENT_ID
        )
    except ValueError:
        raise InvalidTokenError("Invalid Google ID token")

    email = idinfo["email"]
    google_sub = idinfo["sub"]
    name = idinfo.get("name", email.split("@")[0])
    avatar = idinfo.get("picture")

    # Upsert user
    user = await self.user_repo.get_by_email(email)
    if not user:
        user = await self.user_repo.create(User(
            email=email,
            display_name=name,
            avatar_url=avatar,
            auth_provider="google",
            is_verified=True,
        ))
        await self.oauth_repo.create(OAuthAccount(
            user_id=user.id,
            provider="google",
            provider_id=google_sub,
        ))

    # Issue tokens
    access_token = create_access_token(str(user.id), user.role)
    raw_refresh, hashed_refresh = create_refresh_token()
    await self.auth_repo.store_refresh_token(user.id, hashed_refresh, days=30)

    return AuthResult(access_token=access_token, refresh_token=raw_refresh, user=user)
```

---

# 11. Security Architecture

## 11.1 Encryption

| Data | Encryption | Storage |
|------|-----------|---------|
| Passwords | bcrypt (cost=12) | PostgreSQL |
| Refresh tokens | SHA-256 hash (raw in cookie only) | PostgreSQL |
| S3 files | AES-256 SSE-S3 | AWS S3 |
| Database at rest | AWS RDS encryption | AWS KMS |
| Secrets | AWS Secrets Manager | Never in env files in production |
| PII in transit | TLS 1.3 | All connections |

---

## 11.2 File Validation

```python
# modules/knowledge/validators.py
import magic  # python-magic for MIME detection
from pathlib import Path

ALLOWED_MIME_TYPES = {
    "application/pdf": "pdf",
    "application/vnd.openxmlformats-officedocument.wordprocessingml.document": "docx",
    "application/vnd.openxmlformats-officedocument.presentationml.presentation": "pptx",
    "image/jpeg": "image",
    "image/png": "image",
    "image/webp": "image",
    "audio/mpeg": "audio",
    "audio/wav": "audio",
    "video/mp4": "video",
}

MAX_FILE_SIZE_BYTES = 100 * 1024 * 1024  # 100MB

async def validate_uploaded_file(s3_key: str) -> str:
    """Download first 8KB to detect MIME type (never trust Content-Type header)"""
    header_bytes = await s3_client.get_object_range(s3_key, 0, 8192)
    detected_mime = magic.from_buffer(header_bytes, mime=True)

    if detected_mime not in ALLOWED_MIME_TYPES:
        raise ValidationError(f"File type '{detected_mime}' is not allowed")

    # Check file size
    file_size = await s3_client.get_object_size(s3_key)
    if file_size > MAX_FILE_SIZE_BYTES:
        raise ValidationError("File exceeds 100MB limit")

    return ALLOWED_MIME_TYPES[detected_mime]
```

---

## 11.3 Prompt Injection Protection

```python
# ai/security/prompt_guard.py
INJECTION_PATTERNS = [
    r"ignore previous instructions",
    r"you are now",
    r"system prompt",
    r"disregard your",
    r"act as",
    r"pretend to be",
    r"forget everything",
    r"new personality",
    r"DAN mode",
    r"jailbreak",
]

def sanitize_user_input(text: str) -> str:
    """Basic prompt injection detection and sanitization"""
    import re
    lower = text.lower()

    for pattern in INJECTION_PATTERNS:
        if re.search(pattern, lower):
            # Log suspicious input for review
            logger.warning("Potential prompt injection detected", input_preview=text[:100])
            raise ValidationError("Your message contains content that cannot be processed")

    # Truncate to max input length
    return text[:4000]

# In prompt assembly, always use strict boundaries
def build_tutor_prompt(query: str, context: str) -> list[dict]:
    # User input is always in USER role, never injected into system prompt
    return [
        {"role": "system", "content": TUTOR_SYSTEM_PROMPT},
        {"role": "user", "content": f"Context:\n{context}\n\nQuestion: {query}"}
    ]
    # Never: f"System: {user_provided_text}"
```

---

## 11.4 Tenant Isolation

For MVP (single-tenant SaaS), isolation is enforced at the application layer:
- Every database query includes `WHERE user_id = {current_user.id}`
- The `BaseRepository.get_or_raise(id, user_id)` pattern ensures cross-user data access returns 404
- No shared data structures between users except reference tables

For future multi-tenancy, migrate to PostgreSQL Row-Level Security:
```sql
ALTER TABLE resources ENABLE ROW LEVEL SECURITY;
CREATE POLICY resources_isolation ON resources
    USING (user_id = current_setting('app.current_user_id')::uuid);
```

---

## 11.5 Security Headers

```python
# core/middleware.py
from fastapi.middleware.cors import CORSMiddleware
from starlette.middleware.base import BaseHTTPMiddleware

class SecurityHeadersMiddleware(BaseHTTPMiddleware):
    async def dispatch(self, request, call_next):
        response = await call_next(request)
        response.headers["X-Content-Type-Options"] = "nosniff"
        response.headers["X-Frame-Options"] = "DENY"
        response.headers["X-XSS-Protection"] = "1; mode=block"
        response.headers["Strict-Transport-Security"] = "max-age=31536000; includeSubDomains"
        response.headers["Referrer-Policy"] = "strict-origin-when-cross-origin"
        response.headers["Permissions-Policy"] = "camera=(), microphone=(), geolocation=()"
        return response
```

---

## 11.6 Audit Logging

```python
# core/audit.py
class AuditLogger:
    """Log all sensitive actions for compliance and security review"""

    async def log(
        self,
        user_id: str,
        action: str,
        resource_type: str,
        resource_id: str | None = None,
        details: dict | None = None,
        ip_address: str | None = None,
    ):
        # Write to separate audit_logs table (append-only, never deleted)
        await self.db.execute("""
            INSERT INTO audit_logs (user_id, action, resource_type, resource_id, details, ip_address)
            VALUES ($1, $2, $3, $4, $5, $6)
        """, user_id, action, resource_type, resource_id, details, ip_address)

# Usage
await audit_logger.log(
    user_id=current_user.id,
    action="resource.delete",
    resource_type="Resource",
    resource_id=str(resource_id),
    ip_address=request.client.host,
)
```

---

# 12. DevOps & Infrastructure

## 12.1 Docker Architecture

```dockerfile
# docker/Dockerfile.api
FROM python:3.12-slim AS builder
WORKDIR /app
RUN pip install poetry==1.8.0
COPY pyproject.toml poetry.lock ./
RUN poetry config virtualenvs.in-project true
RUN poetry install --only=main --no-dev

FROM python:3.12-slim AS runtime
WORKDIR /app
COPY --from=builder /app/.venv /app/.venv
COPY app/ ./app/
COPY alembic/ ./alembic/
COPY alembic.ini ./

ENV PATH="/app/.venv/bin:$PATH"
ENV PYTHONPATH="/app"

EXPOSE 8000
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000", \
     "--workers", "4", "--loop", "uvloop", "--log-config", "logging.json"]
```

```yaml
# docker-compose.yml (Development)
version: "3.9"
services:
  api:
    build:
      context: .
      dockerfile: docker/Dockerfile.api
    ports:
      - "8000:8000"
    env_file: .env.development
    volumes:
      - ./app:/app/app  # Hot reload
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
    command: uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload

  worker:
    build:
      context: .
      dockerfile: docker/Dockerfile.worker
    env_file: .env.development
    command: celery -A app.workers.celery_app worker -Q knowledge,embedding -c 4 -l info
    depends_on:
      - api

  beat:
    build:
      context: .
      dockerfile: docker/Dockerfile.worker
    env_file: .env.development
    command: celery -A app.workers.celery_app beat -l info --scheduler redbeat.RedBeatScheduler
    depends_on:
      - redis

  flower:
    image: mher/flower
    ports:
      - "5555:5555"
    environment:
      CELERY_BROKER_URL: redis://redis:6379/0

  postgres:
    image: pgvector/pgvector:pg16
    ports:
      - "5432:5432"
    environment:
      POSTGRES_DB: studyos
      POSTGRES_USER: studyos
      POSTGRES_PASSWORD: devpassword
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U studyos"]
      interval: 5s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    command: redis-server --maxmemory 256mb --maxmemory-policy allkeys-lru
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 5s
      retries: 3

  opensearch:
    image: opensearchproject/opensearch:2.11.0
    ports:
      - "9200:9200"
    environment:
      discovery.type: single-node
      plugins.security.disabled: "true"

volumes:
  postgres_data:
```

---

## 12.2 AWS Infrastructure

```
Production Infrastructure:

VPC (10.0.0.0/16)
├── Public Subnets (10.0.1.0/24, 10.0.2.0/24)
│   ├── ALB (Application Load Balancer)
│   └── NAT Gateway
└── Private Subnets (10.0.3.0/24, 10.0.4.0/24)
    ├── ECS Cluster
    │   ├── studyos-api (2 tasks, 0.5 vCPU, 1GB RAM each)
    │   ├── studyos-ws (2 tasks, 0.25 vCPU, 512MB RAM)
    │   ├── studyos-worker-knowledge (2 tasks, 1 vCPU, 2GB RAM)
    │   ├── studyos-worker-embedding (2 tasks, 0.5 vCPU, 1GB RAM)
    │   └── studyos-beat (1 task, 0.25 vCPU, 256MB RAM)
    ├── RDS PostgreSQL (db.t4g.medium, Multi-AZ)
    ├── ElastiCache Redis (cache.t4g.medium, cluster mode)
    └── OpenSearch Service (t3.small.search x2)

Edge:
├── CloudFront (CDN for Next.js static assets)
├── WAF (rate limiting, bot protection)
└── Route 53 (DNS)

Storage:
└── S3 (studyos-resources bucket)
    ├── Lifecycle: IA after 30 days, Glacier after 365 days
    └── CloudFront OAC for secure serving

Secrets:
└── AWS Secrets Manager (DB credentials, API keys)
```

---

## 12.3 CI/CD Pipeline

```yaml
# .github/workflows/deploy.yml
name: Deploy to Production

on:
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: pgvector/pgvector:pg16
        env:
          POSTGRES_DB: studyos_test
          POSTGRES_USER: test
          POSTGRES_PASSWORD: test
        ports: ["5432:5432"]
      redis:
        image: redis:7-alpine
        ports: ["6379:6379"]

    steps:
      - uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.12"

      - name: Install Poetry
        run: pip install poetry==1.8.0

      - name: Install dependencies
        run: poetry install

      - name: Run linting
        run: |
          poetry run ruff check app/
          poetry run mypy app/ --ignore-missing-imports

      - name: Run tests
        run: poetry run pytest tests/ -v --cov=app --cov-report=xml
        env:
          DATABASE_URL: postgresql+asyncpg://test:test@localhost/studyos_test
          REDIS_URL: redis://localhost:6379

      - name: Upload coverage
        uses: codecov/codecov-action@v4

  build-and-push:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::123456789:role/github-actions-deploy
          aws-region: us-east-1

      - name: Login to ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v2

      - name: Build and push API image
        env:
          REGISTRY: ${{ steps.login-ecr.outputs.registry }}
          IMAGE_TAG: ${{ github.sha }}
        run: |
          docker build -f docker/Dockerfile.api -t $REGISTRY/studyos-api:$IMAGE_TAG .
          docker push $REGISTRY/studyos-api:$IMAGE_TAG
          docker tag $REGISTRY/studyos-api:$IMAGE_TAG $REGISTRY/studyos-api:latest
          docker push $REGISTRY/studyos-api:latest

  deploy:
    needs: build-and-push
    runs-on: ubuntu-latest
    steps:
      - name: Run DB migrations
        run: |
          aws ecs run-task \
            --cluster studyos-prod \
            --task-definition studyos-migrate \
            --launch-type FARGATE \
            --wait

      - name: Deploy API service
        run: |
          aws ecs update-service \
            --cluster studyos-prod \
            --service studyos-api \
            --force-new-deployment \
            --deployment-configuration minimumHealthyPercent=100,maximumPercent=200

      - name: Deploy Worker service
        run: |
          aws ecs update-service \
            --cluster studyos-prod \
            --service studyos-worker \
            --force-new-deployment

      - name: Notify Slack
        uses: slackapi/slack-github-action@v1.25.0
        with:
          payload: '{"text": "✅ StudyOS deployed to production - ${{ github.sha }}"}'
```

---

## 12.4 Monitoring Stack

```yaml
# Prometheus scrape config
scrape_configs:
  - job_name: 'studyos-api'
    static_configs:
      - targets: ['api:8000']
    metrics_path: '/metrics'
    scrape_interval: 15s

  - job_name: 'celery'
    static_configs:
      - targets: ['flower:5555']
    metrics_path: '/metrics'
```

**Key Metrics to Monitor:**

| Metric | Alert Threshold | Action |
|--------|----------------|--------|
| API P99 latency | > 2s | Page on-call |
| Error rate (5xx) | > 1% over 5min | Page on-call |
| Celery queue depth (knowledge) | > 100 tasks | Scale workers |
| PostgreSQL connection pool | > 80% | Alert, scale |
| Redis memory | > 80% | Alert, add node |
| AI embedding latency | > 10s | Check OpenAI |
| LLM error rate | > 5% | Switch to fallback |

**Grafana Dashboards:**
1. System Health (API latency, error rates, pod health)
2. Business Metrics (DAU, resources uploaded, AI queries, flashcards reviewed)
3. AI Performance (token usage, cost per user, model latency, cache hit rate)
4. Infrastructure (RDS, Redis, S3, ECS)

---

## 12.5 Logging

```python
# core/logging.py
import structlog

def configure_logging():
    structlog.configure(
        processors=[
            structlog.stdlib.add_log_level,
            structlog.stdlib.add_logger_name,
            structlog.processors.TimeStamper(fmt="iso"),
            structlog.processors.StackInfoRenderer(),
            structlog.processors.format_exc_info,
            structlog.processors.JSONRenderer(),
        ],
        context_class=dict,
        logger_factory=structlog.stdlib.LoggerFactory(),
    )

# Usage throughout app
logger = structlog.get_logger()

# In request middleware
logger.info(
    "request_received",
    request_id=request.state.request_id,
    method=request.method,
    path=request.url.path,
    user_id=getattr(request.state, "user_id", None),
)

logger.info(
    "request_completed",
    request_id=request.state.request_id,
    status_code=response.status_code,
    duration_ms=elapsed,
)
```

**Log shipping:** CloudWatch Logs → CloudWatch Log Insights for ad-hoc queries, export to S3 for long-term retention.

---

# 13. Development Roadmap

## Phase 0 — Developer Environment Setup (Week 1–2)

**Objectives:** Every engineer can run the full stack locally in under 15 minutes.

**Work:**
- Repository setup (monorepo with `studyos-backend/` and `studyos-frontend/`)
- Docker Compose with all services
- `.env.example` files
- Makefile with common commands (`make dev`, `make test`, `make migrate`)
- Pre-commit hooks (ruff, mypy, prettier, eslint)
- GitHub Actions: lint + test on PRs
- Sentry setup (dev + prod DSNs)
- LangSmith tracing setup

**Deliverables:** `make dev` starts everything, `make test` runs all unit tests green.

---

## Phase 1 — Foundation (Weeks 3–8)

**Objectives:** Working authentication, file upload, and AI chat MVP.

**Database Work:**
- Alembic initial migration: `users`, `oauth_accounts`, `refresh_tokens`, `user_profiles`, `resources`, `folders`, `chunks`, `embeddings`, `conversations`, `messages`
- pgvector extension, HNSW index
- OpenSearch index creation script

**Backend Work:**
- FastAPI app factory, middleware, exception handlers
- Auth module (Google OAuth + email/password, JWT, refresh tokens)
- Users module (profile CRUD)
- Knowledge module: presigned URL upload, S3 integration
- Celery workers: knowledge processing pipeline (PDF → clean → chunk → embed)
- Basic RAG retriever (vector search only for MVP)
- LiteLLM proxy setup with GPT-4o + GPT-4o-mini
- AI Tutor (LangGraph, basic RAG, streaming WebSocket)
- Search module (basic vector search)

**Frontend Work:**
- Next.js project scaffolding, TypeScript config
- Auth pages (login, signup, Google OAuth button)
- App shell (sidebar, header, navigation)
- Dashboard skeleton
- Knowledge Hub: upload dropzone, resource list, processing status polling
- AI Tutor chat UI with streaming

**Testing:**
- Unit tests: auth service, knowledge service (>80% coverage)
- Integration tests: upload flow, auth endpoints

**Deliverable:** User can sign up, upload a PDF, and chat with an AI tutor about it.

**Completion Criteria:**
- [ ] Google OAuth login/logout works
- [ ] PDF uploaded → processed → searchable within 60 seconds
- [ ] AI Tutor answers questions with citations, streams in real-time
- [ ] Sentry captures errors
- [ ] All tests pass in CI

---

## Phase 2 — Learning Tools (Weeks 9–14)

**Objectives:** Full note generation, flashcards, and quizzes.

**Database Work:**
- New tables: `notes`, `flashcard_decks`, `flashcards`, `flashcard_reviews`, `quizzes`, `questions`, `quiz_attempts`

**Backend Work:**
- Notes module: async generation job, all note types, job polling API
- Flashcard module: AI generation, SM-2 spaced repetition, review API
- Quiz module: adaptive quiz generation, attempt tracking, scoring
- Improve RAG: add BM25 (OpenSearch), implement RRF, add cross-encoder re-ranking
- Knowledge Graph extraction agent
- Mastery Engine: update scores on review events

**Frontend Work:**
- Notes generation UI (type selector, generation modal, markdown viewer)
- Notes editor (Tiptap rich text)
- Flashcard deck list, card review UI (flip animation, SM-2 rating buttons)
- Quiz flow (question → answer → next, timer, completion screen)
- Resource detail page (notes, flashcards, quizzes tabs)

**Testing:**
- SM-2 algorithm unit tests
- Note generation integration tests with mocked LLM
- Quiz scoring logic unit tests

**Deliverable:** User can generate notes, review flashcards (with spaced repetition), and take quizzes from any uploaded resource.

---

## Phase 3 — Planning & Scheduling (Weeks 15–20)

**Objectives:** Study planner, calendar, tasks, and notifications.

**Database Work:**
- New tables: `study_plans`, `study_sessions`, `calendar_events`, `tasks`, `notifications`, `analytics_daily`

**Backend Work:**
- Planner module: AI plan generation (LangGraph), session management
- Calendar module: CRUD + Google Calendar OAuth sync
- Tasks module: CRUD, priority sorting, AI suggestions
- Notifications module: in-app, email (AWS SES), Celery Beat schedules
- Analytics module: daily aggregation job, summary API
- Pomodoro session tracking

**Frontend Work:**
- Study Planner UI (week/month view, drag-and-drop sessions)
- Calendar (FullCalendar integration, event creation, Google sync button)
- Tasks (kanban board, due dates, priority badges)
- Notifications bell (real-time via WebSocket)
- Daily study schedule on dashboard

**Testing:**
- Planner agent integration tests
- Calendar sync unit tests
- Notification delivery tests

**Deliverable:** User has an AI-generated study schedule, sees exams on calendar, gets reminders.

---

## Phase 4 — Intelligence (Weeks 21–28)

**Objectives:** Knowledge graph, mastery tracking, adaptive learning, analytics dashboard.

**Database Work:**
- New tables: `knowledge_nodes`, `knowledge_edges`, `mastery_records`

**Backend Work:**
- Knowledge Graph: build from resource extraction, update on new uploads
- Mastery Engine: forgetting curve modeling, revision scheduler
- Adaptive quiz difficulty (use mastery scores to select questions)
- Adaptive flashcard sorting (surface weak cards more often)
- Analytics: complete dashboard with trends, heatmap, predictions
- Research Agent: paper search (Semantic Scholar API), YouTube recommendations

**Frontend Work:**
- Knowledge Graph visualization (D3.js, interactive nodes)
- Mastery dashboard (topic grid, confidence heatmap)
- Analytics page (recharts: study time trends, mastery progression, quiz scores)
- Research Assistant page
- AI Workspace (general-purpose chat, code assistant)

**Testing:**
- Mastery algorithm unit tests
- Knowledge graph extraction accuracy tests
- Analytics aggregation tests

**Deliverable:** Platform adapts to the learner's performance, identifies weak areas, and recommends targeted review.

---

## Phase 5 — Platform & Scale (Weeks 29–40)

**Objectives:** Polish, performance, collaboration foundations, and production hardening.

**Work:**
- Performance: DB query optimization, N+1 elimination, connection pooling
- Caching: comprehensive Redis caching layer audit
- Mobile: React Native app (or PWA with offline support)
- User onboarding flow
- Subscription/billing (Stripe integration)
- Study Groups (shared resources, group flashcard decks)
- Public API (developer platform foundation)
- GDPR compliance (data export, account deletion)
- Comprehensive E2E tests

---

# 14. Project Structure

## 14.1 Repository Layout

```
studyos/                              # Monorepo root
├── .github/
│   ├── workflows/
│   │   ├── pr-checks.yml             # On PRs: lint, test
│   │   ├── deploy-staging.yml        # On develop branch merge
│   │   └── deploy-production.yml     # On main branch merge
│   ├── PULL_REQUEST_TEMPLATE.md
│   └── ISSUE_TEMPLATE/
│
├── studyos-backend/                  # FastAPI application
│   └── (see Section 2.2 for full structure)
│
├── studyos-frontend/                 # Next.js application
│   └── (see Section 3.2 for full structure)
│
├── studyos-infrastructure/           # IaC (future Terraform)
│   ├── terraform/
│   │   ├── modules/
│   │   │   ├── ecs/
│   │   │   ├── rds/
│   │   │   ├── elasticache/
│   │   │   └── s3/
│   │   ├── environments/
│   │   │   ├── staging/
│   │   │   └── production/
│   │   └── main.tf
│   └── scripts/
│       ├── bootstrap.sh              # First-time AWS setup
│       └── rotate-secrets.sh
│
├── studyos-docs/                     # Technical documentation
│   ├── adr/                          # Architecture Decision Records
│   │   ├── 001-modular-monolith.md
│   │   ├── 002-langgraph-agents.md
│   │   └── 003-pgvector-vs-pinecone.md
│   ├── api/                          # OpenAPI specs
│   └── runbooks/                     # Operational runbooks
│
├── docker-compose.yml                # Local development
├── docker-compose.test.yml           # CI testing
├── Makefile                          # Developer commands
└── README.md
```

---

# 15. Coding Standards

## 15.1 Python Standards

```python
# Naming conventions
class KnowledgeService:              # PascalCase for classes
    MAX_CHUNK_SIZE = 500             # SCREAMING_SNAKE for constants

    def __init__(self, repo: ResourceRepository):  # Type annotations required
        self.repo = repo

    async def get_resource_by_id(    # snake_case for methods
        self, resource_id: UUID, user_id: UUID
    ) -> Resource:                   # Return types required
        return await self.repo.get_or_raise(resource_id, user_id)
```

**Required:** `ruff` for linting + formatting (replaces black + flake8 + isort), `mypy` in strict mode for type safety.

```toml
# pyproject.toml
[tool.ruff]
line-length = 100
target-version = "py312"
select = ["E", "F", "I", "N", "UP", "B", "SIM"]

[tool.mypy]
strict = true
python_version = "3.12"
plugins = ["pydantic.mypy"]
```

---

## 15.2 TypeScript Standards

```typescript
// All types must be explicit - no implicit `any`
// Use interface for objects, type for unions
interface Resource {
    id: string;
    title: string;
    fileType: ResourceFileType;
    processingStatus: ProcessingStatus;
    createdAt: string;
}

type ResourceFileType = "pdf" | "docx" | "pptx" | "image" | "audio" | "video" | "url" | "youtube";
type ProcessingStatus = "pending" | "processing" | "ready" | "failed";

// Function components: explicit return type
export function ResourceCard({ resource }: { resource: Resource }): JSX.Element {
    // ...
}

// No default exports from service files - named exports only
export { useResources, useUploadResource, knowledgeKeys };
```

---

## 15.3 Git Workflow

**Branch Strategy:** GitHub Flow (simplified for small team, expand to GitFlow at 5+ engineers)

- `main` — production
- `develop` — staging (optional at team size < 5)
- `feature/issue-123-knowledge-upload` — feature branches
- `fix/issue-456-flashcard-sm2` — bug fixes

**Commit Convention** (Conventional Commits):
```
feat(knowledge): add PDF OCR with Google Document AI
fix(auth): resolve refresh token race condition
refactor(rag): extract hybrid retriever into separate class
docs(api): add OpenAPI examples for quiz endpoints
test(flashcards): add SM-2 algorithm edge case tests
chore(deps): upgrade langchain to 0.3.0
```

**PR Requirements:**
- Linked GitHub issue
- Test coverage maintained (>80%)
- All CI checks pass
- At least 1 review approval
- No `TODO` without a linked issue

---

## 15.4 API Response Standards

```python
# Every API response uses this structure
from pydantic import BaseModel
from typing import TypeVar, Generic

T = TypeVar("T")

class PaginationMeta(BaseModel):
    total: int
    page: int
    size: int
    next_cursor: str | None = None

class ApiResponse(BaseModel, Generic[T]):
    data: T
    meta: dict | None = None

class PaginatedResponse(BaseModel, Generic[T]):
    data: list[T]
    meta: PaginationMeta

# Error response
class ErrorDetail(BaseModel):
    code: str
    message: str
    field: str | None = None

class ErrorResponse(BaseModel):
    error: ErrorDetail
```

---

# 16. Scaling Strategy

## 16.1 MVP → 10,000 Users

**Infrastructure:** Single ECS cluster, RDS `db.t4g.medium`, ElastiCache `cache.t4g.small`

**Key actions:**
- All DB queries use indexes (audit `EXPLAIN ANALYZE` from day one)
- Redis caching for all list endpoints
- Celery workers autoscale 2–8 tasks on queue depth
- S3 + CloudFront for all static assets and processed file serving
- Connection pooling with PgBouncer (transaction mode)

---

## 16.2 10,000 → 100,000 Users

**Architecture changes:**
- Extract Knowledge Pipeline worker into separate ECS service with own scaling policy
- Add PostgreSQL read replica for analytics + search queries
- ElastiCache cluster mode (3 shards)
- OpenSearch dedicated master + 2 data nodes
- Implement request-level caching in Redis for all AI responses
- Add CDN caching for generated notes (read-heavy, rarely changed)
- Move to RDS `db.r7g.large`

**New infrastructure:**
- AWS SQS replaces Redis Streams for reliability at scale (at-least-once delivery)
- Separate RDS instance for analytics (TimescaleDB)

---

## 16.3 100,000 → 1,000,000 Users

**Architecture changes:**
- Extract AI Orchestrator into independent service (separate deployment, own Redis)
- Shard PostgreSQL by user_id prefix (using Citus or horizontal RDS partitioning)
- Separate pgvector database for embeddings only (query patterns are very different from OLTP)
- Move search to dedicated OpenSearch domain (production-sized)
- Add Kafka for event streaming (Redis Streams doesn't scale to millions of events/day reliably)
- Global CDN for international users
- Edge functions for auth token validation (sub-ms latency)

**New services:**
- Dedicated LiteLLM fleet (rate limit pooling across all users)
- Embedding cache service (similar embeddings → reuse)

---

## 16.4 1,000,000 → 10,000,000 Users

**Architecture changes:**
- Full microservices extraction along module boundaries
- Multi-region deployment (US + EU + APAC)
- Database per service (each microservice owns its schema)
- Event sourcing for mastery and analytics (append-only event log → derived read models)
- CQRS: separate read/write paths for high-traffic endpoints
- AI response caching at semantic level (similar queries → same response)
- Dedicated ML infrastructure for custom embedding models fine-tuned on educational content
- Real-time collaboration features (operational transforms or CRDTs)

---

# 17. Cost Optimization

## 17.1 LLM Cost Strategy

| Strategy | Savings | Implementation |
|----------|---------|----------------|
| LiteLLM prompt caching | 60-80% for repeated prompts | Enable in litellm_config.yaml |
| Model routing by complexity | 40-60% | gpt-4o-mini for classification/flashcards |
| Token budget enforcement | 20-30% | Context compression before LLM calls |
| Semantic response caching | 50-70% | Redis + embedding similarity check |
| Batch processing | 20% | Batch embedding generation (100 chunks/call) |

**Estimated cost at 10K users:** ~$500-1500/month on LLM (if aggressively cached)
**At 100K users:** ~$5,000-15,000/month → prioritize caching and model routing

---

## 17.2 Embedding Cost Optimization

```python
# Never re-embed content that hasn't changed
async def get_or_create_embedding(chunk_id: str, content: str) -> list[float]:
    # Check if embedding exists and content hash matches
    existing = await db.fetchrow(
        "SELECT embedding, content_hash FROM embeddings WHERE chunk_id = $1",
        chunk_id
    )
    content_hash = hashlib.sha256(content.encode()).hexdigest()
    if existing and existing["content_hash"] == content_hash:
        return existing["embedding"]

    # Generate new embedding
    embedding = await embedding_service.embed(content)
    await store_embedding(chunk_id, embedding, content_hash)
    return embedding
```

`text-embedding-3-large` costs $0.13/million tokens. At 10K users with 1000 chunks each → ~13M tokens → ~$1.69 one-time cost. Very low.

---

## 17.3 Storage Cost Optimization

```python
# S3 Lifecycle Policy (applied via Terraform)
lifecycle_rules = [
    {
        "id": "move-to-ia",
        "status": "Enabled",
        "transitions": [
            {"days": 30, "storage_class": "STANDARD_IA"},   # 45% cost reduction
            {"days": 365, "storage_class": "GLACIER"},       # 80% cost reduction
        ],
    },
    {
        "id": "delete-failed-uploads",
        "status": "Enabled",
        "expiration": {"days": 1},
        "filter": {"prefix": "temp/"},
    }
]
```

---

## 17.4 Database Cost Optimization

- Use RDS `db.t4g` (Graviton2) instances — 40% cheaper than x86
- Reserved instances for RDS (1-year, no upfront) — 40% discount
- Turn off dev/staging DB on nights + weekends (RDS stop/start)
- Partition old analytics data to S3 via Parquet for long-term storage

---

# 18. Future Architecture

## 18.1 Mobile Apps

**Strategy:** React Native with shared TypeScript types between web and mobile.

- Shared: `types/`, `lib/validators.ts`, Zod schemas
- Different: Platform-specific navigation (Expo Router), native file pickers, push notifications (Expo Notifications → FCM/APNs)
- Same API — no mobile-specific API changes needed
- Offline: SQLite (via Expo SQLite) for flashcard reviews + note viewing without connectivity

---

## 18.2 Multi-Tenancy (Institution Accounts)

**Database changes:**
```sql
-- Add tenant dimension
CREATE TABLE organizations (
    id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name        VARCHAR(255) NOT NULL,
    slug        VARCHAR(100) UNIQUE NOT NULL,
    plan        VARCHAR(50) DEFAULT 'institution',
    settings    JSONB DEFAULT '{}',
    created_at  TIMESTAMPTZ NOT NULL DEFAULT now()
);

ALTER TABLE users ADD COLUMN organization_id UUID REFERENCES organizations(id);

-- All queries gain: AND (user_id = $user_id OR organization_id = $org_id)
-- Shared resources within an organization
CREATE TABLE organization_resources (
    organization_id UUID REFERENCES organizations(id),
    resource_id     UUID REFERENCES resources(id),
    created_by      UUID REFERENCES users(id),
    PRIMARY KEY (organization_id, resource_id)
);
```

**Migration:** Switch to PostgreSQL RLS at this point. Too much complexity to maintain in-application at multi-tenant scale.

---

## 18.3 Marketplace

- **Data model:** Add `products`, `purchases`, `seller_profiles` tables
- **Sellable content:** Flashcard decks, quiz sets, study plans, curated note collections
- **Revenue share:** 70% creator, 30% platform (Stripe Connect for payouts)
- **Discovery:** OpenSearch-powered content search with curation ranking
- **Quality control:** AI-powered content review before listing

---

## 18.4 Plugin System

```python
# Future plugin architecture
class StudyOSPlugin(ABC):
    """Base class for all plugins"""
    name: str
    version: str
    author: str

    @abstractmethod
    async def on_resource_processed(self, resource_id: str, content: str) -> None:
        """Called after any resource is processed"""

    @abstractmethod
    async def register_tools(self) -> list[AgentTool]:
        """Register custom LangGraph tools for agent use"""
```

Plugins run in isolated containers (AWS Lambda or separate ECS tasks) with limited API access via scoped API keys.

---

## 18.5 Public APIs

When launching a developer platform:

```yaml
# Public API design
Base URL: https://api.studyos.io/public/v1

Rate limits:
  - Free tier: 100 requests/day
  - Developer: 10,000 requests/day
  - Business: 1,000,000 requests/day

Endpoints exposed:
  - GET /resources           # List resources (owned by API key's user)
  - POST /resources/upload   # Upload a resource
  - POST /chat               # Query the tutor
  - GET /flashcards          # List flashcards
  - POST /flashcards/review  # Submit a review
  - GET /analytics/summary   # Get user analytics

Authentication: API key in X-StudyOS-Key header
SDK: Python, TypeScript, JavaScript packages on PyPI/npm
```

---

## 18.6 Offline Support

**Web:** Service Worker (Workbox) for offline flashcard reviews. Cache flashcard deck data + images on deck open. Sync reviews when reconnected.

**Architecture:**
```typescript
// Background sync for offline reviews
const bgSync = new BackgroundSyncPlugin("flashcard-review-queue", {
    maxRetentionTime: 24 * 60, // Retain for 1 day
});

registerRoute(
    /\/api\/v1\/flashcards\/.*\/review/,
    new NetworkOnly({ plugins: [bgSync] }),
    "POST"
);
```

---

# Appendix A: Technology Reference

| Component | Technology | Version | Notes |
|-----------|-----------|---------|-------|
| Backend Framework | FastAPI | 0.115+ | With Starlette 0.41+ |
| Python Runtime | Python | 3.12+ | Type annotations, asyncio |
| ORM | SQLAlchemy | 2.0+ | Async mode required |
| Migrations | Alembic | 1.13+ | |
| Database | PostgreSQL | 16+ | pgvector extension |
| Vector Extension | pgvector | 0.7+ | HNSW support |
| Cache / Queue | Redis | 7+ | Cluster mode in production |
| Task Queue | Celery | 5.4+ | |
| Task Scheduler | Celery Beat / RedBeat | | Redis-backed beat |
| AI Orchestration | LangGraph | 0.2+ | |
| LLM Proxy | LiteLLM | 1.40+ | |
| Embedding | OpenAI text-embedding-3-large | — | 3072 dimensions |
| Frontend | Next.js | 14+ (App Router) | |
| Styling | Tailwind CSS | 3.4+ | |
| UI Components | Shadcn/UI | Latest | |
| State | Zustand | 4.5+ | |
| Data Fetching | TanStack Query | 5+ | |
| Forms | React Hook Form + Zod | 7+ | |
| Search Engine | OpenSearch | 2.11+ | BM25 + semantic |
| File Storage | AWS S3 | — | |
| OCR | Google Document AI | — | Primary |
| OCR fallback | Tesseract | 5+ | Open source |
| Speech-to-text | OpenAI Whisper | — | Audio/video |
| Observability | Prometheus + Grafana | — | |
| Error Tracking | Sentry | — | Backend + Frontend |
| AI Observability | LangSmith | — | Agent traces |
| CI/CD | GitHub Actions | — | |
| Containers | Docker + ECS | — | AWS Fargate |
| IaC | Terraform | 1.7+ | |

---

# Appendix B: Environment Variables Reference

```bash
# .env.example
# App
APP_NAME=StudyOS
DEBUG=false
ENVIRONMENT=production
SECRET_KEY=<generate with: openssl rand -hex 32>

# Database
DATABASE_URL=postgresql+asyncpg://user:pass@host:5432/studyos
DATABASE_POOL_SIZE=20
DATABASE_MAX_OVERFLOW=10

# Redis
REDIS_URL=redis://:password@host:6379/0

# AWS
AWS_REGION=us-east-1
AWS_ACCESS_KEY_ID=<from AWS Secrets Manager in production>
AWS_SECRET_ACCESS_KEY=<from AWS Secrets Manager in production>
S3_BUCKET_NAME=studyos-resources-prod

# Auth
JWT_SECRET_KEY=<generate with: openssl rand -hex 64>
JWT_ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=15
REFRESH_TOKEN_EXPIRE_DAYS=30
GOOGLE_CLIENT_ID=<from Google Cloud Console>
GOOGLE_CLIENT_SECRET=<from Google Cloud Console>

# AI
OPENAI_API_KEY=sk-...
ANTHROPIC_API_KEY=sk-ant-...
GOOGLE_AI_API_KEY=AIza...
LITELLM_MASTER_KEY=<generate random>
LANGSMITH_API_KEY=ls-...

# Celery
CELERY_BROKER_URL=redis://:password@host:6379/1
CELERY_RESULT_BACKEND=redis://:password@host:6379/2

# Search
OPENSEARCH_URL=https://host:9200
OPENSEARCH_INDEX_PREFIX=studyos-prod

# Email
AWS_SES_FROM_EMAIL=hello@studyos.io
AWS_SES_REGION=us-east-1

# Google APIs (Calendar, Document AI)
GOOGLE_DOCUMENT_AI_PROCESSOR_ID=<from GCP>
GOOGLE_CALENDAR_CLIENT_ID=<from Google Cloud Console>
```

---

*This document is the master engineering reference for StudyOS. It should be updated as architectural decisions evolve. All ADRs (Architecture Decision Records) should be documented in `studyos-docs/adr/` with date, context, decision, and consequences.*

*Last Updated: Initial version for 12-18 month build cycle.*
