# DevPilot AI - API Design Document

## 1. Introduction
**Purpose:** This document serves as the official API contract for the DevPilot AI platform. It outlines the RESTful interfaces used by frontend clients and third-party integrations to interact with the backend services.
**Scope:** Covers all HTTP REST endpoints, including request/response formats, headers, authentication mechanisms, error handling, and core module APIs.
**Audience:** Frontend Developers, Backend Developers, Quality Assurance Engineers, and API Consumers.
**References:**
- Requirement.md (Functional Requirements)
- HLD.md (High-Level Architecture)
- LLD.md (Module-level Design)
- database-design.md (Logical Database Schema)

## 2. API Design Principles
- **RESTful Design:** All APIs adhere to standard REST principles, treating business entities as resources.
- **Resource-Oriented URLs:** URLs represent entities (nouns) rather than actions (verbs).
- **Stateless Communication:** Each request contains all information necessary for the server to process it. No client state is stored on the server between requests.
- **Consistent Naming:** Standardized naming conventions (camelCase for JSON fields, lowercase-kebab-case for URLs).
- **Predictable Responses:** Uniform response structures for both success and error scenarios.
- **Idempotency:** Safe methods (`GET`, `PUT`, `DELETE`) are designed to be idempotent.
- **Security First:** Strict enforcement of HTTPS, JWT authentication, RBAC authorization, and input validation.

## 3. Base URL
All API requests are made to the base URL:
```text
/api/v1
```

## 4. API Versioning
The API uses URI versioning to ensure backward compatibility as the platform evolves. The version number is prefixed to the endpoint path.
**Example:**
- Current version: `/api/v1`
- Future version: `/api/v2`

## 5. Authentication
DevPilot AI uses JWT (JSON Web Tokens) for secure authentication.
- **JWT Access Token:** Short-lived token used to authenticate API requests.
- **Refresh Token:** Long-lived token used to obtain a new Access Token when it expires, securely stored as an HTTP-only cookie.
- **Authorization Header:** Requires the `Bearer` schema. Format: `Authorization: Bearer <access_token>`
- **Bearer Token:** Included in the header for all protected API calls.
- **Token Expiration:** Access tokens typically expire in 15 minutes; refresh tokens expire in 7 days.
- **Protected APIs:** Require a valid access token. Requests without one return `401 Unauthorized`.
- **Public APIs:** Accessible without authentication (e.g., login, register, forgot password).

## 6. Authorization
Authorization is enforced using Role-Based Access Control (RBAC) at the workspace level, ensuring Tenant Isolation and Workspace Isolation.
- **Owner:** Full administrative access, including billing and deleting the workspace.
- **Admin:** Can manage projects, workspace settings, and members, but cannot delete the workspace.
- **Developer:** Can create/edit/delete entities like projects, stories, and sprint tasks.
- **Viewer:** Read-only access to workspace resources.

## 7. REST API Standards
- **HTTP Methods:**
  - `GET`: Retrieve a resource or collection.
  - `POST`: Create a new resource or execute a complex operation.
  - `PUT`: Completely replace an existing resource.
  - `PATCH`: Partially update an existing resource.
  - `DELETE`: Remove a resource.
- **Naming Conventions:**
  - **Plural resources:** Use plural nouns (e.g., `/workspaces`, not `/workspace`).
  - **Resource hierarchy:** Structure URLs to reflect relationships.
  - **Nested resources:** Used when an entity strictly belongs to another (e.g., `/projects/{projectId}/stories`).

## 8. URI Standards
**Examples:**
- `/workspaces`
- `/projects`
- `/projects/{projectId}`
- `/projects/{projectId}/stories`
- `/workspaces/{workspaceId}/members`

## 9. Request Standards
**Headers:**
- `Content-Type`: Must be `application/json` for requests with a body.
- `Authorization`: `Bearer <token>` for authenticated endpoints.
- `Accept`: Must be `application/json`.
**Request Body Rules:** Use camelCase for keys. Extra properties should be ignored by the server.
**Query Parameters:** Use camelCase for pagination, filtering, and sorting (e.g., `?sortBy=createdAt&page=1`).
**Path Parameters:** Use camelCase for dynamic identifiers (e.g., `{projectId}`).

## 10. Response Standards
**Common Success Response:**
```json
{
  "success": true,
  "data": { },
  "metadata": {
    "timestamp": "2024-05-01T12:00:00.000Z",
    "requestId": "req-1234abcd"
  }
}
```
**Common Error Response:**
```json
{
  "success": false,
  "error": { },
  "metadata": {
    "timestamp": "2024-05-01T12:00:00.000Z",
    "requestId": "req-1234abcd"
  }
}
```

## 11. HTTP Status Codes
- **200 OK:** Request succeeded.
- **201 Created:** Resource successfully created.
- **202 Accepted:** Request accepted for asynchronous processing.
- **204 No Content:** Request succeeded, but no body is returned.
- **400 Bad Request:** Invalid request syntax or validation error.
- **401 Unauthorized:** Missing or invalid authentication token.
- **403 Forbidden:** Authenticated, but lacks required permissions.
- **404 Not Found:** Resource does not exist.
- **409 Conflict:** Resource state conflict.
- **422 Unprocessable Entity:** Semantic validation failed.
- **429 Too Many Requests:** Rate limit exceeded.
- **500 Internal Server Error:** Unexpected backend failure.

## 12. Error Response Standard
```json
{
  "success": false,
  "error": {
    "errorCode": "VALIDATION_ERROR",
    "message": "Invalid input provided.",
    "details": [
      {
        "field": "email",
        "issue": "Must be a valid email format."
      }
    ]
  },
  "metadata": {
    "traceId": "trace-9876xyz",
    "timestamp": "2024-05-01T12:00:05.123Z",
    "requestId": "req-1234abcd"
  }
}
```

## 13. Validation Rules
- **Required Fields:** Missing required fields return `400 Bad Request`.
- **String Length:** Configured limits apply (e.g., title max 255 chars).
- **Enums:** Must strictly match predefined uppercase values.
- **Email:** Must comply with RFC 5322.
- **UUID:** Identifiers must be valid UUIDv4 strings.
- **Date:** Must be ISO 8601 formatted strings.
- **Number:** Min/Max constraints apply.
- **Boolean:** Must be strict boolean.
- **Unique Constraints:** Duplicates return `409 Conflict`.

## 14. Pagination
- `page`: Current page number (1-indexed).
- `limit`: Number of items per page.
- `totalRecords`: Total number of records across all pages.
- `totalPages`: Total number of pages.
- `hasNext`: Boolean indicating if a next page exists.
- `hasPrevious`: Boolean indicating if a previous page exists.

## 15. Filtering
**Examples:**
- `?status=IN_PROGRESS`
- `?priority=HIGH`
- `?role=DEVELOPER`
- `?createdAfter=2024-01-01T00:00:00Z`
- `?createdBefore=2024-05-01T00:00:00Z`

## 16. Sorting
- `sortBy`: The field to sort by.
- `order`: `asc` or `desc`.

## 17. Searching
- `search`: Full-text search keyword query.

## 18. Rate Limiting
Standard rate limiting applies to prevent abuse. E.g., 100 requests per minute per authenticated user IP, and 10 requests per minute for public endpoints. Header fields like `X-RateLimit-Remaining` are returned.

## 19. Security
- **HTTPS:** Enforced for all API traffic.
- **JWT:** Cryptographically signed tokens.
- **Input Validation:** Strict parsing to prevent injection.
- **Output Encoding:** Prevents XSS.
- **CORS:** Configured for trusted domains.
- **Audit Logging:** Keeps track of critical actions.
- **Least Privilege:** Handled via RBAC.

## 20. Common Headers
- `Authorization`: `Bearer <token>`
- `Content-Type`: `application/json`
- `Accept`: `application/json`
- `X-Request-ID`: `uuid`

## 21. API Lifecycle
- **Draft:** Unstable, in development.
- **Stable:** Production ready.
- **Deprecated:** Slated for removal, uses `Warning` header.
- **Retired:** Offline, returns 404 or 410.

---

## Module 1: Authentication API

