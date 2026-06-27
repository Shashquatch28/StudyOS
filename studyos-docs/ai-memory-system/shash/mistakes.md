# MISTAKES

> Lessons learned throughout the development of StudyOS.
>
> **Purpose**
>
> This file answers:
>
> **"What mistakes have we already made, and how do we avoid repeating them?"**
>
> Every engineering project accumulates knowledge through failures, debugging sessions, incorrect assumptions, and architectural revisions.
>
> This document ensures those lessons are preserved.
>
> **Rules**
>
> - Record the mistake, not the person.
> - Focus on the lesson rather than assigning blame.
> - Include both the root cause and the preventive measure.
> - Never delete entries.
> - If a mistake leads to an architectural decision, reference the corresponding ADR in `DECISIONS_LOG.md`.

---

# Entry Template

## MISTAKE-XXX

**Date**

YYYY-MM-DD

**Category**

Architecture

Frontend

Backend

Database

AI

Infrastructure

Deployment

Testing

Documentation

Performance

Security

Developer Workflow

---

### Problem

What happened?

---

### Root Cause

Why did it happen?

---

### Resolution

How was it fixed?

---

### Lesson Learned

What should future developers remember?

---

### Prevention

How do we avoid this happening again?

---

### Related Files

-

---

### Related Decision

(Optional)

Reference the ADR if one was created.

---

# Lessons Learned

## MISTAKE-001

**Date**

2026-06-27

**Category**

Project Architecture

### Problem

The initial repository structure attempted to create every planned directory upfront, including backend, infrastructure, Docker, and deployment folders before they were actually needed.

### Root Cause

Trying to mirror the final architecture immediately rather than allowing the repository to evolve naturally.

### Resolution

Only create folders when active development begins in that area.

Examples:

- `studyos-backend/` → when backend development starts.
- `studyos-infrastructure/` → when infrastructure work begins.
- `docker/` → when containerization is implemented.

### Lesson Learned

A clean repository should reflect the current state of development, not the final vision.

### Prevention

Avoid scaffolding unused directories.

Build the architecture incrementally.

---

## MISTAKE-002

**Date**

2026-06-27

**Category**

Frontend

### Problem

The default Next.js starter application included placeholder assets, example pages, and boilerplate that were unrelated to StudyOS.

### Root Cause

Generated templates are designed for demonstration purposes rather than production projects.

### Resolution

Removed all unnecessary assets before beginning feature development.

Replaced the starter homepage with a minimal placeholder.

### Lesson Learned

Always clean generated templates before writing production code.

### Prevention

Treat framework generators as starting points, not finished architectures.

---

## MISTAKE-003

**Date**

2026-06-27

**Category**

Developer Workflow

### Problem

Project documentation was originally scattered and lacked a clear hierarchy.

### Root Cause

Documentation structure evolved organically without a defined organization.

### Resolution

Reorganized documentation into:

studyos-docs/

- architecture/
- roles/

Maintained AI operational memory separately.

### Lesson Learned

Documentation architecture is as important as code architecture.

### Prevention

Keep long-term documentation organized by purpose rather than chronology.

---

## MISTAKE-004

**Date**

2026-06-27

**Category**

Project Planning

### Problem

There was a temptation to install libraries and start building features immediately before agreeing on engineering conventions.

### Root Cause

Skipping architectural decisions in favor of faster implementation.

### Resolution

Paused implementation to define:

- State management
- API strategy
- Folder structure
- Theme system
- Development workflow
- Design system

before writing application code.

### Lesson Learned

A few hours spent on architecture can save weeks of refactoring.

### Prevention

Always finalize foundational decisions before implementing features.

---

## MISTAKE-005

**Date**

2026-06-27

**Category**

Git Workflow

### Problem

Repository-level changes were staged from within the frontend directory, causing root-level changes to remain unstaged.

### Root Cause

Running `git add .` from a subdirectory stages changes relative to the current working directory.

### Resolution

Repository-wide staging should always be performed from the repository root.

### Lesson Learned

Always verify `git status` from the repository root before committing.

### Prevention

Repository operations should be performed from the project root.

---

# General Engineering Principles

These are recurring lessons that should guide future development.

---

## Keep Features Independent

Every module should own:

- Components
- Hooks
- Services
- Types
- Business logic

Avoid unnecessary coupling between modules.

---

## Build Reusable Components

If a component is used across multiple features, move it into:

components/common/

If it belongs to only one feature, keep it inside that feature.

---

## Do Not Prematurely Optimize

Build for clarity first.

Optimize only when profiling demonstrates a real performance issue.

---

## Never Skip Documentation

Every architectural decision should be recorded.

Future developers should never rely on chat history to understand the project.

---

## Avoid TODO Debt

Do not leave large TODO comments throughout the codebase.

Instead:

- Record work in `NEXT_STEPS.md`
- Track blockers in `BLOCKERS.md`
- Track technical debt explicitly

---

## Prefer Explicit Architecture

Favor clear folder structures, service boundaries, and typed interfaces over "clever" abstractions.

Readability is more valuable than brevity.

---

# Future Entries

Future mistakes worth documenting include:

- Performance bottlenecks
- Security issues
- Poor architectural choices
- Database migration problems
- Debugging discoveries
- Infrastructure incidents
- AI hallucination patterns
- Prompt engineering failures
- Deployment failures

Every significant lesson should be recorded here.

---

Last Updated:

YYYY-MM-DD

Updated By:

Developer / AI