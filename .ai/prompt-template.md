# DevPilot AI Prompt Template

## 1. Introduction

This document defines the reusable prompt framework for AI assistants contributing to DevPilot AI. It is designed for Gemini, GPT, Claude, and other capable assistants. It is not a single task prompt; copy the universal template, replace the bracketed fields, and remove sections that are genuinely not applicable.

The template supports feature development, bug fixes, refactoring, documentation, API and database work, testing, code review, and UI improvements. It reinforces Documentation First Development, Story Driven Development, the Modular Monolith, Domain Driven Design, and Feature Driven Development.

AI must understand the documented task and existing implementation before proposing or generating code. It must preserve module ownership, use the Knowledge Service as the sole provider of project context for AI workflows, and keep users responsible for material decisions and acceptance.

## 2. Usage Instructions

1. Start from the **Universal Prompt Template** below.
2. Replace every applicable `[placeholder]` with task-specific information. Delete optional fields that do not apply; do not leave ambiguous placeholders for the AI to guess.
3. Provide the story or issue first. If none exists, state that explicitly and ask the AI to identify the missing product/technical context before implementation.
4. Give the AI access to the relevant documents and implementation files. Use the document-loading priority below.
5. State the allowed and prohibited file scope. If the scope is unknown, instruct the AI to propose it before making changes.
6. Include observable acceptance criteria and proportionate testing expectations.
7. Ask for a concise plan before implementation when the task changes behavior, data, APIs, architecture, or more than a small localized file.

### Required document-loading order

Before coding, the AI must review context in this order:

1. The story, issue, or approved task definition.
2. `docs/architecture/architecture.md` and any applicable ADR or technical design.
3. `.ai/project-context.md`.
4. `.ai/coding-rules.md` and `.ai/architecture-rules.md`.
5. The relevant existing implementation, tests, API contracts, migrations, and shared packages.

Consult `docs/product/requirements.md`, `docs/product/vision.md`, and `.ai/workflow.md` when the task affects product behavior, scope, lifecycle, or delivery process. Where documents conflict, follow the source-of-truth order in `.ai/project-context.md`.

The AI must not skip directly to implementation. If the story, ownership, architecture, permissions, data model, or acceptance criteria are unclear, it must explain the gap and request clarification or propose a documented plan rather than inventing requirements.

## 3. Universal Prompt Template

Copy this template into an AI chat and replace the bracketed content.