### 1. Register
- **Endpoint:** `POST /api/v1/auth/register`
- **Description:** Registers a new user.
- **Authorization:** Public
- **Request Headers:** `Content-Type: application/json`
- **Path Parameters:** None
- **Query Parameters:** None
- **Request Body:**
```json
{
  "fullName": "John Doe",
  "email": "john@example.com",
  "password": "Password123!"
}
```
- **Validation Rules:** email (valid format), password (min 8 chars, strong), fullName (required).
- **Success Response:** 201 Created
- **Error Responses:** 400 Bad Request, 409 Conflict
- **Example Request:**
```http
POST /api/v1/auth/register
Content-Type: application/json

{ "fullName": "John Doe", "email": "john@example.com", "password": "Password123!" }
```
- **Example Response:**
```json
{
  "success": true,
  "data": { "userId": "uuid-1234", "email": "john@example.com" },
  "metadata": { "timestamp": "2024-05-01T12:00:00Z", "requestId": "req-1" }
}
```

### 2. Login
- **Endpoint:** `POST /api/v1/auth/login`
- **Description:** Authenticates user and issues tokens.
- **Authorization:** Public
- **Request Headers:** `Content-Type: application/json`
- **Path Parameters:** None
- **Query Parameters:** None
- **Request Body:**
```json
{
  "email": "john@example.com",
  "password": "Password123!"
}
```
- **Validation Rules:** email, password (required).
- **Success Response:** 200 OK
- **Error Responses:** 400 Bad Request, 401 Unauthorized
- **Example Request:**
```http
POST /api/v1/auth/login
Content-Type: application/json

{ "email": "john@example.com", "password": "Password123!" }
```
- **Example Response:**
```json
{
  "success": true,
  "data": { "accessToken": "eyJhb...", "user": { "userId": "uuid-1", "email": "john@example.com" } },
  "metadata": { "timestamp": "2024-05-01T12:00:00Z", "requestId": "req-2" }
}
```

### 3. Logout
- **Endpoint:** `POST /api/v1/auth/logout`
- **Description:** Invalidates session/tokens.
- **Authorization:** Bearer Token
- **Request Headers:** `Authorization: Bearer <token>`
- **Path Parameters:** None
- **Query Parameters:** None
- **Request Body:** None
- **Validation Rules:** None
- **Success Response:** 200 OK
- **Error Responses:** 401 Unauthorized
- **Example Request:**
```http
POST /api/v1/auth/logout
Authorization: Bearer eyJhb...
```
- **Example Response:**
```json
{ "success": true, "data": { "message": "Logged out" }, "metadata": { "timestamp": "2024-05-01T12:00:00Z", "requestId": "req-3" } }
```

### 4. Refresh Token
- **Endpoint:** `POST /api/v1/auth/refresh`
- **Description:** Obtains new access token via HTTP-only cookie.
- **Authorization:** Public (uses cookie)
- **Request Headers:** None
- **Path Parameters:** None
- **Query Parameters:** None
- **Request Body:** None
- **Validation Rules:** Refresh token cookie must be present and valid.
- **Success Response:** 200 OK
- **Error Responses:** 401 Unauthorized
- **Example Request:**
```http
POST /api/v1/auth/refresh
Cookie: refreshToken=eyJhb...
```
- **Example Response:**
```json
{ "success": true, "data": { "accessToken": "eyJhb...new" }, "metadata": { "timestamp": "2024-05-01T12:00:00Z", "requestId": "req-4" } }
```

### 5. Forgot Password
- **Endpoint:** `POST /api/v1/auth/forgot-password`
- **Description:** Triggers password reset email.
- **Authorization:** Public
- **Request Headers:** `Content-Type: application/json`
- **Path Parameters:** None
- **Query Parameters:** None
- **Request Body:** `{ "email": "john@example.com" }`
- **Validation Rules:** email (valid format).
- **Success Response:** 200 OK
- **Error Responses:** 400 Bad Request
- **Example Request:**
```http
POST /api/v1/auth/forgot-password
Content-Type: application/json

{ "email": "john@example.com" }
```
- **Example Response:**
```json
{ "success": true, "data": { "message": "Email sent" }, "metadata": { "timestamp": "2024-05-01T12:00:00Z", "requestId": "req-5" } }
```

### 6. Reset Password
- **Endpoint:** `POST /api/v1/auth/reset-password`
- **Description:** Resets password using token.
- **Authorization:** Public
- **Request Headers:** `Content-Type: application/json`
- **Path Parameters:** None
- **Query Parameters:** None
- **Request Body:** `{ "token": "xyz", "newPassword": "NewPassword1!" }`
- **Validation Rules:** token (required), newPassword (strong).
- **Success Response:** 200 OK
- **Error Responses:** 400 Bad Request, 401 Unauthorized
- **Example Request:**
```http
POST /api/v1/auth/reset-password
Content-Type: application/json

{ "token": "xyz", "newPassword": "NewPassword1!" }
```
- **Example Response:**
```json
{ "success": true, "data": { "message": "Password reset successful" }, "metadata": { "timestamp": "2024-05-01T12:00:00Z", "requestId": "req-6" } }
```

### 7. Change Password
- **Endpoint:** `POST /api/v1/auth/change-password`
- **Description:** Changes password for logged in user.
- **Authorization:** Bearer Token
- **Request Headers:** `Content-Type: application/json`, `Authorization: Bearer <token>`
- **Path Parameters:** None
- **Query Parameters:** None
- **Request Body:** `{ "currentPassword": "old", "newPassword": "new" }`
- **Validation Rules:** currentPassword (required), newPassword (strong, diff from old).
- **Success Response:** 200 OK
- **Error Responses:** 400 Bad Request, 401 Unauthorized
- **Example Request:**
```http
POST /api/v1/auth/change-password
Content-Type: application/json
Authorization: Bearer eyJhb...

{ "currentPassword": "old", "newPassword": "new" }
```
- **Example Response:**
```json
{ "success": true, "data": { "message": "Password changed" }, "metadata": { "timestamp": "2024-05-01T12:00:00Z", "requestId": "req-7" } }
```

### 8. Current User
- **Endpoint:** `GET /api/v1/auth/me`
- **Description:** Returns current user profile.
- **Authorization:** Bearer Token
- **Request Headers:** `Authorization: Bearer <token>`
- **Path Parameters:** None
- **Query Parameters:** None
- **Request Body:** None
- **Validation Rules:** None
- **Success Response:** 200 OK
- **Error Responses:** 401 Unauthorized
- **Example Request:**
```http
GET /api/v1/auth/me
Authorization: Bearer eyJhb...
```
- **Example Response:**
```json
{ "success": true, "data": { "userId": "uuid-1", "fullName": "John Doe", "email": "john@example.com" }, "metadata": { "timestamp": "2024-05-01T12:00:00Z", "requestId": "req-8" } }
```

---

## Module 2: Workspace API

### 1. CRUD Workspace
- **Endpoint:** `POST /api/v1/workspaces`
- **Description:** Creates a new workspace.
- **Authorization:** Bearer Token
- **Request Headers:** `Content-Type: application/json`, `Authorization: Bearer <token>`
- **Path Parameters:** None
- **Query Parameters:** None
- **Request Body:** `{ "name": "Acme", "slug": "acme" }`
- **Validation Rules:** name (required), slug (unique, alphanumeric with hyphens).
- **Success Response:** 201 Created
- **Error Responses:** 400 Bad Request, 409 Conflict
- **Example Request:**
```http
POST /api/v1/workspaces
Content-Type: application/json
Authorization: Bearer eyJhb...

{ "name": "Acme", "slug": "acme" }
```
- **Example Response:**
```json
{ "success": true, "data": { "workspaceId": "ws-1", "name": "Acme", "slug": "acme" }, "metadata": { "timestamp": "2024-05-01T12:00:00Z", "requestId": "req-9" } }
```

*(Other CRUD omitted for brevity but follows standard GET, PUT, DELETE structure on /workspaces/{workspaceId})*

### 2. Workspace Members
- **Endpoint:** `GET /api/v1/workspaces/{workspaceId}/members`
- **Description:** Lists workspace members.
- **Authorization:** Bearer Token (Workspace Member)
- **Request Headers:** `Authorization: Bearer <token>`
- **Path Parameters:** `workspaceId` (UUID)
- **Query Parameters:** `page`, `limit`
- **Request Body:** None
- **Validation Rules:** Valid workspaceId.
- **Success Response:** 200 OK
- **Error Responses:** 401 Unauthorized, 403 Forbidden, 404 Not Found
- **Example Request:**
```http
GET /api/v1/workspaces/ws-1/members?page=1&limit=20
Authorization: Bearer eyJhb...
```
- **Example Response:**
```json
{ "success": true, "data": [ { "userId": "uuid-1", "role": "OWNER" } ], "metadata": { "timestamp": "2024-05-01T12:00:00Z", "requestId": "req-10" } }
```

