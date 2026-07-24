# High-Level Design (HLD)
# DevPilot AI

**Version:** 1.0  
**Status:** Approved  

---

## 1. Introduction

### 1.1 Purpose
This High-Level Design (HLD) document outlines the system architecture for DevPilot AI. It translates the business and functional requirements defined in the Software Requirements Specification (SRS) into a foundational technical architecture. This document will serve as the primary guide for subsequent Low-Level Design (LLD), database schema design, and API specification.

### 1.2 Scope
This document covers the architectural design for the Minimum Viable Product (MVP) of DevPilot AI. It encompasses the frontend, backend, database, AI engine, and external integrations necessary to support the core "Idea to Code" lifecycle. Future scope items (e.g., multi-agent orchestration, IDE extensions) are deliberately excluded.

### 1.3 Audience
- **Software Architects**: To validate the overall system design.
- **Backend & Frontend Developers**: To understand module boundaries and interactions.
- **AI Engineers**: To comprehend the integration of language models into the core workflow.
- **DevOps Engineers**: To plan infrastructure and deployment strategies.

### 1.4 References
- **Requirement.md**: DevPilot AI Software Requirements Specification (SRS).

### 1.5 Relationship with SRS
This HLD strictly adheres to the approved SRS. It dictates *how* the system will structurally achieve the *what* defined in the SRS, without introducing new features or altering business logic.

---

## 2. Architectural Goals

- **Scalability**: Support horizontal scaling of backend services to handle concurrent organizational workspaces and AI generation workloads.
- **Maintainability**: Utilize a modular structure to ensure isolated business domains, facilitating easier onboarding and updates.
- **Security**: Enforce strict multi-tenant isolation, robust Role-Based Access Control (RBAC), and secure handling of intellectual property.
- **Performance**: Ensure responsive user interfaces and minimize perceived latency during AI generation via streaming and aggressive caching.
- **Availability**: Design a fault-tolerant architecture capable of recovering from external service degradations.
- **Extensibility**: Establish clear internal boundaries to allow future decoupling into microservices.
- **Observability**: Integrate centralized logging and metrics to ensure operational transparency.
- **AI-First Development**: Deeply embed context aggregation into the architecture, ensuring AI operations are deterministic and strictly constrained by project rules.

---

## 3. High-Level System Overview

### 3.1 Overall Architecture
DevPilot AI follows a **Modular Monolith Architecture**. The system operates as a single deployable backend application, internally partitioned into highly cohesive, loosely coupled business domains (modules). This approach maximizes development velocity and simplifies deployment for the MVP while preserving the ability to extract microservices in the future.

### 3.2 Major Components
The system is composed of a Single Page Application (SPA) frontend, a unified backend application housing the business logic and AI orchestration engine, a relational database for persistent state, a distributed cache for high-speed operations and messaging, and object storage for document attachments.

### 3.3 User Interaction
Users interact with the platform entirely through the SPA via secure APIs and persistent connections for real-time updates and AI response streaming.

### 3.4 AI Integration
The AI Engine operates as a dedicated module within the backend monolith. It aggregates structured context from the database and coordinates with external LLMs using an orchestrated flow, ensuring code generation is context-aware and compliant with project constraints.

### 3.5 System Boundaries
The internal system consists of the frontend, backend, database, cache, and object storage. External boundaries include version control systems (GitHub), OpenAI-compatible LLM providers, and SMTP providers for notifications.

---

## 4. Architecture Diagram

