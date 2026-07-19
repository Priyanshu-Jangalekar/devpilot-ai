# Low-Level Design (LLD)
# DevPilot AI

**Version:** 1.0  
**Status:** Approved  

---

## 1. Introduction

### 1.1 Purpose
This Low-Level Design (LLD) document provides the implementation blueprint for DevPilot AI. It bridges the gap between the architectural vision established in the High-Level Design (HLD) and the concrete code implementation. It instructs software engineers on exactly how to organize, structure, and build internal modules.

### 1.2 Scope
This document covers the internal design of the Minimum Viable Product (MVP) modules for the frontend and backend of DevPilot AI. It details class/component responsibilities, dependency structures, internal data flows, and design patterns.

### 1.3 Audience
- **Backend Developers**: To implement backend modules, application services, and infrastructure repositories.
- **Frontend Developers**: To implement React features, state management, and UI components.
- **AI Engineers**: To implement the AI orchestration pipelines and context aggregation logic.

### 1.4 Relationship with HLD
The HLD defines *what* components exist and how they connect at a system level. This LLD defines *how* those components are built internally.

### 1.5 Relationship with SRS
The SRS defines business rules and functional requirements. This LLD provides the structural mechanisms through which those rules will be enforced in code.

---

## 2. Design Principles

- **Modularity**: The codebase is strictly partitioned by business domain. Cross-domain data access is strictly forbidden without utilizing exposed interfaces.
- **Single Responsibility (SRP)**: Every architectural component and function must have only one reason to change.
- **Loose Coupling**: Modules interact via interfaces and Dependency Injection (DI), never by direct instantiation of concrete classes.
- **High Cohesion**: Code that changes together stays together within the same domain boundary.
- **SOLID**: All logic adheres to SOLID principles to maximize maintainability.
- **DRY (Don't Repeat Yourself)**: Shared logic is abstracted into the `Shared Layer` or generic utility services.
- **KISS (Keep It Simple, Stupid)**: Avoid over-engineering; implement the simplest logic that satisfies the requirement.
- **YAGNI (You Aren't Gonna Need It)**: Do not implement abstractions for future features not defined in the MVP SRS.
- **Dependency Injection (DI)**: Inversion of Control is heavily utilized on the backend and frontend to inject dependencies.
- **Clean Architecture Principles**: Infrastructure concerns (HTTP, Database) are decoupled from pure business domain logic.

---

## 3. Overall Layered Design

Each backend module follows a strict layered architecture:

- **Presentation Layer**: Responsible purely for handling incoming HTTP/WebSocket requests, validating payloads, and returning formatted responses. No business logic resides here.
- **Application Layer**: Consists of Application Services/Use Cases. Orchestrates the flow of data between the Presentation layer and Domain layer.
- **Domain Layer**: Consists of Domain Services and Entities. Encapsulates pure business rules and state mutations. Completely agnostic of the framework or database.
- **Infrastructure Layer**: Consists of Repositories and third-party adapters (e.g., Version Control Client, LLM Client). Responsible for all external I/O and data persistence.
- **Shared Layer**: Consists of cross-cutting utilities, generic exception handlers, logging middlewares, and core configurations used universally across all layers.

---

## 4. Shared Components

- **Authentication**: A unified Security Layer validates identity tokens for all protected routes, attaching the user identity to the request context.
- **Authorization**: A unified RBAC Layer reads route metadata and evaluates permissions against the request context.
- **Configuration**: A centralized Configuration Layer loads, validates, and exposes strongly typed environment variables.
- **Logging**: A global structured logger intercepts all requests and formats outputs consistently.
- **Validation**: Global Validation Middleware automatically evaluates incoming request bodies against schema definitions.
- **Caching**: A unified Cache Service exposes generic methods used by specific modules to interface with the distributed cache.
- **Exception Handling**: A Global Exception Handler catches unhandled errors, mapping them to standardized HTTP responses.
- **Audit Logging**: An asynchronous event listener captures state mutations and persists them without blocking the HTTP response.
- **Utilities**: Shared string manipulation, date formatting, and cryptographic helpers.
- **File Storage**: A generic Storage Service implementing an object storage adapter, utilized by the Document and Profile modules.
- **Notification**: An internal Event Emitter dispatches domain events, which the Notification module consumes to trigger emails or in-app alerts.

---

## 5. Module Design

### 5.1 Authentication & User Management
- **Purpose**: Manage user identity, sessions, and registration.
- **Internal Components**:
  - `Presentation Layer (Auth)`: Handles login, registration, and refresh requests.
  - `Application Layer (Auth)`: Orchestrates credential verification and token generation.
  - `Domain Layer (User)`: Manages user profile mutations.
  - `Infrastructure Layer (User)`: Interfaces with the database for user data.
- **Data Transfer Strategy**: Strictly typed payloads for Registration, Login, and Profile Updates.
- **Business Rules**: Passwords protected via strong cryptographic hashing.
- **Dependencies**: Configuration Layer, Token Management Service, Notification Module.

### 5.2 Workspace & Project Management
- **Purpose**: Manage tenant boundaries and software projects.
- **Internal Components**:
  - `Presentation Layer (Workspace/Project)`
  - `Application Layer (Workspace/Project)`
  - `Infrastructure Layer (Workspace/Project)`
- **Validation**: Enforce unique workspace names per user.
- **Interaction**: Exports authorization logic to other modules to verify user membership before action execution.

### 5.3 Documentation Management
- **Purpose**: Manage the lifecycle and versioning of project documents.
- **Internal Components**:
  - `Presentation Layer (Document)`: Handles CRUD for docs.
  - `Application Layer (Document)`: Handles version bumping and approval workflows.
  - `Infrastructure Layer (Document)`: Persists markdown content and metadata.
- **Caching**: Frequently accessed documents (like coding standards) are cached in the distributed cache.

### 5.4 Story & Sprint Management
- **Purpose**: Manage Agile work items (Epics, Stories, Tasks).
- **Internal Components**:
  - `Presentation Layer (Story)`: Handles state transitions and assignment.
  - `Application Layer (Story)`: Enforces workflow rules (e.g., cannot start without AC).
  - `Infrastructure Layer (Story)`: Handles hierarchical data retrieval.
- **Events**: Emits domain events when statuses change.

### 5.5 AI Engineering Assistant
- **Purpose**: Aggregate context and orchestrate LLM code generation.
- **Internal Components**:
  - `Presentation Layer (AI)`: Handles persistent connections and generation requests.
  - `Context Service`: Fetches data from Document and Story modules.
  - `Prompt Service`: Assembles validated system prompts.
  - `AI Orchestration Layer`: Manages the state machine (Plan, Generate, Validate).
  - `LLM Provider Layer`: Wraps the external LLM integration.
- **Dependencies**: Document Module, Story Module, Project Module.
- **Error Handling**: Implements circuit breakers and retry logic for LLM timeouts.

### 5.6 GitHub Integration
- **Purpose**: Manage VCS synchronization and webhook handling.
- **Internal Components**:
  - `Presentation Layer (Webhook)`: Ingests version control payloads.
  - `Application Layer (GitHub)`: Orchestrates branch creation and PR generation.
  - `Infrastructure Layer (GitHub)`: Low-level wrapper around the VCS API.
- **Dependencies**: Project Module (to map repositories to projects).

### 5.7 Notification Module
- **Purpose**: Dispatch alerts to users.
- **Internal Components**:
  - `Notification Listener`: Subscribes to internal Event Emitter events.
  - `Application Layer (Notification)`: Manages delivery preferences.
  - `Infrastructure Layer (Email)`: Interfaces with the SMTP provider.
- **Caching**: Unread notification counts cached per user.

### 5.8 Search Module
- **Purpose**: Provide global entity search.
- **Internal Components**:
  - `Presentation Layer (Search)`: Exposes unified query endpoint.
  - `Application Layer (Search)`: Fans out queries to respective domain repositories and ranks results.

### 5.9 Dashboard Module
- **Purpose**: Aggregate analytics for frontend display.
- **Internal Components**:
  - `Presentation Layer (Dashboard)`: Exposes metric endpoints.
  - `Application Layer (Dashboard)`: Calculates velocity, health, and aggregates status counts.
- **Caching**: Dashboard metrics are heavily cached with appropriate TTLs to reduce database load.

---

## 6. Frontend Design

### 6.1 Application Structure
The React application follows a feature-driven folder structure, ensuring components, hooks, and services related to a specific domain are co-located.

### 6.2 Routing Strategy
Utilizes client-side routing with lazy loading for feature modules.
- **Protected Routes**: Wrapped in an authorization component that verifies token presence and validity before rendering children.

### 6.3 Layout Strategy
- **AuthLayout**: Minimal layout for login/registration.
- **AppLayout**: Includes persistent Sidebar, Top Navigation, and global context providers.

### 6.4 Shared Components
UI components (Buttons, Modals, Inputs) are built using generic foundational components and utility classes, housed in `src/shared/ui`. They contain zero domain logic.

### 6.5 State Management
- **Server State**: Managed by a data-fetching library for querying, caching, and optimistic updates.
- **Client State**: Managed by a lightweight global store for UI toggles (e.g., theme, sidebar state).

### 6.6 Forms & Validation
Forms are managed using schema-based validation. Schemas are structurally shared with backend data transfer definitions where possible.

### 6.7 API Communication
A centralized HTTP client handles all requests, utilizing request middleware to attach authorization claims and response middleware to handle unauthorized responses (triggering token refresh or redirect to login).

### 6.8 Error & Loading Strategy
- **Loading**: Suspense boundaries for route transitions; skeleton loaders for data fetching.
- **Error**: Error Boundaries catch rendering errors. API errors trigger toast notifications via a centralized error handling utility.

---

## 7. Backend Design

### 7.1 Module Organization
Every domain is a discrete architectural module. Modules must explicitly export components intended for use by other internal boundaries.

### 7.2 Request & Business Logic Layers
- **Request Handling Layer**: Manages routing, input extraction, and delegation.
- **Business Logic Layer**: Contains synchronous and asynchronous domain logic.

### 7.3 Persistence Layer
Repositories wrap database client calls. They isolate the application from specific SQL syntax, allowing easier mocking during unit tests.

### 7.4 Security, Transformation, and Validation
- **Security Layer**: Authentication and Role evaluation logic applied globally or per-route.
- **Transformation Layer**: Intercepts and standardizes outgoing response payloads.
- **Validation Layer**: Enabled globally to parse payloads and strip unknown properties.

### 7.5 Global Error Handling
A global error handler catches unhandled errors, logs the stack trace, and returns a standardized JSON error format to the frontend.

---

## 8. AI Module Internal Design

### 8.1 Pipeline Stages
1. **Context Collection**: Service fetches Story ACs, linked Documents, and Workspace rules.
2. **Prompt Construction**: Service injects collected context into a strictly formatted system prompt template.
3. **Generation Pipeline**: Orchestrates the implementation plan and code generation logic.
4. **Validation Pipeline**: Output is checked for syntax and rule violations.
5. **Review Pipeline**: Self-reflection loop evaluates the generated code against the original ACs.

### 8.2 Conversation Memory
User interactions during AI execution are saved to a session log. The Presentation Layer fetches the recent message history to append to the LLM context, maintaining conversational continuity.

### 8.3 Retry & Fallback Strategy
The LLM Provider Layer implements an exponential backoff retry strategy for transient errors and rate limits from the external provider.

### 8.4 Safety & Token Usage
Input prompts are evaluated for size before dispatch. If context exceeds the provider's context window, the system falls back to a summarization service to condense historical documents before generation.

---

## 9. Security Design

### 9.1 Authentication Flow
1. Client submits credentials.
2. Server validates the cryptographic hash.
3. Server generates a short-lived Access Token and a long-lived Refresh Token.
4. Refresh Token is managed via a secure Session Management Strategy (protected from client-side access).
5. Access Token is securely maintained in client-side state memory.

### 9.2 Authorization (RBAC)
Authorization is enforced via the Security Layer. It inspects the token payload for the user's identity, evaluates the user's role within the requested scope, and compares it against the required permissions defined on the endpoint.

### 9.3 Input & Output Validation
All inputs are validated using structural schemas. Outputs are serialized, explicitly excluding sensitive fields (e.g., passwords, hidden tokens) before leaving the server boundary.

### 9.4 Secrets Management
All secrets are loaded via the Configuration Module and validated centrally on application startup to ensure no missing credentials.

### 9.5 Rate Limiting
A distributed cache-backed rate limiting strategy restricts requests per IP/User to prevent brute force and DDoS attacks.

---

## 10. Folder Structure

### 10.1 Frontend (`src/`)
```text
src/
├── app/                  # App setup, global providers, router
├── core/                 # API clients, auth utilities
├── features/             # Feature modules (Domain logic)
│   ├── auth/
│   ├── projects/
│   ├── stories/
│   └── ai-workspace/
├── shared/               # Shared components, hooks, utils
│   ├── ui/               # Generic UI components
│   ├── hooks/
│   └── utils/
└── assets/               # Static assets
```

### 10.2 Backend (`src/`)
```text
src/
├── main.ts               # Application entry point
├── app.module.ts         # Root module
├── common/               # Shared layer across all modules
│   ├── decorators/
│   ├── filters/
│   ├── guards/
│   ├── interceptors/
│   └── utils/
├── config/               # Environment configuration schemas
├── modules/              # Domain modules
│   ├── auth/
│   ├── workspace/
│   ├── project/
│   ├── story/
│   ├── document/
│   ├── ai/
│   └── github/
└── prisma/               # Database ORM schema and migrations
```

---

## 11. Module Interaction

Modules interact strictly via injected Application Services or via the Event Emitter for decoupled asynchronous actions.

```text
+----------------+        (Injects)       +----------------+
|  StoryModule   | ---------------------> | DocumentModule |
| (Application)  |                        | (Application)  |
+-------+--------+                        +----------------+
        |
        | (Emits event)
        v
+----------------+
|  EventEmitter  |
+-------+--------+
        |
        | (Listens)
        v
+----------------+
|   Notification |
|     Module     |
+----------------+
```

---

## 12. Sequence Flows

### 12.1 Login Flow
```text
Client       Presentation Layer     Application Layer     Infrastructure Layer     DB
  |                  |                      |                      |               |
  |--- Login Request |                      |                      |               |
  |                  |--- validate() ------>|                      |               |
  |                  |                      |--- findUser() ------>|               |
  |                  |                      |                      |-- query ----->|
  |                  |                      |                      |<-- user ------|
  |                  |                      |<-- user obj ---------|               |
  |                  |                      |--- verifyHash()      |               |
  |                  |                      |--- genTokens()       |               |
  |                  |<-- tokens -----------|                      |               |
  |<-- Response -----|                      |                      |               |
```

### 12.2 AI Code Generation Flow
```text
Client     Presentation Layer    AI Orchestration Layer     Context Service     LLM Provider Layer
  |               |                        |                       |                    |
  |-- Connect --->|                        |                       |                    |
  |-- prompt ---->|                        |                       |                    |
  |               |--- execute() --------->|                       |                    |
  |               |                        |-- fetchContext() ---->|                    |
  |               |                        |<-- context rules -----|                    |
  |               |                        |-- generate() ----------------------------->|
  |               |                        |<-- stream chunk ---------------------------|
  |<-- chunk -----|                        |                       |                    |
  |               |                        |<-- stream end -----------------------------|
  |<-- done ------|                        |                       |                    |
```

---

## 13. Error Handling Strategy

- **Business Errors**: Handled by throwing appropriate logical exceptions (e.g., "Story cannot start without Acceptance Criteria").
- **Validation Errors**: Handled automatically by the Validation Layer, returning a structured list of specific field violations.
- **Authentication Errors**: Handled by the Security Layer throwing Unauthorized exceptions.
- **Authorization Errors**: Handled by the Security Layer throwing Forbidden exceptions.
- **AI Errors**: Handled by the LLM Provider Layer, catching timeouts and returning a Service Unavailable status or streaming an error frame.
- **Infrastructure Errors**: Database exceptions (e.g., unique constraint violations) are intercepted by a custom persistence error handler and mapped to standard HTTP responses.

---

## 14. Logging Strategy

- **Application Logs**: Standard request logging capturing method, URL, status code, and duration.
- **Audit Logs**: Captured asynchronously by the audit interceptor. Contains identity, action, resource, timestamp, and payload changes.
- **Security Logs**: Failed login attempts and unauthorized access attempts are explicitly logged at the WARN level for intrusion detection.
- **AI Logs**: LLM interactions log prompt metadata, token usage, and latency, but intentionally exclude sensitive source code payloads from plain-text logs.

---

## 15. Design Patterns

- **Repository Pattern**: Used in the Infrastructure Layer to abstract database access. *Why*: Decouples business logic from persistence technology, simplifying testing.
- **Dependency Injection (DI)**: Utilized natively across layers. *Why*: Promotes loose coupling and lifecycle management of services.
- **Observer Pattern**: Implemented via the event emitter. *Why*: Allows modules (like Story) to trigger side-effects (like Notifications) without tight coupling.
- **Strategy Pattern**: Used in the Auth module for authentication variants. *Why*: Allows seamless swapping or addition of authentication methods.
- **Adapter Pattern**: Used for external integrations. *Why*: Standardizes the internal interface regardless of external vendor API changes.

---

## 16. Coding Standards

- **Backend**: Strict typings enabled. Interfaces are used for data shapes, Classes for implementable logic. Return types must be explicitly declared on all endpoints.
- **Frontend**: Functional components only. Hooks used for all logic. No inline CSS; strictly utility classes.
- **Naming Conventions**: 
  - `camelCase` for variables and functions.
  - `PascalCase` for classes, interfaces, and React components.
  - `kebab-case` for file names and URLs.
- **Module Conventions**: Each domain folder must contain dedicated presentation, application, and infrastructure files.
- **Error Conventions**: Controllers must never return unhandled raw errors. Always throw standard structured exceptions.

---

## 17. Assumptions
- Developers are proficient in the chosen ecosystem conventions.
- The underlying infrastructure provides a stable distributed cache for real-time messaging distribution.
- The ORM schema generation matches the structural requirements outlined by the repositories defined herein.

---

## 18. Risks
- **Technical Risk**: Persistent connection state management across multiple backend pods during deployments. *Mitigation*: Utilize cache-backed adapters for event broadcasting across instances.
- **AI Risk**: Orchestration failing mid-execution, leaving the client hanging. *Mitigation*: Implement strict timeouts on all orchestration phases and emit explicit failure events to the client.
- **Operational Risk**: High latency during complex database joins for Project Context aggregation. *Mitigation*: Enforce database indexing on foreign keys and utilize caching for static project rules.

---

## 19. Future Extensibility
The modular boundary enforcement guarantees that future additions (e.g., a standalone Billing Module) can be introduced into the domains directory. They will inject existing application services for validation without requiring modifications to existing core module code. If the application outgrows a monolith, a module can be extracted, and its injected service calls replaced with internal network calls, keeping the surrounding application architecture completely intact.