### 3. Invite Member
- **Endpoint:** `POST /api/v1/workspaces/{workspaceId}/members/invite`
- **Description:** Invites user to workspace.
- **Authorization:** Bearer Token (Admin, Owner)
- **Request Headers:** `Content-Type: application/json`, `Authorization: Bearer <token>`
- **Path Parameters:** `workspaceId` (UUID)
- **Query Parameters:** None
- **Request Body:** `{ "email": "alice@example.com", "role": "DEVELOPER" }`
- **Validation Rules:** email (valid), role (valid enum).
- **Success Response:** 200 OK
- **Error Responses:** 400 Bad Request, 403 Forbidden, 404 Not Found
- **Example Request:**
```http
POST /api/v1/workspaces/ws-1/members/invite
Content-Type: application/json
Authorization: Bearer eyJhb...

{ "email": "alice@example.com", "role": "DEVELOPER" }
```
- **Example Response:**
```json
{ "success": true, "data": { "message": "Invitation sent" }, "metadata": { "timestamp": "2024-05-01T12:00:00Z", "requestId": "req-11" } }
```

### 4. Remove Member
- **Endpoint:** `DELETE /api/v1/workspaces/{workspaceId}/members/{userId}`
- **Description:** Removes user from workspace.
- **Authorization:** Bearer Token (Admin, Owner)
- **Request Headers:** `Authorization: Bearer <token>`
- **Path Parameters:** `workspaceId` (UUID), `userId` (UUID)
- **Query Parameters:** None
- **Request Body:** None
- **Validation Rules:** Valid IDs.
- **Success Response:** 204 No Content
- **Error Responses:** 403 Forbidden, 404 Not Found
- **Example Request:**
```http
DELETE /api/v1/workspaces/ws-1/members/uuid-2
Authorization: Bearer eyJhb...
```
- **Example Response:** *(Empty Body)*

### 5. Update Member Role
- **Endpoint:** `PATCH /api/v1/workspaces/{workspaceId}/members/{userId}/role`
- **Description:** Updates a member's role.
- **Authorization:** Bearer Token (Owner)
- **Request Headers:** `Content-Type: application/json`, `Authorization: Bearer <token>`
- **Path Parameters:** `workspaceId` (UUID), `userId` (UUID)
- **Query Parameters:** None
- **Request Body:** `{ "role": "ADMIN" }`
- **Validation Rules:** role (valid enum).
- **Success Response:** 200 OK
- **Error Responses:** 400 Bad Request, 403 Forbidden, 404 Not Found
- **Example Request:**
```http
PATCH /api/v1/workspaces/ws-1/members/uuid-2/role
Content-Type: application/json
Authorization: Bearer eyJhb...

{ "role": "ADMIN" }
```
- **Example Response:**
```json
{ "success": true, "data": { "message": "Role updated" }, "metadata": { "timestamp": "2024-05-01T12:00:00Z", "requestId": "req-12" } }
```

---

## Module 3: User API

### 1. User Profile
- **Endpoint:** `GET /api/v1/users/{userId}`
- **Description:** Retrieves user profile.
- **Authorization:** Bearer Token
- **Request Headers:** `Authorization: Bearer <token>`
- **Path Parameters:** `userId` (UUID)
- **Query Parameters:** None
- **Request Body:** None
- **Validation Rules:** Valid UUID.
- **Success Response:** 200 OK
- **Error Responses:** 401 Unauthorized, 404 Not Found
- **Example Request:**
```http
GET /api/v1/users/uuid-1
Authorization: Bearer eyJhb...
```
- **Example Response:**
```json
{ "success": true, "data": { "userId": "uuid-1", "fullName": "John Doe", "avatarUrl": null }, "metadata": { "timestamp": "2024-05-01T12:00:00Z", "requestId": "req-13" } }
```

### 2. Update Profile
- **Endpoint:** `PATCH /api/v1/users/{userId}`
- **Description:** Updates user profile information.
- **Authorization:** Bearer Token (Self)
- **Request Headers:** `Content-Type: application/json`, `Authorization: Bearer <token>`
- **Path Parameters:** `userId` (UUID)
- **Query Parameters:** None
- **Request Body:** `{ "fullName": "Johnny Doe", "bio": "Dev" }`
- **Validation Rules:** Valid UUID, self-modification only.
- **Success Response:** 200 OK
- **Error Responses:** 400 Bad Request, 403 Forbidden
- **Example Request:**
```http
PATCH /api/v1/users/uuid-1
Content-Type: application/json
Authorization: Bearer eyJhb...

{ "fullName": "Johnny Doe", "bio": "Dev" }
```
- **Example Response:**
```json
{ "success": true, "data": { "userId": "uuid-1", "fullName": "Johnny Doe" }, "metadata": { "timestamp": "2024-05-01T12:00:00Z", "requestId": "req-14" } }
```

### 3. Avatar
- **Endpoint:** `POST /api/v1/users/{userId}/avatar`
- **Description:** Uploads a user avatar.
- **Authorization:** Bearer Token (Self)
- **Request Headers:** `Content-Type: multipart/form-data`, `Authorization: Bearer <token>`
- **Path Parameters:** `userId` (UUID)
- **Query Parameters:** None
- **Request Body:** Binary image file
- **Validation Rules:** Max size 5MB, valid image format (png/jpeg).
- **Success Response:** 200 OK
- **Error Responses:** 400 Bad Request, 403 Forbidden
- **Example Request:**
```http
POST /api/v1/users/uuid-1/avatar
Content-Type: multipart/form-data; boundary=---123
Authorization: Bearer eyJhb...
```
- **Example Response:**
```json
{ "success": true, "data": { "avatarUrl": "https://cdn.example.com/avatars/uuid-1.png" }, "metadata": { "timestamp": "2024-05-01T12:00:00Z", "requestId": "req-15" } }
```

### 4. User Preferences
- **Endpoint:** `GET /api/v1/users/{userId}/preferences`
- **Description:** Retrieves user preferences.
- **Authorization:** Bearer Token (Self)
- **Request Headers:** `Authorization: Bearer <token>`
- **Path Parameters:** `userId` (UUID)
- **Query Parameters:** None
- **Request Body:** None
- **Validation Rules:** Valid UUID.
- **Success Response:** 200 OK
- **Error Responses:** 403 Forbidden
- **Example Request:**
```http
GET /api/v1/users/uuid-1/preferences
Authorization: Bearer eyJhb...
```
- **Example Response:**
```json
{ "success": true, "data": { "theme": "dark", "notificationsEnabled": true }, "metadata": { "timestamp": "2024-05-01T12:00:00Z", "requestId": "req-16" } }
```

---

## Module 4: Project API

### 1. CRUD Project
- **Endpoint:** `POST /api/v1/workspaces/{workspaceId}/projects`
- **Description:** Creates a new project in a workspace.
- **Authorization:** Bearer Token (Admin, Owner, Developer)
- **Request Headers:** `Content-Type: application/json`, `Authorization: Bearer <token>`
- **Path Parameters:** `workspaceId` (UUID)
- **Query Parameters:** None
- **Request Body:** `{ "name": "Frontend", "key": "FE", "description": "UI" }`
- **Validation Rules:** name (required), key (required, unique per workspace, uppercase alphanumeric).
- **Success Response:** 201 Created
- **Error Responses:** 400 Bad Request, 403 Forbidden, 409 Conflict
- **Example Request:**
```http
POST /api/v1/workspaces/ws-1/projects
Content-Type: application/json
Authorization: Bearer eyJhb...

{ "name": "Frontend", "key": "FE", "description": "UI" }
```
- **Example Response:**
```json
{ "success": true, "data": { "projectId": "proj-1", "name": "Frontend", "key": "FE" }, "metadata": { "timestamp": "2024-05-01T12:00:00Z", "requestId": "req-17" } }
```

### 2. Archive
- **Endpoint:** `POST /api/v1/projects/{projectId}/archive`
- **Description:** Archives a project.
- **Authorization:** Bearer Token (Admin, Owner)
- **Request Headers:** `Authorization: Bearer <token>`
- **Path Parameters:** `projectId` (UUID)
- **Query Parameters:** None
- **Request Body:** None
- **Validation Rules:** Valid projectId.
- **Success Response:** 200 OK
- **Error Responses:** 403 Forbidden, 404 Not Found
- **Example Request:**
```http
POST /api/v1/projects/proj-1/archive
Authorization: Bearer eyJhb...
```
- **Example Response:**
```json
{ "success": true, "data": { "message": "Project archived" }, "metadata": { "timestamp": "2024-05-01T12:00:00Z", "requestId": "req-18" } }
```