```text
                  +---------------------------+
                  |       Web Browser         |
                  |     (Frontend SPA)        |
                  +-------------+-------------+
                                |
                                | HTTPS / WSS
                                v
                  +-------------+-------------+
                  |    Reverse Proxy Layer    |
                  +-------------+-------------+
                                |
                                v
+-----------------------------------------------------------------+
|                       DevPilot Backend (Modular Monolith)       |
|                                                                 |
|  +--------------+  +--------------+  +--------------+           |
|  |              |  |              |  |              |           |
|  | Auth Module  |  | Core Modules |  | Support Mods |           |
|  |              |  | (Workspace,  |  | (Dashboards, |           |
|  +--------------+  |  Project,    |  |  Audit,      |           |
|                    |  Story, Docs)|  |  Billing)    |           |
|  +--------------+  |              |  +--------------+           |
|  |  AI Engine   |  +--------------+                             |
|  |              |                                               |
|  +--------------+                                               |
+-------+-------------------+-------------------+-----------------+
        |                   |                   |
        v                   v                   v
+-------+-------+   +-------+-------+   +-------+-------+
|  Relational   |   |  Distributed  |   |    Object     |
|   Database    |   |     Cache     |   |    Storage    |
+-------+-------+   +---------------+   +---------------+
        |                                       |
        +-------------------+-------------------+
                            |
           +----------------v----------------+
           |       External Services         |
           |  - Version Control (GitHub)     |
           |  - LLM Providers                |
           |  - SMTP Provider                |
           +---------------------------------+
```

---

## 5. Major Components

### 5.1 Frontend (SPA)
- **Purpose**: Provides the interactive user interface for all organizational operations.
- **Responsibilities**: Rendering views, managing local state, routing, and communicating with backend APIs.
- **Interactions**: Calls backend APIs and listens to real-time events.
- **Dependencies**: Backend APIs.

### 5.2 Backend (Modular Monolith)
- **Purpose**: Houses the core business logic, API boundaries, and orchestration workflows.
- **Responsibilities**: Enforcing RBAC, managing data persistence, and orchestrating workflows between modules.
- **Interactions**: Serves the frontend, reads/writes to DB and Cache, integrates with external APIs.
- **Dependencies**: Relational Database, Distributed Cache, External APIs.

### 5.3 AI Engine
- **Purpose**: Acts as the intelligent context aggregator and code generator.
- **Responsibilities**: Building contexts, managing LLM interactions, validating generated code, and streaming responses.
- **Interactions**: Fetches context from Core Modules, calls external LLM APIs.
- **Dependencies**: LLM APIs, Core Modules.

### 5.4 Database
- **Purpose**: Persistent storage for all relational platform data.
- **Responsibilities**: Maintaining ACID compliance for users, workspaces, projects, documents, and stories.
- **Interactions**: Accessed exclusively by the Backend via the persistence layer.

### 5.5 Cache & Real-time Messaging
- **Purpose**: Accelerate read operations and handle real-time messaging.
- **Responsibilities**: Storing ephemeral sessions, rate-limiting counters, and broadcasting events.
- **Interactions**: Accessed by the Backend.

### 5.6 Storage
- **Purpose**: Store unstructured binary data (e.g., attachments, avatars).
- **Responsibilities**: Securely storing and serving files.
- **Interactions**: Accessed by the Backend via secure URLs or direct client upload.

### 5.7 Version Control Integration
- **Purpose**: Synchronize the platform with the external source of truth for code.
- **Responsibilities**: Managing event ingestion, automated branch creation, and pull request automation.

---

## 6. Frontend Architecture

### 6.1 Application Layers
The frontend separates concerns into three primary layers:
- **Presentation Layer**: UI components responsible purely for rendering data.
- **State Layer**: Manages global client-side state and caching of remote data.
- **Service Layer**: Encapsulates API client calls and real-time communication.

### 6.2 Routing
Client-side routing manages navigation between core views. Routes are guarded, requiring valid authorization to render protected layouts.

### 6.3 State Management
Remote data state (server state) is managed via a dedicated data-fetching abstraction that handles caching, invalidation, and optimistic updates. Global UI state is managed via a lightweight centralized store.

### 6.4 Authentication Flow
Upon successful login, an authentication token is securely maintained within the application state, while long-lived session tokens are handled via secure transport and storage mechanisms. Request middleware automatically injects authorization claims into outgoing requests and attempts silent session renewals upon expiry.

### 6.5 UI Module Organization
The application is organized by feature domain (e.g., Auth, Projects, Stories). Each feature encapsulates its own components, logic, and state.

