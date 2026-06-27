# StudyOS — Product Engineer README
> Owner: Person B (Product / Frontend / User Experience)
>
> Your responsibility is to build everything the user sees and interacts with.
>
> Think like a Product Engineer, not just a Frontend Developer.
>
> Your job is to turn backend capabilities into an intuitive, polished learning experience.

---

# Philosophy

You own the product.

Every screen should feel complete.

Every interaction should feel intentional.

Whenever possible:

- Build reusable UI.
- Keep UX consistent.
- Don't duplicate components.
- Think from the user's perspective.

Your responsibility isn't to "make pages."

It's to build StudyOS.

---

# Core Responsibilities

You own

- NextJS
- React
- UI/UX
- Design System
- Dashboard
- Knowledge Hub UI
- AI Tutor UI
- Notes
- Flashcards
- Quiz
- Planner
- Calendar
- Analytics
- Notifications
- Workspace
- State Management

You DO NOT own

- Database
- Backend
- Authentication Logic
- AI Pipeline
- Search Infrastructure
- DevOps

---

# Folder Ownership

```
studyos-frontend/
```

Everything inside this folder belongs to you.

Exception:

Shared API contracts.

---

# Shared Documentation

```
studyos-docs/
```

Both developers contribute.

---

# Overall Roadmap

```
Foundation

↓

Authentication UI

↓

Dashboard

↓

Knowledge Hub

↓

Tutor

↓

Learning Tools

↓

Planner

↓

Analytics

↓

Polish
```

---

# Phase 0 — Foundation

Goal

Build the frontend foundation.

---

## Checklist

### NextJS

- [ ] Project setup
- [ ] TypeScript
- [ ] ESLint
- [ ] Prettier

---

### Styling

- [ ] Tailwind
- [ ] Shadcn
- [ ] Global Theme
- [ ] Typography
- [ ] Dark Mode

---

### Layout

- [ ] Sidebar
- [ ] Header
- [ ] Navigation
- [ ] Responsive Layout
- [ ] Loading Screens

---

### State

- [ ] Zustand
- [ ] TanStack Query
- [ ] API Client
- [ ] Authentication Store

---

### Components

- [ ] Button
- [ ] Card
- [ ] Modal
- [ ] Dialog
- [ ] Table
- [ ] Toast
- [ ] Empty State
- [ ] Skeleton
- [ ] Error State

---

# Deliverable

Complete frontend shell.

No backend required.

---

# Phase 1 — Authentication

## Checklist

### Pages

- [ ] Login
- [ ] Register
- [ ] Forgot Password
- [ ] Verify Email

---

### Components

- [ ] Google Login
- [ ] Email Login
- [ ] Forms
- [ ] Validation

---

### Auth Flow

- [ ] Protected Routes
- [ ] Session Handling
- [ ] Logout

---

# Deliverable

Authentication UI complete.

Waiting only for APIs.

---

# Phase 2 — Dashboard

## Checklist

- [ ] Dashboard Layout
- [ ] Welcome Card
- [ ] Recent Resources
- [ ] Today's Study
- [ ] Progress Widget
- [ ] Recommendations
- [ ] Activity Feed

---

# Deliverable

Dashboard ready.

Can consume backend APIs immediately.

---

# Phase 3 — Knowledge Hub

## Upload

- [ ] Upload Modal
- [ ] Drag & Drop
- [ ] Progress Bar
- [ ] Upload Queue

---

## Resources

- [ ] Resource Cards
- [ ] Folder Tree
- [ ] Search Bar
- [ ] Filters
- [ ] Sorting
- [ ] Resource Details

---

## States

- [ ] Processing
- [ ] Failed
- [ ] Ready

---

# Deliverable

Complete Knowledge Hub UI.

---

# Phase 4 — AI Tutor

## Checklist

### Chat

- [ ] Conversation List
- [ ] Chat Window
- [ ] Streaming Messages
- [ ] Markdown Rendering
- [ ] Code Blocks
- [ ] Citations
- [ ] Typing Indicator

---

### Workspace

- [ ] Resource Context
- [ ] Prompt Suggestions
- [ ] Conversation History