### 3. Restore
- **Endpoint:** `POST /api/v1/projects/{projectId}/restore`
- **Description:** Restores an archived project.
- **Authorization:** Bearer Token (Admin, Owner)
- **Request Headers:** `Authorization: Bearer <token>`
- **Path Parameters:** `projectId` (UUID)
- **Query Parameters:** None
- **Request Body:** None
- **Validation Rules:** Valid projectId.
- **Success Response:** 200 OK
- **Error Responses:** 403 Forbidden, 404 Not Found
- **Example Request:**
```http
POST /api/v1/projects/proj-1/restore
Authorization: Bearer eyJhb...
```
- **Example Response:**
```json
{ "success": true, "data": { "message": "Project restored" }, "metadata": { "timestamp": "2024-05-01T12:00:00Z", "requestId": "req-19" } }
```

### 4. Members
- **Endpoint:** `GET /api/v1/projects/{projectId}/members`
- **Description:** Lists members of a project.
- **Authorization:** Bearer Token
- **Request Headers:** `Authorization: Bearer <token>`
- **Path Parameters:** `projectId` (UUID)
- **Query Parameters:** None
- **Request Body:** None
- **Validation Rules:** Valid projectId.
- **Success Response:** 200 OK
- **Error Responses:** 403 Forbidden, 404 Not Found
- **Example Request:**
```http
GET /api/v1/projects/proj-1/members
Authorization: Bearer eyJhb...
```
- **Example Response:**
```json
{ "success": true, "data": [ { "userId": "uuid-1", "role": "DEVELOPER" } ], "metadata": { "timestamp": "2024-05-01T12:00:00Z", "requestId": "req-20" } }
```

---

## Module 5: Documentation API

### 1. CRUD Documents
- **Endpoint:** `POST /api/v1/projects/{projectId}/docs`
- **Description:** Creates a new document.
- **Authorization:** Bearer Token
- **Request Headers:** `Content-Type: application/json`, `Authorization: Bearer <token>`
- **Path Parameters:** `projectId` (UUID)
- **Query Parameters:** None
- **Request Body:** `{ "title": "Setup", "content": "# Guide", "folderId": null }`
- **Validation Rules:** title (required), content (string).
- **Success Response:** 201 Created
- **Error Responses:** 400 Bad Request
- **Example Request:**
```http
POST /api/v1/projects/proj-1/docs
Content-Type: application/json
Authorization: Bearer eyJhb...

{ "title": "Setup", "content": "# Guide", "folderId": null }
```
- **Example Response:**
```json
{ "success": true, "data": { "docId": "doc-1", "title": "Setup" }, "metadata": { "timestamp": "2024-05-01T12:00:00Z", "requestId": "req-21" } }
```

### 2. Draft
- **Endpoint:** `POST /api/v1/docs/{docId}/draft`
- **Description:** Saves a draft version of a document.
- **Authorization:** Bearer Token
- **Request Headers:** `Content-Type: application/json`, `Authorization: Bearer <token>`
- **Path Parameters:** `docId` (UUID)
- **Query Parameters:** None
- **Request Body:** `{ "content": "# Updated Guide" }`
- **Validation Rules:** content (string).
- **Success Response:** 200 OK
- **Error Responses:** 400 Bad Request
- **Example Request:**
```http
POST /api/v1/docs/doc-1/draft
Content-Type: application/json
Authorization: Bearer eyJhb...

{ "content": "# Updated Guide" }
```
- **Example Response:**
```json
{ "success": true, "data": { "message": "Draft saved" }, "metadata": { "timestamp": "2024-05-01T12:00:00Z", "requestId": "req-22" } }
```

### 3. Publish
- **Endpoint:** `POST /api/v1/docs/{docId}/publish`
- **Description:** Publishes the current draft.
- **Authorization:** Bearer Token
- **Request Headers:** `Authorization: Bearer <token>`
- **Path Parameters:** `docId` (UUID)
- **Query Parameters:** None
- **Request Body:** None
- **Validation Rules:** None
- **Success Response:** 200 OK
- **Error Responses:** 404 Not Found
- **Example Request:**
```http
POST /api/v1/docs/doc-1/publish
Authorization: Bearer eyJhb...
```
- **Example Response:**
```json
{ "success": true, "data": { "message": "Document published" }, "metadata": { "timestamp": "2024-05-01T12:00:00Z", "requestId": "req-23" } }
```

### 4. Version History
- **Endpoint:** `GET /api/v1/docs/{docId}/versions`
- **Description:** Retrieves version history of a document.
- **Authorization:** Bearer Token
- **Request Headers:** `Authorization: Bearer <token>`
- **Path Parameters:** `docId` (UUID)
- **Query Parameters:** None
- **Request Body:** None
- **Validation Rules:** Valid docId.
- **Success Response:** 200 OK
- **Error Responses:** 404 Not Found
- **Example Request:**
```http
GET /api/v1/docs/doc-1/versions
Authorization: Bearer eyJhb...
```
- **Example Response:**
```json
{ "success": true, "data": [ { "versionId": "v1", "createdAt": "2024-05-01T12:00:00Z" } ], "metadata": { "timestamp": "2024-05-01T12:00:00Z", "requestId": "req-24" } }
```

### 5. Comments
- **Endpoint:** `POST /api/v1/docs/{docId}/comments`
- **Description:** Adds a comment to a document.
- **Authorization:** Bearer Token
- **Request Headers:** `Content-Type: application/json`, `Authorization: Bearer <token>`
- **Path Parameters:** `docId` (UUID)
- **Query Parameters:** None
- **Request Body:** `{ "content": "Looks good!" }`
- **Validation Rules:** content (required).
- **Success Response:** 201 Created
- **Error Responses:** 400 Bad Request
- **Example Request:**
```http
POST /api/v1/docs/doc-1/comments
Content-Type: application/json
Authorization: Bearer eyJhb...

{ "content": "Looks good!" }
```
- **Example Response:**
```json
{ "success": true, "data": { "commentId": "cmt-1", "content": "Looks good!" }, "metadata": { "timestamp": "2024-05-01T12:00:00Z", "requestId": "req-25" } }
```

---

## Module 6: Story API

### 1. CRUD Story
- **Endpoint:** `POST /api/v1/projects/{projectId}/stories`
- **Description:** Creates a new story.
- **Authorization:** Bearer Token
- **Request Headers:** `Content-Type: application/json`, `Authorization: Bearer <token>`
- **Path Parameters:** `projectId` (UUID)
- **Query Parameters:** None
- **Request Body:** `{ "title": "Login", "description": "Flow", "type": "FEATURE", "points": 5 }`
- **Validation Rules:** title, type (enum) required.
- **Success Response:** 201 Created
- **Error Responses:** 400 Bad Request
- **Example Request:**
```http
POST /api/v1/projects/proj-1/stories
Content-Type: application/json
Authorization: Bearer eyJhb...

{ "title": "Login", "description": "Flow", "type": "FEATURE", "points": 5 }
```
- **Example Response:**
```json
{ "success": true, "data": { "storyId": "story-1", "title": "Login" }, "metadata": { "timestamp": "2024-05-01T12:00:00Z", "requestId": "req-26" } }
```

### 2. Status
- **Endpoint:** `PATCH /api/v1/stories/{storyId}/status`
- **Description:** Updates story status.
- **Authorization:** Bearer Token
- **Request Headers:** `Content-Type: application/json`, `Authorization: Bearer <token>`
- **Path Parameters:** `storyId` (UUID)
- **Query Parameters:** None
- **Request Body:** `{ "status": "IN_PROGRESS" }`
- **Validation Rules:** status (valid enum).
- **Success Response:** 200 OK
- **Error Responses:** 400 Bad Request
- **Example Request:**
```http
PATCH /api/v1/stories/story-1/status
Content-Type: application/json
Authorization: Bearer eyJhb...

{ "status": "IN_PROGRESS" }
```
- **Example Response:**
```json
{ "success": true, "data": { "message": "Status updated" }, "metadata": { "timestamp": "2024-05-01T12:00:00Z", "requestId": "req-27" } }
```

