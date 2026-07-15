# Development Workflow

This document defines the standard development workflow for DevPilot AI. Every feature developed for this project should follow this lifecycle.

## Phase 1 — Product Discovery

```text
Idea
  ↓
Vision
  ↓
Requirements
  ↓
Architecture
```

The goal is to fully understand the problem before implementation begins.

## Phase 2 — Planning

```text
Requirements
  ↓
Epic
  ↓
Stories
  ↓
Technical Design
```

Large features are broken into epics. Epics are broken into stories. Stories are small enough to be completed independently.

## Phase 3 — Implementation

For every story:

```text
Understand Story
  ↓
Review Architecture
  ↓
Review Existing Code
  ↓
Frontend Development + Backend Development
  ↓
API Integration
  ↓
Database Changes
  ↓
Manual Testing
  ↓
Documentation Update
  ↓
Pull Request
```

Frontend and backend should be developed together whenever possible. A story should deliver a complete user-facing feature.

## Phase 4 — Code Review

Every pull request should be reviewed for:

- Correctness
- Readability
- Maintainability
- Architecture compliance
- Documentation updates
- Testing

## Phase 5 — Deployment

After approval:

```text
Merge
  ↓
CI/CD
  ↓
Deployment
  ↓
Manual Verification
  ↓
Story Closed
```

## Definition of Done

A story is complete only if:

- Requirements are implemented.
- Acceptance criteria are satisfied.
- Code is reviewed.
- Tests pass.
- Documentation is updated.
- Manual verification is completed.

## Working Principles

- Prefer improving existing code instead of rewriting it.
- Avoid unnecessary abstractions.
- Avoid premature optimization.
- Prefer small, incremental changes over large rewrites.

## AI Workflow

Whenever an AI receives a task, it should:

1. Understand the story.
2. Review project context.
3. Review architecture.
4. Review existing implementation.
5. Produce a plan.
6. Implement changes.
7. Validate the implementation.
8. Suggest tests.
9. Update documentation if required.

Never skip directly to code generation without understanding the task.