### 6.6 Shared Components
A robust, accessible design system is implemented using a utility-first approach and a headless component abstraction, ensuring consistency across all screens without tying business logic to UI rendering.

---

## 7. Backend Architecture

### 7.1 Framework & Module Organization
The backend utilizes a heavily structured modular design. Each business domain is encapsulated in its own module, exposing strict interfaces to other modules to prevent tight coupling.

### 7.2 Core Modules
- **Auth Module**: Handles login, registration, SSO, and session lifecycle.
- **Workspace Module**: Manages tenant isolation and member invitations.
- **Project Module**: Manages project configurations and repository links.
- **Document Module**: Handles the creation, versioning, and linking of project context documents.
- **Story Module**: Manages the agile lifecycle, transitions, and acceptance criteria.

### 7.3 Supporting Modules
- **Notification Module**: Dispatches internal events and external alerts.
- **Audit Module**: Captures and stores immutable historical actions.
- **Search Module**: Indexes and queries global artifacts.
- **Billing Module**: Tracks organizational usage against subscription limits.

### 7.4 Layered Architecture
Each backend module is internally segregated into strict architectural layers:
- **Presentation Layer**: Handles incoming API requests, enforces input validation, and formats API responses.
- **Application Layer**: Coordinates use cases, delegates business rules to the domain, and manages transactions.
- **Domain Layer**: Contains pure business logic, entities, and domain rules agnostic of external frameworks or infrastructure.
- **Infrastructure Layer**: Manages external I/O, including database interactions, cache communication, and third-party API clients.

---

## 8. Module Interaction

The modules communicate through strictly defined internal boundaries.

```text
+-------------------+
|   Presentation    |
+---------+---------+
          |
+---------v---------+       +------------------+
|   Auth Module     +------>+   Audit Module   |
+---------+---------+       +------------------+
          |                          ^
+---------v---------+                |
| Workspace Module  +----------------+
+---------+---------+                |
          |                          |
+---------v---------+       +--------+---------+
|  Project Module   +------>+ Search Module    |
+---------+---------+       +--------+---------+
          |                          |
+---------v---------+                |
|  Document Module  +----------------+
+---------+---------+                |
          |                          |
+---------v---------+       +--------+---------+
|   Story Module    +------>+ Notifications    |
+---------+---------+       +------------------+
          |
+---------v---------+       +------------------+
|    AI Engine      +------>+  VCS Integration |
+-------------------+       +------------------+
```

---

## 9. AI Architecture

### 9.1 Context Sources
The AI Engine relies on deterministic data ingestion. Before generation, it pulls from:
- **Project Context**: Core constraints, tech stack.
- **Document Context**: Approved architectural documents and coding standards.
- **Story Context**: Specific acceptance criteria and associated technical designs.
- **Repository Context**: Analyzed metadata regarding existing codebase structures.

### 9.2 Architectural Flow
The AI Engine leverages a state machine to orchestrate complex reasoning steps:
1. **Context Aggregation Phase**: Gathers all constraints.
2. **Planning Phase**: Drafts an implementation strategy.
3. **Validation Phase**: Checks the plan against established constraints.
4. **Generation Phase**: Formulates and streams actual code output.
5. **Review Phase**: Self-evaluates the output before finalization.

### 9.3 Memory Strategy
The AI Engine persists interactions logically grouped by sessions. When a new request is received for an existing workflow, previous conversational state is loaded to maintain contextual continuity.

### 9.4 LLM Integration
The engine communicates strictly with standard, compatible API interfaces, allowing the platform to route requests to different underlying models without altering core business logic.

---

## 10. Security Architecture

### 10.1 Authentication & Authorization
Authentication leverages standard asymmetric cryptographic tokens. The system guarantees multi-tenant security by strictly evaluating tenant claims on every request, ensuring users cannot access cross-tenant data.

### 10.2 Data Protection
- **In Transit**: All external communication enforces strong encryption standards (e.g., TLS).
- **At Rest**: Database volumes and object storage buckets are encrypted using industry-standard algorithms.