### 3. Priority
- **Endpoint:** `PATCH /api/v1/stories/{storyId}/priority`
- **Description:** Updates story priority.
- **Authorization:** Bearer Token
- **Request Headers:** `Content-Type: application/json`, `Authorization: Bearer <token>`
- **Path Parameters:** `storyId` (UUID)
- **Query Parameters:** None
- **Request Body:** `{ "priority": "HIGH" }`
- **Validation Rules:** priority (valid enum).
- **Success Response:** 200 OK
- **Error Responses:** 400 Bad Request
- **Example Request:**
```http
PATCH /api/v1/stories/story-1/priority
Content-Type: application/json
Authorization: Bearer eyJhb...

{ "priority": "HIGH" }
```
- **Example Response:**
```json
{ "success": true, "data": { "message": "Priority updated" }, "metadata": { "timestamp": "2024-05-01T12:00:00Z", "requestId": "req-28" } }
```

### 4. Assignee
- **Endpoint:** `PATCH /api/v1/stories/{storyId}/assignee`
- **Description:** Assigns story to user.
- **Authorization:** Bearer Token
- **Request Headers:** `Content-Type: application/json`, `Authorization: Bearer <token>`
- **Path Parameters:** `storyId` (UUID)
- **Query Parameters:** None
- **Request Body:** `{ "assigneeId": "uuid-1234" }`
- **Validation Rules:** assigneeId (valid UUID or null).
- **Success Response:** 200 OK
- **Error Responses:** 400 Bad Request
- **Example Request:**
```http
PATCH /api/v1/stories/story-1/assignee
Content-Type: application/json
Authorization: Bearer eyJhb...

{ "assigneeId": "uuid-1234" }
```
- **Example Response:**
```json
{ "success": true, "data": { "message": "Assignee updated" }, "metadata": { "timestamp": "2024-05-01T12:00:00Z", "requestId": "req-29" } }
```

### 5. Labels
- **Endpoint:** `POST /api/v1/stories/{storyId}/labels`
- **Description:** Adds a label to a story.
- **Authorization:** Bearer Token
- **Request Headers:** `Content-Type: application/json`, `Authorization: Bearer <token>`
- **Path Parameters:** `storyId` (UUID)
- **Query Parameters:** None
- **Request Body:** `{ "labelId": "label-1" }`
- **Validation Rules:** labelId (valid UUID).
- **Success Response:** 201 Created
- **Error Responses:** 400 Bad Request
- **Example Request:**
```http
POST /api/v1/stories/story-1/labels
Content-Type: application/json
Authorization: Bearer eyJhb...

{ "labelId": "label-1" }
```
- **Example Response:**
```json
{ "success": true, "data": { "message": "Label added" }, "metadata": { "timestamp": "2024-05-01T12:00:00Z", "requestId": "req-30" } }
```

### 6. Comments
- **Endpoint:** `POST /api/v1/stories/{storyId}/comments`
- **Description:** Adds a comment to a story.
- **Authorization:** Bearer Token
- **Request Headers:** `Content-Type: application/json`, `Authorization: Bearer <token>`
- **Path Parameters:** `storyId` (UUID)
- **Query Parameters:** None
- **Request Body:** `{ "content": "Started work" }`
- **Validation Rules:** content (required).
- **Success Response:** 201 Created
- **Error Responses:** 400 Bad Request
- **Example Request:**
```http
POST /api/v1/stories/story-1/comments
Content-Type: application/json
Authorization: Bearer eyJhb...

{ "content": "Started work" }
```
- **Example Response:**
```json
{ "success": true, "data": { "commentId": "cmt-2" }, "metadata": { "timestamp": "2024-05-01T12:00:00Z", "requestId": "req-31" } }
```

---

## Module 7: Sprint API

### 1. CRUD Sprint
- **Endpoint:** `POST /api/v1/projects/{projectId}/sprints`
- **Description:** Creates a new sprint.
- **Authorization:** Bearer Token
- **Request Headers:** `Content-Type: application/json`, `Authorization: Bearer <token>`
- **Path Parameters:** `projectId` (UUID)
- **Query Parameters:** None
- **Request Body:** `{ "name": "Sprint 1", "startDate": "2024-06-01T00:00:00Z", "endDate": "2024-06-14T23:59:59Z", "goal": "Setup auth" }`
- **Validation Rules:** name, dates required.
- **Success Response:** 201 Created
- **Error Responses:** 400 Bad Request
- **Example Request:**
```http
POST /api/v1/projects/proj-1/sprints
Content-Type: application/json
Authorization: Bearer eyJhb...

{ "name": "Sprint 1", "startDate": "2024-06-01T00:00:00Z", "endDate": "2024-06-14T23:59:59Z", "goal": "Setup auth" }
```
- **Example Response:**
```json
{ "success": true, "data": { "sprintId": "sprint-1", "name": "Sprint 1" }, "metadata": { "timestamp": "2024-05-01T12:00:00Z", "requestId": "req-32" } }
```

### 2. Start Sprint
- **Endpoint:** `POST /api/v1/sprints/{sprintId}/start`
- **Description:** Starts a sprint.
- **Authorization:** Bearer Token
- **Request Headers:** `Authorization: Bearer <token>`
- **Path Parameters:** `sprintId` (UUID)
- **Query Parameters:** None
- **Request Body:** None
- **Validation Rules:** Sprint must be in planned state.
- **Success Response:** 200 OK
- **Error Responses:** 409 Conflict
- **Example Request:**
```http
POST /api/v1/sprints/sprint-1/start
Authorization: Bearer eyJhb...
```
- **Example Response:**
```json
{ "success": true, "data": { "message": "Sprint started" }, "metadata": { "timestamp": "2024-05-01T12:00:00Z", "requestId": "req-33" } }
```

### 3. Complete Sprint
- **Endpoint:** `POST /api/v1/sprints/{sprintId}/complete`
- **Description:** Completes a sprint.
- **Authorization:** Bearer Token
- **Request Headers:** `Authorization: Bearer <token>`
- **Path Parameters:** `sprintId` (UUID)
- **Query Parameters:** None
- **Request Body:** None
- **Validation Rules:** Sprint must be active.
- **Success Response:** 200 OK
- **Error Responses:** 409 Conflict
- **Example Request:**
```http
POST /api/v1/sprints/sprint-1/complete
Authorization: Bearer eyJhb...
```
- **Example Response:**
```json
{ "success": true, "data": { "message": "Sprint completed" }, "metadata": { "timestamp": "2024-05-01T12:00:00Z", "requestId": "req-34" } }
```

### 4. Sprint Backlog
- **Endpoint:** `POST /api/v1/sprints/{sprintId}/stories`
- **Description:** Adds a story to a sprint.
- **Authorization:** Bearer Token
- **Request Headers:** `Content-Type: application/json`, `Authorization: Bearer <token>`
- **Path Parameters:** `sprintId` (UUID)
- **Query Parameters:** None
- **Request Body:** `{ "storyId": "story-1" }`
- **Validation Rules:** Valid storyId.
- **Success Response:** 200 OK
- **Error Responses:** 400 Bad Request
- **Example Request:**
```http
POST /api/v1/sprints/sprint-1/stories
Content-Type: application/json
Authorization: Bearer eyJhb...

{ "storyId": "story-1" }
```
- **Example Response:**
```json
{ "success": true, "data": { "message": "Story added to sprint" }, "metadata": { "timestamp": "2024-05-01T12:00:00Z", "requestId": "req-35" } }
```

---

## Module 8: AI Workspace API

### 1. Generate Requirement
- **Endpoint:** `POST /api/v1/projects/{projectId}/ai/generate-requirement`
- **Description:** AI generates requirements based on prompt.
- **Authorization:** Bearer Token
- **Request Headers:** `Content-Type: application/json`, `Authorization: Bearer <token>`
- **Path Parameters:** `projectId` (UUID)
- **Query Parameters:** None
- **Request Body:** `{ "prompt": "E-commerce platform", "context": {} }`
- **Validation Rules:** prompt (required).
- **Success Response:** 200 OK
- **Error Responses:** 400 Bad Request, 500 Internal Server Error
- **Example Request:**
```http
POST /api/v1/projects/proj-1/ai/generate-requirement
Content-Type: application/json
Authorization: Bearer eyJhb...

{ "prompt": "E-commerce platform", "context": {} }
```
- **Example Response:**
```json
{ "success": true, "data": { "markdown": "# Requirements\n..." }, "metadata": { "timestamp": "2024-05-01T12:00:00Z", "requestId": "req-36" } }
```