---

# Deliverable

Production-quality AI chat.

---

# Phase 5 — Learning Tools

## Notes

- [ ] Notes List
- [ ] Markdown Viewer
- [ ] Rich Editor
- [ ] Export Button

---

## Flashcards

- [ ] Deck List
- [ ] Review Screen
- [ ] Flip Animation
- [ ] Progress

---

## Quiz

- [ ] Quiz Generator
- [ ] Question Screen
- [ ] Timer
- [ ] Results
- [ ] Review Answers

---

# Deliverable

Complete learning experience.

---

# Phase 6 — Planner

## Planner

- [ ] Weekly View
- [ ] Monthly View
- [ ] Session Cards
- [ ] Drag & Drop

---

## Calendar

- [ ] Calendar View
- [ ] Events
- [ ] Deadlines
- [ ] Sync Status

---

## Tasks

- [ ] Task List
- [ ] Kanban
- [ ] Priorities
- [ ] Due Dates

---

# Deliverable

Planner complete.

---

# Phase 7 — Analytics

## Checklist

- [ ] Study Graphs
- [ ] Progress
- [ ] Streaks
- [ ] Heatmaps
- [ ] Mastery Dashboard
- [ ] AI Insights

---

# Deliverable

Analytics complete.

---

# UI Ownership

You own

```
Pages

Components

Navigation

State

Forms

Tables

Charts

Animations

Responsive Design

Accessibility

Theme

UX

Performance
```

---

# Communication Points

These require discussion BEFORE implementation.

---

## 1. API Contracts

Never assume an endpoint.

Backend and frontend must agree on

- Request Body
- Response Body
- Status Codes
- Error Responses

before coding.

---

## 2. Authentication Flow

Need agreement on

- Login
- Logout
- Refresh
- Session Management

---

## 3. WebSocket Events

Tutor UI depends on backend messages.

Need agreement on

- Event Types
- Payloads
- Streaming Format

---

## 4. Database-driven Features

Examples

- Flashcards
- Planner
- Analytics

Need agreement on

- Available fields
- Enums
- Sorting

---

## 5. Shared Types

Response Interfaces

Enums

Status Values

Need synchronization.

---

# Areas With Full Freedom

You DO NOT need approval for

- Component Architecture
- Folder Structure
- Styling
- UX
- Animations
- Responsiveness
- Accessibility
- State Management
- Custom Hooks
- Reusable Components
- Loading Screens
- Empty States
- Error States
- Theme Improvements

As long as API contracts remain unchanged.

---

# Before Starting Any Feature

Complete

```
Wireframe

↓

Component Breakdown

↓

API Requirements

↓

Reusable Components

↓

Implementation

↓

Testing

↓

Documentation
```

Never start with code.

Always start with UX.

---

# UI Standards

Every page should have

- [ ] Loading State
- [ ] Empty State
- [ ] Error State
- [ ] Success Feedback
- [ ] Responsive Layout
- [ ] Keyboard Accessibility
- [ ] Mobile Support

---

# Pull Request Checklist

- [ ] Types complete
- [ ] No console logs
- [ ] Responsive
- [ ] Accessible
- [ ] API integrated
- [ ] Components reusable
- [ ] Documentation updated

---

# Daily Workflow

Morning

- Pull latest changes
- Sync API changes
- Pick one feature

During Work

- Build reusable components first
- Build pages second
- Test responsiveness continuously

Evening

- Push branch
- Open PR
- Update documentation
- Review teammate's PR

---

# Documentation Requirements

Every completed feature updates

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
feature/dashboard

feature/sidebar

feature/knowledge-ui

feature/upload-ui

feature/tutor-ui

feature/flashcards

feature/quiz

feature/planner

feature/calendar

feature/analytics

fix/dashboard

fix/sidebar

refactor/components
```

---

# Definition of Done

A frontend feature is complete only if

- UI is complete
- Responsive
- Accessible
- API integrated
- Loading states implemented
- Error states implemented
- Empty states implemented
- Reusable components extracted
- Documentation updated
- PR approved

If any of these are missing,

**the feature is not complete.**