### 10.3 Secrets Management
Sensitive credentials are strictly managed via environment configurations and never committed to source control. Tenant-provided integration keys are symmetrically encrypted before persistence.

### 10.4 Validation & Rate Limiting
All incoming data boundaries utilize strict structural validation. A distributed cache is utilized to implement rate limiting on public endpoints to prevent abuse.

---

## 11. Cross-Cutting Concerns

The architecture abstracts the following shared responsibilities across all modules to ensure consistency:

- **Authentication**: Validating user identity uniformly across all system entry points.
- **Authorization**: Enforcing RBAC policies dynamically before granting access to specific domain resources.
- **Validation**: Enforcing strict schema and data integrity constraints at the system boundaries.
- **Configuration**: Abstracting environment-specific configurations entirely away from application code.
- **Logging**: Implementing standardized, structured event tracking across all layers.
- **Exception Handling**: Global interception of errors to ensure graceful degradation and uniform API responses.
- **Audit Logging**: Asynchronously recording security and state-mutating events without blocking primary flows.
- **Caching**: Optimizing read-heavy operations through distributed caching strategies.
- **Monitoring**: Emitting standardized health and performance metrics for external observability.

---

## 12. High-Level Request Lifecycles

### Flow 1: User Login
Browser
↓
Frontend
↓
Backend
↓
Authentication Module
↓
Database
↓
Response

### Flow 2: AI Code Generation
Browser
↓
Frontend
↓
Backend
↓
AI Engine
↓
Project Context
↓
LLM Provider
↓
Validation
↓
GitHub Integration
↓
Frontend

---

## 13. Data Flow

### 13.1 Core Idea-to-Deployment Flow
1. **Idea & Planning**: A user creates an Epic and underlying Stories.
2. **Contextualization**: A user authors technical documentation and links it to the Story.
3. **AI Execution**: The user assigns the Story to the AI Engine.
4. **Orchestration**: The Backend triggers the AI Engine, which pulls the required constraints.
5. **Generation**: The AI Engine processes the request and streams the response back to the client.
6. **Integration**: Upon approval, the system uses the version control integration to create a branch, commit code, and open a pull request.
7. **Deployment**: Once manually reviewed and merged, the external pipeline deploys the code. The system receives a status update and reflects this in the dashboard.

---

## 14. External Integrations

### 14.1 Version Control (GitHub)
- **Purpose**: Read codebase state and automate branch/PR creation.
- **Data Exchanged**: Source code diffs, branch metadata, event payloads.
- **Failure Handling**: Implement backoff strategies for API limits; enqueue missed events for background processing.

### 14.2 LLM Providers
- **Purpose**: Execute natural language processing and code generation.
- **Data Exchanged**: System constraints, contextual codebase snippets, generated code responses.
- **Failure Handling**: Circuit breakers to prevent cascading failures; graceful degradation with user-facing error messaging.

### 14.3 SMTP Provider
- **Purpose**: Dispatch transactional communications.
- **Data Exchanged**: Delivery addresses, templated content.
- **Failure Handling**: Asynchronous processing via queues; retry mechanisms for temporary network failures.

### 14.4 Storage Provider
- **Purpose**: Store unstructured binary data.
- **Data Exchanged**: Binary files.
- **Failure Handling**: Synchronous error reporting to the user if uploads fail.

---

## 15. Deployment Overview

### 15.1 Containerization Layer
The entire platform is packaged into generic, isolated containers to ensure consistency across development, staging, and production environments.
- **Web Server Layer**: Serves static compiled client assets.
- **Application Backend Layer**: Runs the core server-side application.
- **Database Layer**: Hosts the relational data persistence.
- **Cache Layer**: Hosts the distributed caching and messaging infrastructure.

### 15.2 Reverse Proxy Layer
A reverse proxy sits at the edge of the deployment, terminating secure connections and appropriately routing traffic to the frontend or backend layers.

### 15.3 Environment Configuration
All environment-specific variables are injected into the layers at runtime, completely separating code from configuration.

---

## 16. Logging & Monitoring