### 2. Generate HLD
- **Endpoint:** `POST /api/v1/projects/{projectId}/ai/generate-hld`
- **Description:** Generates High Level Design.
- **Authorization:** Bearer Token
- **Request Headers:** `Content-Type: application/json`, `Authorization: Bearer <token>`
- **Path Parameters:** `projectId` (UUID)
- **Query Parameters:** None
- **Request Body:** `{ "requirementsDocId": "doc-1" }`
- **Validation Rules:** requirementsDocId (required).
- **Success Response:** 200 OK
- **Error Responses:** 400 Bad Request
- **Example Request:**
```http
POST /api/v1/projects/proj-1/ai/generate-hld
Content-Type: application/json
Authorization: Bearer eyJhb...

{ "requirementsDocId": "doc-1" }
```
- **Example Response:**
```json
{ "success": true, "data": { "markdown": "# HLD\n..." }, "metadata": { "timestamp": "2024-05-01T12:00:00Z", "requestId": "req-37" } }
```

### 3. Generate LLD
- **Endpoint:** `POST /api/v1/projects/{projectId}/ai/generate-lld`
- **Description:** Generates Low Level Design.
- **Authorization:** Bearer Token
- **Request Headers:** `Content-Type: application/json`, `Authorization: Bearer <token>`
- **Path Parameters:** `projectId` (UUID)
- **Query Parameters:** None
- **Request Body:** `{ "hldDocId": "doc-2" }`
- **Validation Rules:** hldDocId (required).
- **Success Response:** 200 OK
- **Error Responses:** 400 Bad Request
- **Example Request:**
```http
POST /api/v1/projects/proj-1/ai/generate-lld
Content-Type: application/json
Authorization: Bearer eyJhb...

{ "hldDocId": "doc-2" }
```
- **Example Response:**
```json
{ "success": true, "data": { "markdown": "# LLD\n..." }, "metadata": { "timestamp": "2024-05-01T12:00:00Z", "requestId": "req-37a" } }
```

### 4. Generate Database Design
- **Endpoint:** `POST /api/v1/projects/{projectId}/ai/generate-db-design`
- **Description:** Generates Database Design.
- **Authorization:** Bearer Token
- **Request Headers:** `Content-Type: application/json`, `Authorization: Bearer <token>`
- **Path Parameters:** `projectId` (UUID)
- **Query Parameters:** None
- **Request Body:** `{ "lldDocId": "doc-3" }`
- **Validation Rules:** lldDocId (required).
- **Success Response:** 200 OK
- **Error Responses:** 400 Bad Request
- **Example Request:**
```http
POST /api/v1/projects/proj-1/ai/generate-db-design
Content-Type: application/json
Authorization: Bearer eyJhb...

{ "lldDocId": "doc-3" }
```
- **Example Response:**
```json
{ "success": true, "data": { "markdown": "# Database\n..." }, "metadata": { "timestamp": "2024-05-01T12:00:00Z", "requestId": "req-37b" } }
```

### 5. Generate API Design
- **Endpoint:** `POST /api/v1/projects/{projectId}/ai/generate-api-design`
- **Description:** Generates API Design.
- **Authorization:** Bearer Token
- **Request Headers:** `Content-Type: application/json`, `Authorization: Bearer <token>`
- **Path Parameters:** `projectId` (UUID)
- **Query Parameters:** None
- **Request Body:** `{ "lldDocId": "doc-3" }`
- **Validation Rules:** lldDocId (required).
- **Success Response:** 200 OK
- **Error Responses:** 400 Bad Request
- **Example Request:**
```http
POST /api/v1/projects/proj-1/ai/generate-api-design
Content-Type: application/json
Authorization: Bearer eyJhb...

{ "lldDocId": "doc-3" }
```
- **Example Response:**
```json
{ "success": true, "data": { "markdown": "# API\n..." }, "metadata": { "timestamp": "2024-05-01T12:00:00Z", "requestId": "req-37c" } }
```

### 6. Generate User Stories
- **Endpoint:** `POST /api/v1/projects/{projectId}/ai/generate-stories`
- **Description:** Generates User Stories.
- **Authorization:** Bearer Token
- **Request Headers:** `Content-Type: application/json`, `Authorization: Bearer <token>`
- **Path Parameters:** `projectId` (UUID)
- **Query Parameters:** None
- **Request Body:** `{ "requirementsDocId": "doc-1" }`
- **Validation Rules:** requirementsDocId (required).
- **Success Response:** 200 OK
- **Error Responses:** 400 Bad Request
- **Example Request:**
```http
POST /api/v1/projects/proj-1/ai/generate-stories
Content-Type: application/json
Authorization: Bearer eyJhb...

{ "requirementsDocId": "doc-1" }
```
- **Example Response:**
```json
{ "success": true, "data": { "markdown": "# Stories\n..." }, "metadata": { "timestamp": "2024-05-01T12:00:00Z", "requestId": "req-37d" } }
```

### 7. Generate Tasks
- **Endpoint:** `POST /api/v1/stories/{storyId}/ai/generate-tasks`
- **Description:** AI generates tasks for a story.
- **Authorization:** Bearer Token
- **Request Headers:** `Authorization: Bearer <token>`
- **Path Parameters:** `storyId` (UUID)
- **Query Parameters:** None
- **Request Body:** None
- **Validation Rules:** None
- **Success Response:** 200 OK
- **Error Responses:** 404 Not Found
- **Example Request:**
```http
POST /api/v1/stories/story-1/ai/generate-tasks
Authorization: Bearer eyJhb...
```
- **Example Response:**
```json
{ "success": true, "data": { "tasks": [ { "title": "Setup DB" } ] }, "metadata": { "timestamp": "2024-05-01T12:00:00Z", "requestId": "req-38" } }
```

### 8. Generate Acceptance Criteria
- **Endpoint:** `POST /api/v1/stories/{storyId}/ai/generate-acceptance-criteria`
- **Description:** Generates AC for a story.
- **Authorization:** Bearer Token
- **Request Headers:** `Authorization: Bearer <token>`
- **Path Parameters:** `storyId` (UUID)
- **Query Parameters:** None
- **Request Body:** None
- **Validation Rules:** None
- **Success Response:** 200 OK
- **Error Responses:** 404 Not Found
- **Example Request:**
```http
POST /api/v1/stories/story-1/ai/generate-acceptance-criteria
Authorization: Bearer eyJhb...
```
- **Example Response:**
```json
{ "success": true, "data": { "criteria": [ "User can login" ] }, "metadata": { "timestamp": "2024-05-01T12:00:00Z", "requestId": "req-39" } }
```

### 9. Generate Test Cases
- **Endpoint:** `POST /api/v1/stories/{storyId}/ai/generate-test-cases`
- **Description:** Generates Test Cases.
- **Authorization:** Bearer Token
- **Request Headers:** `Authorization: Bearer <token>`
- **Path Parameters:** `storyId` (UUID)
- **Query Parameters:** None
- **Request Body:** None
- **Validation Rules:** None
- **Success Response:** 200 OK
- **Error Responses:** 404 Not Found
- **Example Request:**
```http
POST /api/v1/stories/story-1/ai/generate-test-cases
Authorization: Bearer eyJhb...
```
- **Example Response:**
```json
{ "success": true, "data": { "testCases": [ "Test valid login" ] }, "metadata": { "timestamp": "2024-05-01T12:00:00Z", "requestId": "req-40" } }
```

### 10. Improve Prompt
- **Endpoint:** `POST /api/v1/ai/improve-prompt`
- **Description:** Improves a user's prompt.
- **Authorization:** Bearer Token
- **Request Headers:** `Content-Type: application/json`, `Authorization: Bearer <token>`
- **Path Parameters:** None
- **Query Parameters:** None
- **Request Body:** `{ "prompt": "Make an app" }`
- **Validation Rules:** prompt (required).
- **Success Response:** 200 OK
- **Error Responses:** 400 Bad Request
- **Example Request:**
```http
POST /api/v1/ai/improve-prompt
Content-Type: application/json
Authorization: Bearer eyJhb...

{ "prompt": "Make an app" }
```
- **Example Response:**
```json
{ "success": true, "data": { "improvedPrompt": "Design a modern web application..." }, "metadata": { "timestamp": "2024-05-01T12:00:00Z", "requestId": "req-41" } }
```

