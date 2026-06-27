# BLOCKERS

> Active blockers that are preventing or slowing development.
>
> **Purpose**
>
> This file answers:
>
> **"What is currently preventing progress?"**
>
> Every issue that blocks implementation should be documented here before work is paused.
>
> This allows any developer or AI assistant to immediately understand what requires attention without rediscovering the problem.
>
> **Rules**
>
> - Only active blockers belong here.
> - Remove or archive blockers once resolved.
> - Include enough context for another developer to continue investigation.
> - Link relevant issues, PRs, or documentation whenever possible.
> - If multiple teams are involved, clearly assign ownership.

---

# Blocker Template

## BLOCKER-XXX

**Title**

Short description.

**Status**

🟢 Monitoring

🟡 Investigating

🔴 Blocked

⚪ Waiting

**Priority**

Critical

High

Medium

Low

**Owner**

Product Engineer

Platform Engineer

AI Engineer

Shared

**Created**

YYYY-MM-DD

**Description**

Detailed explanation of the issue.

**Impact**

What cannot proceed because of this blocker?

**Possible Solutions**

-

-

-

**Dependencies**

List any people, APIs, infrastructure, or decisions required.

**Related Files**

-

**Resolution Notes**

(To be completed when resolved.)

---

# Active Blockers

## None

There are currently no active blockers.

Project development is proceeding according to plan.

---

# Pending External Dependencies

The following are **not blockers**, but future dependencies that the Product Engineer should be aware of.

---

## Platform Infrastructure

**Owner**

Platform Engineer

**Status**

Pending

Dependencies

- Backend initialization
- FastAPI project
- PostgreSQL
- Redis
- Celery
- Docker
- OpenSearch
- pgvector

Impact

Frontend development should continue using mocked data and mocked API responses until backend services become available.

---

## Authentication API

**Owner**

Platform Engineer

**Status**

Pending

Dependencies

- JWT authentication
- Refresh token flow
- Session management
- User profile endpoint

Impact

Authentication UI can be completed using mocked services before backend integration.

---

## AI Services

**Owner**

Platform Engineer / AI Engineer

**Status**

Pending

Dependencies

- LangGraph
- LiteLLM
- Vector Search
- Retrieval Pipeline
- Citation Engine

Impact

AI interfaces should initially use mocked responses.

---

# Risk Register

The following are potential project risks that should be monitored.

---

## RISK-001

### Scope Expansion

Likelihood

Medium

Impact

High

Description

StudyOS contains many interconnected modules.

Without disciplined milestone planning, feature creep may delay the MVP.

Mitigation

- Follow roadmap strictly.
- Complete one module before starting another.
- Avoid implementing "nice-to-have" features before core functionality.

---

## RISK-002

### Tight Frontend / Backend Coupling

Likelihood

Medium

Impact

Medium

Description

Waiting for backend APIs could slow frontend development.

Mitigation

- Build against typed interfaces.
- Use mocked services.
- Keep API layer isolated.
- Replace mocks without changing UI components.

---

## RISK-003

### AI Cost Growth

Likelihood

Medium

Impact

Medium

Description

Running multiple LLM-powered workflows may become expensive as usage increases.

Mitigation

- Route requests through LiteLLM.
- Cache responses where appropriate.
- Prefer cheaper models for non-critical tasks.
- Monitor token usage and latency.

---

## RISK-004

### Architectural Drift

Likelihood

Low

Impact

High

Description

As contributors increase, developers may introduce inconsistent patterns that diverge from the agreed architecture.

Mitigation

- Follow the Architecture Blueprint.
- Record all architectural changes in `DECISIONS_LOG.md`.
- Keep `CODEBASE_MAP.md` updated.
- Review folder structure before introducing new modules.

---

# Escalation Process

If a blocker cannot be resolved within one development session:

1. Document the blocker here.
2. Record any attempted solutions.
3. Assign an owner.
4. Update `NEXT_STEPS.md`.
5. Continue with independent work where possible.
6. Revisit the blocker after dependencies are resolved.

---

# Resolution Workflow

When a blocker is resolved:

- Remove it from the Active Blockers section.
- Add a short summary under "Resolved Blockers".
- Update `COMPLETED_WORK.md` if significant work was completed.
- Record any lessons learned in `MISTAKES.md` if applicable.

---

# Resolved Blockers

None.