### 16.1 Application Logs
The system utilizes a structured logging strategy, outputting logs in a standard format for easy ingestion by log aggregators. Log levels are strictly enforced based on environment configurations.

### 16.2 Error Logging & Exception Handling
A global exception abstraction intercepts all unhandled errors, ensuring the user receives a standardized response while logging the complete trace context internally.

### 16.3 Metrics & Health Checks
The system exposes dedicated endpoints for orchestration platforms to verify liveness and readiness. Basic application metrics are collected for performance monitoring.

---

## 17. Technology Stack

- **Frontend Framework: React with Vite and TypeScript**
  *Justification*: React provides massive ecosystem and component reusability. Vite ensures rapid build times. TypeScript guarantees type safety, reducing runtime errors.
- **Frontend UI: Tailwind CSS & shadcn/ui**
  *Justification*: Tailwind allows rapid, consistent styling without leaving the component. shadcn/ui provides highly accessible, customizable unstyled components, accelerating UI development.
- **Backend Framework: NestJS (TypeScript)**
  *Justification*: NestJS enforces a highly opinionated, modular architecture out of the box, which is critical for maintaining a clean Modular Monolith.
- **Database: PostgreSQL**
  *Justification*: A robust, ACID-compliant relational database ideal for complex hierarchical data (Workspaces -> Projects -> Stories).
- **ORM: Prisma**
  *Justification*: Prisma provides excellent type-safety and a highly readable schema definition, accelerating database interactions and migrations.
- **Cache & Pub/Sub: Redis**
  *Justification*: In-memory speed is necessary for rate limiting, session caching, and event broadcasting across instances.
- **AI Orchestration: LangGraph**
  *Justification*: Provides a stateful, graph-based framework for constructing complex, multi-step LLM workflows (planning, generation, validation) required for context-aware code generation.

---

## 18. Architectural Decisions (ADR Summary)

### 18.1 ADR: Modular Monolith vs. Microservices
**Decision**: Adopt a Modular Monolith.
**Reasoning**: Microservices introduce massive operational overhead. A Modular Monolith allows the startup to move rapidly, deploy a single artifact, and easily trace bugs, while strict module boundaries preserve the ability to split the application later if scaling demands it.

### 18.2 ADR: Graph-Based AI Orchestration vs. Linear Prompts
**Decision**: Utilize LangGraph for AI workflows.
**Reasoning**: Linear zero-shot prompts are insufficient for complex engineering tasks. A graph-based approach allows the AI engine to loop, self-correct, validate against rules, and maintain state before returning a final output.

### 18.3 ADR: Relational Database over NoSQL
**Decision**: Use PostgreSQL.
**Reasoning**: The platform relies heavily on structured relationships (Users belong to Workspaces, Projects have Stories, Stories have Tasks). Relational integrity and complex join capabilities are mandatory.

---

## 19. Risks

### 19.1 Technical Risks
- **Risk**: Connection instability during long-running AI generations.
  **Mitigation**: Implement robust client-side reconnection logic and server-side state recovery to resume streams.

### 19.2 AI Risks
- **Risk**: LLM context window limits being exceeded by massive repositories.
  **Mitigation**: Implement strict context chunking, summarization, and retrieval-augmented generation techniques to prioritize only highly relevant codebase context.

### 19.3 Operational Risks
- **Risk**: Hitting external API rate limits during repository synchronization.
  **Mitigation**: Rely primarily on incoming events rather than aggressive polling; utilize authenticated integrations to maximize rate limits.

---

## 20. Assumptions
- The target deployment environment supports standard containerization strategies.
- Required external APIs will maintain their current interface structures and authentication protocols.
- The platform will not handle real-time synchronous collaborative editing in the MVP, but rather standard asynchronous saves with locking or basic conflict resolution.

---

## 21. Future Evolution
While designed as a Modular Monolith, the system enforces strict boundaries (e.g., the Story Module cannot directly mutate the Auth Module's state; it must communicate via an exposed abstraction). As the product scales and teams grow, specific high-load domains can be detached into standalone microservices communicating via RPC or message queues with minimal refactoring of the core business logic.