### 11. AI Chat
- **Endpoint:** `POST /api/v1/projects/{projectId}/ai/chat`
- **Description:** Sends message to AI context.
- **Authorization:** Bearer Token
- **Request Headers:** `Content-Type: application/json`, `Authorization: Bearer <token>`
- **Path Parameters:** `projectId` (UUID)
- **Query Parameters:** None
- **Request Body:** `{ "message": "How do I?", "conversationId": "uuid" }`
- **Validation Rules:** message (required).
- **Success Response:** 200 OK
- **Error Responses:** 400 Bad Request
- **Example Request:**
```http
POST /api/v1/projects/proj-1/ai/chat
Content-Type: application/json
Authorization: Bearer eyJhb...

{ "message": "How do I?", "conversationId": "uuid" }
```
- **Example Response:**
```json
{ "success": true, "data": { "reply": "You can do this by..." }, "metadata": { "timestamp": "2024-05-01T12:00:00Z", "requestId": "req-42" } }
```

### 12. Conversation History
- **Endpoint:** `GET /api/v1/projects/{projectId}/ai/conversations`
- **Description:** Lists conversation histories.
- **Authorization:** Bearer Token
- **Request Headers:** `Authorization: Bearer <token>`
- **Path Parameters:** `projectId` (UUID)
- **Query Parameters:** None
- **Request Body:** None
- **Validation Rules:** Valid projectId.
- **Success Response:** 200 OK
- **Error Responses:** 404 Not Found
- **Example Request:**
```http
GET /api/v1/projects/proj-1/ai/conversations
Authorization: Bearer eyJhb...
```
- **Example Response:**
```json
{ "success": true, "data": [ { "conversationId": "conv-1", "title": "Setup chat" } ], "metadata": { "timestamp": "2024-05-01T12:00:00Z", "requestId": "req-43" } }
```

---

## Module 9: GitHub Integration API

### 1. Connect Repository
- **Endpoint:** `POST /api/v1/projects/{projectId}/github/connect`
- **Description:** Links GitHub repo to project.
- **Authorization:** Bearer Token
- **Request Headers:** `Content-Type: application/json`, `Authorization: Bearer <token>`
- **Path Parameters:** `projectId` (UUID)
- **Query Parameters:** None
- **Request Body:** `{ "repositoryUrl": "https://github.com/org/repo", "installationId": "1234" }`
- **Validation Rules:** URL and installationId required.
- **Success Response:** 200 OK
- **Error Responses:** 400 Bad Request
- **Example Request:**
```http
POST /api/v1/projects/proj-1/github/connect
Content-Type: application/json
Authorization: Bearer eyJhb...

{ "repositoryUrl": "https://github.com/org/repo", "installationId": "1234" }
```
- **Example Response:**
```json
{ "success": true, "data": { "message": "Repository connected" }, "metadata": { "timestamp": "2024-05-01T12:00:00Z", "requestId": "req-44" } }
```

### 2. Disconnect Repository
- **Endpoint:** `POST /api/v1/projects/{projectId}/github/disconnect`
- **Description:** Unlinks repository.
- **Authorization:** Bearer Token
- **Request Headers:** `Authorization: Bearer <token>`
- **Path Parameters:** `projectId` (UUID)
- **Query Parameters:** None
- **Request Body:** None
- **Validation Rules:** None
- **Success Response:** 200 OK
- **Error Responses:** 404 Not Found
- **Example Request:**
```http
POST /api/v1/projects/proj-1/github/disconnect
Authorization: Bearer eyJhb...
```
- **Example Response:**
```json
{ "success": true, "data": { "message": "Repository disconnected" }, "metadata": { "timestamp": "2024-05-01T12:00:00Z", "requestId": "req-45" } }
```

### 3. List Repositories
- **Endpoint:** `GET /api/v1/workspaces/{workspaceId}/github/repositories`
- **Description:** Lists available GitHub repos.
- **Authorization:** Bearer Token
- **Request Headers:** `Authorization: Bearer <token>`
- **Path Parameters:** `workspaceId` (UUID)
- **Query Parameters:** None
- **Request Body:** None
- **Validation Rules:** Valid workspaceId.
- **Success Response:** 200 OK
- **Error Responses:** 403 Forbidden
- **Example Request:**
```http
GET /api/v1/workspaces/ws-1/github/repositories
Authorization: Bearer eyJhb...
```
- **Example Response:**
```json
{ "success": true, "data": [ { "name": "repo1", "url": "..." } ], "metadata": { "timestamp": "2024-05-01T12:00:00Z", "requestId": "req-46" } }
```

### 4. Repository Details
- **Endpoint:** `GET /api/v1/projects/{projectId}/github/repository`
- **Description:** Retrieves repo details.
- **Authorization:** Bearer Token
- **Request Headers:** `Authorization: Bearer <token>`
- **Path Parameters:** `projectId` (UUID)
- **Query Parameters:** None
- **Request Body:** None
- **Validation Rules:** Valid projectId.
- **Success Response:** 200 OK
- **Error Responses:** 404 Not Found
- **Example Request:**
```http
GET /api/v1/projects/proj-1/github/repository
Authorization: Bearer eyJhb...
```
- **Example Response:**
```json
{ "success": true, "data": { "name": "repo", "url": "..." }, "metadata": { "timestamp": "2024-05-01T12:00:00Z", "requestId": "req-47" } }
```

### 5. Branches
- **Endpoint:** `GET /api/v1/projects/{projectId}/github/branches`
- **Description:** Lists repo branches.
- **Authorization:** Bearer Token
- **Request Headers:** `Authorization: Bearer <token>`
- **Path Parameters:** `projectId` (UUID)
- **Query Parameters:** None
- **Request Body:** None
- **Validation Rules:** Valid projectId.
- **Success Response:** 200 OK
- **Error Responses:** 404 Not Found
- **Example Request:**
```http
GET /api/v1/projects/proj-1/github/branches
Authorization: Bearer eyJhb...
```
- **Example Response:**
```json
{ "success": true, "data": [ { "name": "main" } ], "metadata": { "timestamp": "2024-05-01T12:00:00Z", "requestId": "req-48" } }
```

### 6. Commits
- **Endpoint:** `GET /api/v1/projects/{projectId}/github/commits`
- **Description:** Lists commits.
- **Authorization:** Bearer Token
- **Request Headers:** `Authorization: Bearer <token>`
- **Path Parameters:** `projectId` (UUID)
- **Query Parameters:** None
- **Request Body:** None
- **Validation Rules:** Valid projectId.
- **Success Response:** 200 OK
- **Error Responses:** 404 Not Found
- **Example Request:**
```http
GET /api/v1/projects/proj-1/github/commits
Authorization: Bearer eyJhb...
```
- **Example Response:**
```json
{ "success": true, "data": [ { "sha": "123", "message": "fix" } ], "metadata": { "timestamp": "2024-05-01T12:00:00Z", "requestId": "req-49" } }
```

### 7. Pull Requests
- **Endpoint:** `GET /api/v1/projects/{projectId}/github/pull-requests`
- **Description:** Lists PRs.
- **Authorization:** Bearer Token
- **Request Headers:** `Authorization: Bearer <token>`
- **Path Parameters:** `projectId` (UUID)
- **Query Parameters:** None
- **Request Body:** None
- **Validation Rules:** Valid projectId.
- **Success Response:** 200 OK
- **Error Responses:** 404 Not Found
- **Example Request:**
```http
GET /api/v1/projects/proj-1/github/pull-requests
Authorization: Bearer eyJhb...
```
- **Example Response:**
```json
{ "success": true, "data": [ { "number": 1, "title": "Feature" } ], "metadata": { "timestamp": "2024-05-01T12:00:00Z", "requestId": "req-50" } }
```

### 8. AI Code Review
- **Endpoint:** `POST /api/v1/projects/{projectId}/github/pull-requests/{prNumber}/ai-review`
- **Description:** Triggers AI PR review.
- **Authorization:** Bearer Token
- **Request Headers:** `Authorization: Bearer <token>`
- **Path Parameters:** `projectId` (UUID), `prNumber` (Int)
- **Query Parameters:** None
- **Request Body:** None
- **Validation Rules:** Valid IDs.
- **Success Response:** 202 Accepted
- **Error Responses:** 404 Not Found
- **Example Request:**
```http
POST /api/v1/projects/proj-1/github/pull-requests/1/ai-review
Authorization: Bearer eyJhb...
```
- **Example Response:**
```json
{ "success": true, "data": { "message": "Review started" }, "metadata": { "timestamp": "2024-05-01T12:00:00Z", "requestId": "req-51" } }
```