```markdown
# AI Role

You are a [role, for example: senior full-stack engineer, backend engineer, frontend engineer, QA engineer, technical writer, or code reviewer] contributing to DevPilot AI.

Follow Documentation First Development, Story Driven Development, the Modular Monolith, Domain Driven Design, Feature Driven Development, and all repository rules. Do not make material product, architecture, security, or data-model decisions without sufficient documented context.

# Task Objective

[Describe the desired outcome, user problem, and task type. State what success looks like; do not prescribe an implementation unless it is already an approved constraint.]

# Project Context

DevPilot AI is an AI-assisted software engineering workspace, not an IDE. It connects project knowledge, story-driven work, architecture, documentation, GitHub activity, and human-guided AI assistance.

Preserve the Modular Monolith and domain ownership. For AI product features, Knowledge Service is the only provider of authorized project context. AI must not access databases, infrastructure, integrations, or business modules directly for context.

# Relevant Documents

Review these before implementation, in priority order:

1. Story / issue: [path or link]
2. Architecture and applicable ADR / technical design: [paths or links]
3. Project context: `.ai/project-context.md`
4. Coding and architecture rules: `.ai/coding-rules.md`, `.ai/architecture-rules.md`
5. Existing implementation and tests: [paths]

Also review when applicable:

- Vision: `docs/product/vision.md`
- Requirements: `docs/product/requirements.md`
- Workflow: `.ai/workflow.md`
- Related API, database, design-system, and deployment documents: [paths]

If documents conflict, follow the source-of-truth order in `.ai/project-context.md`. State any unresolved conflict before changing code.

# Story / Epic Information

- Epic: [identifier and title, or `Not applicable`]
- Story / issue: [identifier, title, and link]
- User or business outcome: [outcome]
- Current status and dependencies: [status, blockers, linked stories/documents]
- Relevant traceability links: [requirements, technical design, ADR, GitHub activity, test/deployment records]

# Functional Requirements

- [Requirement 1]
- [Requirement 2]
- [Requirement 3]

Out of scope:

- [Explicit exclusions]

# Technical Constraints

- Technology and existing patterns to use: [for example, Next.js/React, NestJS, Prisma, PostgreSQL, Redis]
- Authorization, tenancy, or privacy constraints: [constraints]
- Integration and asynchronous-work constraints: [constraints]
- Compatibility, migration, performance, accessibility, or observability constraints: [constraints]

# Architecture Constraints

- Owning module: [module and reason]
- Data owner for new or changed records: [module]
- Allowed cross-module interaction: [exported service interface, REST contract, domain event, or read projection]
- REST/API impact: [endpoint/contract change, or `None`]
- Knowledge Service / AI context impact: [impact, or `Not applicable`]
- Shared package reuse or change: [existing component/type/configuration, or `None`]

Do not access or modify another module's data directly. Do not import another module's repositories, ORM models, migrations, or internals. Do not introduce a microservice, broker, vector database, autonomous AI action, or breaking API/data change without an approved technical design or ADR.

# Files Allowed to Modify

- `[path or glob]` — [reason]
- `[path or glob]` — [reason]

# Files That Must NOT Be Modified

- `[path or glob]` — [reason]
- `[path or glob]` — [reason]

If a necessary change falls outside the allowed scope, stop and explain why it is needed before modifying it.

# Acceptance Criteria

- [Observable outcome 1]
- [Observable outcome 2]
- [Authorization, error, empty, loading, or lifecycle outcome]
- [Traceability or documentation outcome]

# Testing Requirements

- Unit tests: [required behavior or `Not applicable`, with reason]
- Integration/API tests: [required behavior or `Not applicable`, with reason]
- End-to-end/manual verification: [required workflow or `Not applicable`, with reason]
- Existing relevant test suites to run: [commands or paths]

# Expected Output

1. Briefly summarize the relevant context and state any assumptions or blockers.
2. Provide a concise implementation plan before code when the change is non-trivial.
3. Implement only the approved scope.
4. Report changed files and the purpose of each change.
5. Report validation performed, results, and any tests not run.
6. Identify documentation, follow-up work, risks, or decisions requiring human review.

# Self-Review Checklist

Before responding, verify:

- Does the change satisfy the story and acceptance criteria?
- Does it respect the owning module, data ownership, authorization, and API boundaries?
- Does it reuse existing components, services, utilities, and shared contracts where appropriate?
- Does it avoid duplicate logic, unnecessary abstractions, and unrelated rewrites?
- Are errors, loading/empty states, validation, logging, and sensitive-data handling appropriate?
- Are tests proportionate to the changed behavior and have relevant checks been run?
- Are documentation, technical design, ADR, API contract, or migration updates required?
- If AI context is involved, does it come only from Knowledge Service with permissions and source references preserved?
```

## 4. Example Prompt for Feature Development

