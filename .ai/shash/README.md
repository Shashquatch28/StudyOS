# AI Memory System

> **Purpose**
>
> This folder serves as the long-term memory for the StudyOS project.
>
> Every developer and every AI assistant (ChatGPT, Claude Code, Cursor, Codex, Gemini, etc.) should be able to understand the entire project by reading these files before writing code.
>
> The objective is to make development continuous, even when switching developers or AI models.

---

# Philosophy

The source code explains **how** the project works.

The AI Memory System explains:

* Why things were built
* What is currently happening
* What should happen next
* What problems already exist
* What decisions should never be repeated

If something is important enough to remember, it is important enough to document.

---

# Folder Structure

```text
AI_MEMORY/
│
├── README.md
├── PROJECT_STATE.md
├── DECISIONS_LOG.md
├── NEXT_STEPS.md
├── COMPLETED_WORK.md
├── BLOCKERS.md
├── MISTAKES.md
├── DATABASE_NOTES.md
├── AI_NOTES.md
└── CODEBASE_MAP.md
```

---

# File Responsibilities

## README.md

The documentation guide.

Contains:

* How the memory system works
* Update rules
* Development workflow
* AI workflow

Read first.

---

## PROJECT_STATE.md

The project's current snapshot.

Contains:

* Current milestone
* Current sprint
* Active branches
* Current focus
* Overall completion
* Latest updates

Think of this as the project's dashboard.

Update:

* Beginning of every session
* End of every session

---

## DECISIONS_LOG.md

Permanent record of engineering decisions.

Each entry should include:

* Date
* Decision
* Reason
* Alternatives considered
* Consequences

Never delete entries.

Append only.

---

## NEXT_STEPS.md

The active development queue.

Contains:

* Immediate tasks
* Current sprint goals
* Upcoming features
* Developer assignments

Every completed task should be removed or marked complete.

This is the primary working document.

---

## COMPLETED_WORK.md

Historical record of completed work.

Each entry should include:

* Feature
* Owner
* Completion date
* Related PR/Commit
* Notes

Useful for tracking project progress.

---

## BLOCKERS.md

Tracks everything preventing progress.

Examples:

* Waiting on API
* Library issue
* Infrastructure issue
* Design decision pending
* Bug preventing implementation

Each blocker should include:

* Description
* Owner
* Priority
* Current status

Delete entries once resolved.

---

## MISTAKES.md

Lessons learned.

Examples:

* Architecture mistakes
* Performance issues
* Debugging discoveries
* Poor implementation choices
* Things to avoid in the future

Purpose:

Never repeat the same mistake twice.

---

## DATABASE_NOTES.md

Database reference.

Contains:

* Schema decisions
* Migration history
* Naming conventions
* Relationships
* Indexes
* Query optimizations

Backend developers update this whenever the schema changes.

---

## AI_NOTES.md

Everything related to AI systems.

Examples:

* Prompt engineering
* LangGraph flows
* RAG improvements
* Embedding decisions
* Retrieval tuning
* Model routing
* Evaluation results

This becomes the knowledge base for the AI subsystem.

---

## CODEBASE_MAP.md

High-level architecture map.

Contains:

* Folder structure
* Module ownership
* System architecture
* Service relationships
* Data flow

This file helps new developers understand the project quickly.

---

# Daily Workflow

## Before Starting Work

Read the following files in order:

```text
README.md

↓

PROJECT_STATE.md

↓

NEXT_STEPS.md

↓

BLOCKERS.md

↓

CODEBASE_MAP.md (only if needed)
```

This should take less than 10 minutes.

---

## During Development

Whenever something important happens:

* Update the appropriate memory file immediately.
* Do not postpone documentation until later.

---

## After Completing Work

Update:

```text
PROJECT_STATE.md

↓

NEXT_STEPS.md

↓

COMPLETED_WORK.md

↓

(Any relevant notes file)
```

---

# Documentation Rules

## Rule 1

Every architectural decision must be documented.

→ DECISIONS_LOG.md

---

## Rule 2

Every completed feature must be documented.

→ COMPLETED_WORK.md

---

## Rule 3

Every blocker must be documented.

→ BLOCKERS.md

---

## Rule 4

Every major mistake must be documented.

→ MISTAKES.md

---

## Rule 5

Every database change must be documented.

→ DATABASE_NOTES.md

---

## Rule 6

Every AI-related improvement must be documented.

→ AI_NOTES.md

---

# AI Workflow

Before implementing anything, every AI assistant should:

1. Read README.md
2. Read PROJECT_STATE.md
3. Read NEXT_STEPS.md
4. Read BLOCKERS.md
5. Read CODEBASE_MAP.md (if unfamiliar with the architecture)

Only then begin implementation.

After completing work, the AI should update every relevant memory file before ending the session.

---

# Goal

This system should make it possible for:

* Any developer to onboard in minutes.
* Any AI assistant to continue development seamlessly.
* Architectural knowledge to never be lost.
* Development history to remain searchable.
* The project to continue even if team members change.

The codebase is the implementation.

**The AI Memory System is the project's memory.**