### 9. Webhook
- **Endpoint:** `POST /api/v1/github/webhook`
- **Description:** GitHub webhook receiver.
- **Authorization:** Custom GitHub Signature
- **Request Headers:** `X-Hub-Signature-256`
- **Path Parameters:** None
- **Query Parameters:** None
- **Request Body:** GitHub Payload
- **Validation Rules:** Signature must match secret.
- **Success Response:** 200 OK
- **Error Responses:** 401 Unauthorized
- **Example Request:**
```http
POST /api/v1/github/webhook
X-Hub-Signature-256: sha256=...

{ "action": "opened", "pull_request": {} }
```
- **Example Response:**
```json
{ "success": true, "data": { "message": "Webhook processed" }, "metadata": { "timestamp": "2024-05-01T12:00:00Z", "requestId": "req-52" } }
```

### 10. Sync
- **Endpoint:** `POST /api/v1/projects/{projectId}/github/sync`
- **Description:** Manually syncs repo data.
- **Authorization:** Bearer Token
- **Request Headers:** `Authorization: Bearer <token>`
- **Path Parameters:** `projectId` (UUID)
- **Query Parameters:** None
- **Request Body:** None
- **Validation Rules:** Valid projectId.
- **Success Response:** 202 Accepted
- **Error Responses:** 404 Not Found
- **Example Request:**
```http
POST /api/v1/projects/proj-1/github/sync
Authorization: Bearer eyJhb...
```
- **Example Response:**
```json
{ "success": true, "data": { "message": "Sync queued" }, "metadata": { "timestamp": "2024-05-01T12:00:00Z", "requestId": "req-53" } }
```

---

## Module 10: Notification API

### 1. Get Notifications
- **Endpoint:** `GET /api/v1/users/me/notifications`
- **Description:** Gets user's notifications.
- **Authorization:** Bearer Token
- **Request Headers:** `Authorization: Bearer <token>`
- **Path Parameters:** None
- **Query Parameters:** `page`, `limit`
- **Request Body:** None
- **Validation Rules:** None
- **Success Response:** 200 OK
- **Error Responses:** 401 Unauthorized
- **Example Request:**
```http
GET /api/v1/users/me/notifications
Authorization: Bearer eyJhb...
```
- **Example Response:**
```json
{ "success": true, "data": [ { "id": "n-1", "message": "Assigned task" } ], "metadata": { "timestamp": "2024-05-01T12:00:00Z", "requestId": "req-54" } }
```

### 2. Mark Read
- **Endpoint:** `PATCH /api/v1/notifications/{notificationId}/read`
- **Description:** Marks one notification read.
- **Authorization:** Bearer Token
- **Request Headers:** `Authorization: Bearer <token>`
- **Path Parameters:** `notificationId` (UUID)
- **Query Parameters:** None
- **Request Body:** None
- **Validation Rules:** Valid notificationId.
- **Success Response:** 200 OK
- **Error Responses:** 404 Not Found
- **Example Request:**
```http
PATCH /api/v1/notifications/n-1/read
Authorization: Bearer eyJhb...
```
- **Example Response:**
```json
{ "success": true, "data": { "message": "Marked read" }, "metadata": { "timestamp": "2024-05-01T12:00:00Z", "requestId": "req-55" } }
```

### 3. Mark All Read
- **Endpoint:** `POST /api/v1/users/me/notifications/read-all`
- **Description:** Marks all notifications read.
- **Authorization:** Bearer Token
- **Request Headers:** `Authorization: Bearer <token>`
- **Path Parameters:** None
- **Query Parameters:** None
- **Request Body:** None
- **Validation Rules:** None
- **Success Response:** 200 OK
- **Error Responses:** 401 Unauthorized
- **Example Request:**
```http
POST /api/v1/users/me/notifications/read-all
Authorization: Bearer eyJhb...
```
- **Example Response:**
```json
{ "success": true, "data": { "message": "All marked read" }, "metadata": { "timestamp": "2024-05-01T12:00:00Z", "requestId": "req-56" } }
```

### 4. Delete Notification
- **Endpoint:** `DELETE /api/v1/notifications/{notificationId}`
- **Description:** Deletes a notification.
- **Authorization:** Bearer Token
- **Request Headers:** `Authorization: Bearer <token>`
- **Path Parameters:** `notificationId` (UUID)
- **Query Parameters:** None
- **Request Body:** None
- **Validation Rules:** Valid notificationId.
- **Success Response:** 204 No Content
- **Error Responses:** 404 Not Found
- **Example Request:**
```http
DELETE /api/v1/notifications/n-1
Authorization: Bearer eyJhb...
```
- **Example Response:** *(Empty Body)*

### 5. Notification Preferences
- **Endpoint:** `GET /api/v1/users/me/notification-preferences`
- **Description:** Gets/Updates notification preferences.
- **Authorization:** Bearer Token
- **Request Headers:** `Authorization: Bearer <token>`
- **Path Parameters:** None
- **Query Parameters:** None
- **Request Body:** None
- **Validation Rules:** None
- **Success Response:** 200 OK
- **Error Responses:** 401 Unauthorized
- **Example Request:**
```http
GET /api/v1/users/me/notification-preferences
Authorization: Bearer eyJhb...
```
- **Example Response:**
```json
{ "success": true, "data": { "email": true, "push": false }, "metadata": { "timestamp": "2024-05-01T12:00:00Z", "requestId": "req-57" } }
```

---

## Module 11: Activity API

### 1. Workspace Activity
- **Endpoint:** `GET /api/v1/workspaces/{workspaceId}/activities`
- **Description:** Retrieves workspace activity feed.
- **Authorization:** Bearer Token
- **Request Headers:** `Authorization: Bearer <token>`
- **Path Parameters:** `workspaceId` (UUID)
- **Query Parameters:** `page`, `limit`, `type`, `userId`
- **Request Body:** None
- **Validation Rules:** Valid workspaceId.
- **Success Response:** 200 OK
- **Error Responses:** 403 Forbidden
- **Example Request:**
```http
GET /api/v1/workspaces/ws-1/activities
Authorization: Bearer eyJhb...
```
- **Example Response:**
```json
{ "success": true, "data": [ { "action": "CREATED_PROJECT", "userId": "uuid-1" } ], "metadata": { "timestamp": "2024-05-01T12:00:00Z", "requestId": "req-58" } }
```

### 2. Project Activity
- **Endpoint:** `GET /api/v1/projects/{projectId}/activities`
- **Description:** Retrieves project activity feed.
- **Authorization:** Bearer Token
- **Request Headers:** `Authorization: Bearer <token>`
- **Path Parameters:** `projectId` (UUID)
- **Query Parameters:** `page`, `limit`
- **Request Body:** None
- **Validation Rules:** Valid projectId.
- **Success Response:** 200 OK
- **Error Responses:** 404 Not Found
- **Example Request:**
```http
GET /api/v1/projects/proj-1/activities
Authorization: Bearer eyJhb...
```
- **Example Response:**
```json
{ "success": true, "data": [ { "action": "CREATED_STORY", "userId": "uuid-1" } ], "metadata": { "timestamp": "2024-05-01T12:00:00Z", "requestId": "req-59" } }
```

### 3. Audit Logs
- **Endpoint:** `GET /api/v1/workspaces/{workspaceId}/audit-logs`
- **Description:** Retrieves secure audit logs.
- **Authorization:** Bearer Token (Owner, Admin)
- **Request Headers:** `Authorization: Bearer <token>`
- **Path Parameters:** `workspaceId` (UUID)
- **Query Parameters:** `page`, `limit`
- **Request Body:** None
- **Validation Rules:** Valid workspaceId.
- **Success Response:** 200 OK
- **Error Responses:** 403 Forbidden
- **Example Request:**
```http
GET /api/v1/workspaces/ws-1/audit-logs
Authorization: Bearer eyJhb...
```
- **Example Response:**
```json
{ "success": true, "data": [ { "event": "ROLE_CHANGED", "actor": "uuid-1", "target": "uuid-2" } ], "metadata": { "timestamp": "2024-05-01T12:00:00Z", "requestId": "req-60" } }
```
