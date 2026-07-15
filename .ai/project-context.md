# DevPilot AI — Project Context

> This document provides high-level project context for AI assistants contributing to DevPilot AI.
>
> It is intended to help AI understand the project quickly without reading every document. Detailed architecture, requirements, and implementation details are maintained in the `/docs` directory.

## Project Overview

DevPilot AI is an AI-powered Software Engineering Management Platform.

It is designed to help software teams manage the complete software development lifecycle from idea to deployment while keeping documentation, architecture, stories, GitHub activity, and engineering knowledge connected.

DevPilot AI is **not** an IDE. Developers continue writing code in their preferred editor, such as VS Code, Cursor, or IntelliJ. DevPilot AI provides project management, engineering context, and AI assistance around the development process.

## Primary Goal

Build software with:

- Better planning.
- Better documentation.
- Better collaboration.
- Better engineering decisions.
- AI that understands the project instead of isolated prompts.

## Engineering Philosophy

The project follows Documentation First Development. Every feature starts as an idea and gradually evolves through documented stages before implementation.

```text
Vision
  ↓
Requirements
  ↓
Architecture
  ↓
Epic
  ↓
Story
  ↓
Technical Design
  ↓
Implementation
  ↓
Testing
  ↓
Review
  ↓
Deployment
```

## Development Philosophy

The project follows Story Driven Development. Stories are the smallest deployable unit of work.

Every story should contain:

- Frontend.
- Backend.
- API.
- Database changes.
- Documentation updates.
- Tests.

A story is complete only when all parts are delivered.

## Architecture

The system follows a Modular Monolith architecture. Business domains remain isolated, and modules should be capable of becoming independent microservices in the future without major redesign.

The detailed architecture is documented in [`docs/architecture/`](../docs/architecture/).

## Core Domains

- Authentication
- Workspace
- Projects
- Documentation
- Stories
- GitHub
- Knowledge
- AI
- Collaboration
- Notifications
- Search
- Settings

## Knowledge Service

Knowledge Service is one of the most important architectural concepts. It collects project knowledge from multiple sources, including:

- Documentation
- Stories
- Architecture
- Requirements
- GitHub metadata
- A future vector database

AI should rely on Knowledge Service for project context instead of directly querying business modules.

## Technology Stack

| Area | Technologies |
| --- | --- |
| Frontend | Next.js, React, TypeScript, Tailwind, Shadcn UI |
| Backend | NestJS, PostgreSQL, Prisma, Redis |
| AI | Gemini, LangGraph, MCP |
| Deployment | Vercel, Railway, Neon |

## AI Responsibilities

AI assists developers; it does not replace engineering judgment.

AI should:

- Explain.
- Review.
- Suggest.
- Automate repetitive work.

AI should not make architectural decisions without sufficient context.

## Project Priorities

Always prioritize:

1. Maintainability
2. Readability
3. Documentation
4. Simplicity
5. Modularity
6. Scalability

Performance optimization should happen only when justified.

## Before Making Changes

Before implementing a feature, understand:

- Vision
- Requirements
- Architecture
- Story
- Existing implementation

Never generate code without understanding the context.

## Source of Truth

If multiple documents disagree, use the following priority:

1. Architecture
2. Requirements
3. Vision
4. Stories
5. Code