```markdown
# AI Role

You are a senior full-stack engineer contributing to DevPilot AI.

# Task Objective

Implement the approved story for creating and viewing project documentation records. Deliver the complete authorized vertical slice: frontend experience, REST API, persistence, validation, tests, and documentation updates required by the story.

# Relevant Documents

1. Story: `[story path or issue link]`
2. Architecture: `docs/architecture/architecture.md`
3. Project context: `.ai/project-context.md`
4. Rules: `.ai/coding-rules.md`, `.ai/architecture-rules.md`
5. Existing implementation: `[documentation and project module paths]`

# Story / Epic Information

- Epic: Documentation
- Story: [story identifier and title]
- Outcome: An authorized project member can create and view a documentation record associated with a project.

# Functional Requirements

- Create a documentation record with the fields defined in the story.
- Show the record in the authorized project context.
- Return meaningful validation and access-denied errors.

Out of scope:

- Collaborative editing, external document import, and AI-generated content.

# Architecture Constraints

- Owning module: Documentation.
- Project boundary and membership authorization are obtained through published Projects/Workspace interfaces.
- Documentation owns its records and revisions; no other module writes its tables.
- The frontend uses the versioned REST API only.
- Reuse `packages/shared-ui` and `packages/shared-types` where applicable.

# Files Allowed to Modify

- `apps/web/**` only where needed for the documentation feature.
- `apps/api/**` only where needed for the Documentation module and its public contracts.
- `packages/shared-ui/**` or `packages/shared-types/**` only for genuinely reusable additions.
- Relevant tests and documentation.

# Files That Must NOT Be Modified

- Unrelated modules, migrations, and deployment configuration.

# Acceptance Criteria

- An authorized user can create and view the documentation record in the intended project.
- A user without project access cannot read or create it.
- Invalid input receives a meaningful client-safe error.
- Tests cover the relevant business and authorization behavior.

# Testing Requirements

- Add unit/integration tests for creation validation and project authorization.
- Run relevant API and web test suites, plus manual verification of the create/view flow.

# Expected Output

First summarize the story context and plan. Then implement the smallest complete vertical slice, report changed files, validation, and any documentation follow-up.
```

## 5. Example Prompt for Bug Fix

```markdown
# AI Role

You are a senior engineer diagnosing and fixing a DevPilot AI defect.

# Task Objective

Fix `[bug identifier/title]`: `[observable incorrect behavior]`. Preserve the existing product behavior outside the defect scope.

# Relevant Documents

1. Bug report / story: `[link or path]`
2. Architecture: `docs/architecture/architecture.md`
3. Rules: `.ai/project-context.md`, `.ai/coding-rules.md`, `.ai/architecture-rules.md`
4. Existing implementation, logs, and tests: `[paths]`

# Functional Requirements

- Reproduce or reason from evidence about the reported failure.
- Identify the root cause before changing code.
- Correct the behavior without bypassing authorization, validation, or module ownership.
- Add a regression test at the most appropriate level.

# Technical and Architecture Constraints

- Owning module: `[module]`.
- Do not change unrelated public contracts, data ownership, or workflow states.
- If the bug crosses module boundaries, use the existing published interface; do not add direct table access as a shortcut.

# Files Allowed to Modify

- `[affected feature/module paths]`
- `[relevant tests]`

# Files That Must NOT Be Modified

- `[unrelated paths]`

# Acceptance Criteria

- The reported scenario no longer fails.
- The regression test fails before the fix and passes after it.
- Existing authorized behavior remains unchanged.

# Testing Requirements

- Run the targeted regression test and affected test suite.
- State whether manual verification was performed and what scenario was checked.

# Expected Output

Report the evidence, root cause, minimal fix plan, changed files, and test results. If the report cannot be reproduced or requirements are ambiguous, state what evidence is missing before implementing.
```

## 6. Example Prompt for Refactoring

```markdown
# AI Role

You are a senior software engineer improving maintainability in DevPilot AI.

# Task Objective

Refactor `[scope]` to improve `[readability, duplication, cohesion, testability, or performance evidence]` without changing approved user-visible behavior.

# Relevant Documents

1. Refactoring story / technical design: `[link or path]`
2. Architecture: `docs/architecture/architecture.md`
3. Rules: `.ai/project-context.md`, `.ai/coding-rules.md`, `.ai/architecture-rules.md`
4. Existing implementation and tests: `[paths]`

# Technical and Architecture Constraints

- Owning module: `[module]`.
- Preserve public API contracts and database behavior unless the story explicitly approves a compatible migration.
- Keep behavior in the owning module; do not move domain logic into a generic utility or shared package without cross-feature justification.
- Do not use the refactor as a reason for an unrelated rewrite or new microservice.

# Files Allowed to Modify

- `[paths]`

# Files That Must NOT Be Modified

- `[paths]`

# Acceptance Criteria

- The documented maintainability problem is removed or measurably reduced.
- User-visible behavior and authorization are preserved.
- Existing tests pass and tests are improved where the prior structure obscured behavior.
- No duplicate logic, circular dependency, or cross-module data access is introduced.

# Testing Requirements

- Run the affected unit, integration, and/or end-to-end tests.
- Add characterization tests before refactoring when behavior is uncertain.

# Expected Output

Explain the current problem, proposed minimal refactor, behavior-preservation evidence, changed files, and validation. Escalate if the refactor requires a new boundary, API break, data migration, or ADR.
```

## 7. Example Prompt for Documentation

```markdown
# AI Role

You are a technical writer and software architect contributing to DevPilot AI documentation.

# Task Objective

Create or update `[document path]` so it accurately describes `[approved behavior, workflow, architecture decision, API, or contributor guidance]`.

# Relevant Documents

1. Source story / decision: `[link or path]`
2. Architecture: `docs/architecture/architecture.md`
3. Product requirements and vision: `[paths]`
4. AI contributor rules: `.ai/project-context.md`, `.ai/workflow.md`, `.ai/coding-rules.md`, `.ai/architecture-rules.md`
5. Existing related documentation and implementation: `[paths]`

# Documentation Constraints

- Treat approved product requirements and architecture as source material; do not invent product behavior or implementation decisions.
- Preserve the source-of-truth hierarchy and link to the authoritative document rather than duplicating volatile detail.
- Use clear Markdown, concise headings, tables or diagrams only when they materially improve understanding, and consistent terminology.
- Update related documentation only when it would otherwise become inaccurate.

# Files Allowed to Modify

- `[documentation paths]`

# Files That Must NOT Be Modified

- Application code, database migrations, and unrelated documentation unless explicitly approved.

# Acceptance Criteria

- The document is accurate, internally consistent, and useful to its intended contributor audience.
- It does not contradict approved Vision, Requirements, Architecture, or rules.
- Links and referenced paths are valid.

# Testing Requirements

- Run Markdown/link/format validation if the repository provides it.
- Review the final diff for correctness and unintended scope.

# Expected Output

Summarize the sources used, the documentation changes, validation performed, and any unresolved source conflict or documentation follow-up.
```

## 8. Best Practices

- Give the AI a clear outcome and acceptance criteria, not only a list of files to edit.
- Scope the prompt to the story and name both allowed and prohibited files; permit expansion only after the AI explains the need.
- Include the owning module and data owner for non-trivial changes. If unknown, make ownership determination the first planning task.
- Provide examples, screenshots, API payloads, logs, or reproduction steps for UI and defect work instead of expecting the AI to infer them.
- Request the smallest complete vertical slice. Avoid sequencing a product feature into frontend-only and backend-only deliverables.
- Require source-aware, permission-safe use of Knowledge Service for any AI feature.
- Ask the AI to distinguish verified facts, assumptions, and open questions in its response.
- Keep testing proportional to risk: business rules and authorization deserve focused coverage; a documentation-only edit may require only link/format validation.
- Require a final report of changed files, validation, known limitations, and documentation/ADR follow-up.

## 9. Common Mistakes to Avoid

- Asking for code without a story, acceptance criteria, relevant implementation paths, or source documents.
- Giving instructions that conflict with architecture rules or allowing the AI to resolve the conflict by guessing.
- Omitting data ownership, tenancy, authorization, or migration constraints for backend changes.
- Treating `shared-ui` or `shared-types` as a default location for feature-specific code.
- Allowing direct cross-module table access, frontend infrastructure access, or AI context retrieval outside Knowledge Service.
- Requesting a large refactor without defining the behavior that must remain stable.
- Requiring high test coverage without identifying the behavior and risk that tests should cover.
- Asking the AI to silently make material product, security, architecture, or deployment decisions.
- Forgetting to update requirements, architecture, ADRs, API contracts, technical designs, or contributor documentation when a change makes them inaccurate.
