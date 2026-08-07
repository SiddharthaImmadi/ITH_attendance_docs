# Backend Architecture

> This document describes the internal architecture of the InnoTech Hub Attendance backend. It defines how the FastAPI application is organized, how requests flow through the system, how business logic is implemented, and how backend components interact to provide a scalable, secure, and maintainable attendance management platform.

---

# 1. Overview

The backend is implemented using a layered architecture that separates HTTP handling, business logic, data access, and infrastructure concerns.

The backend serves as the single source of truth for all application data and business decisions. Clients are responsible only for collecting user input and displaying responses, while all validation, authorization, workflow execution, and data persistence are performed by the backend.

The architecture is designed to provide:

- clear separation of responsibilities;
- maintainable and testable business logic;
- secure authentication and authorization;
- consistent API behavior;
- transactional data integrity;
- scalable service-oriented design;
- asynchronous request processing.

The backend is organized around business domains rather than user interfaces. Each domain encapsulates its own routes, services, data models, and validation rules while collaborating through clearly defined service boundaries.

---

# 2. Architecture Goals

The backend architecture follows several core design goals.

## 2.1 Separation of Concerns

Each architectural layer has a single responsibility.

- Route handlers process HTTP requests.
- Services implement business logic.
- Repositories manage database access.
- Models represent persistent data.
- Middleware provides cross-cutting functionality.

Business rules must never be implemented inside route handlers.

---

## 2.2 Backend as the Source of Truth

All business decisions are performed by the backend.

Examples include:

- authentication;
- authorization;
- attendance validation;
- presence monitoring;
- boundary evaluation;
- emergency ticket processing;
- volunteer participation eligibility.

Client applications must never bypass backend validation.

---

## 2.3 Modular Design

The application is organized into independent business modules.

Examples include:

- Authentication
- Users
- Events
- Attendance
- Presence Monitoring
- Emergency Tickets
- Activities
- Notifications
- Reports

Each module is responsible for its own business operations while interacting with other modules through service interfaces.

---

## 2.4 Scalability

The architecture is designed to support future expansion without significant restructuring.

New business modules should integrate using existing architectural patterns rather than introducing new application structures.

---

## 2.5 Maintainability

The backend emphasizes readable, predictable, and well-organized code.

Business logic should remain centralized within service classes to reduce duplication and simplify future enhancements.

---

## 2.6 Consistency

All modules follow consistent architectural conventions for:

- routing;
- validation;
- dependency injection;
- error handling;
- database access;
- logging;
- response formatting.

Consistency improves maintainability and reduces development complexity.

---

## 2.7 Testability

Application components should be independently testable.

Business logic should be isolated from HTTP concerns to enable efficient unit and integration testing.

---

# 3. Technology Stack

The backend is implemented using modern Python technologies.

| Component | Technology |
|-----------|------------|
| Programming Language | Python 3.13+ |
| Web Framework | FastAPI |
| ASGI Server | Uvicorn |
| Database | PostgreSQL |
| ORM | SQLAlchemy 2.x (Async) |
| Database Migrations | Alembic |
| Validation | Pydantic |
| Authentication | JWT |
| Password Hashing | bcrypt |
| API Documentation | OpenAPI / Swagger |
| Dependency Management | pip + virtual environment |

The technology stack prioritizes asynchronous execution, strong typing, automatic API documentation, and long-term maintainability.

---

# 4. Project Structure

The backend is organized into domain-oriented modules.

```text
backend/

├── alembic/
│   └── database migrations
│
app/

├── api/
│   ├── auth.py
│   ├── users.py
│   ├── events.py
│   ├── attendance.py
│   ├── presence.py
│   ├── emergency_tickets.py
│   ├── activities.py
│   ├── notifications.py
│   └── reports.py
│
├── core/
│
├── models/
│
├── schemas/
│
├── services/
│
├── repositories/
│
├── middleware/
│
├── utils/
│
└── main.py
│
├── media/
│
├── logs/
│
└── tests/
```

### activities.py

Responsible for:

- activity creation;
- activity assignment;
- evidence management;
- assignment submission;
- review workflow;
- activity template management.

Each directory has a clearly defined responsibility.

### api/

Contains FastAPI route handlers responsible for HTTP communication.

### core/

Contains application configuration, security, database configuration, and shared infrastructure.

### models/

Defines SQLAlchemy database models.

### schemas/

Defines request and response models using Pydantic.

### services/

Contains business logic.

Services coordinate application workflows while remaining independent of HTTP implementation details.

### repositories/

Provides database access and persistence operations.

Repositories abstract ORM interactions from business services.

### middleware/

Contains reusable middleware components such as logging and request processing.

### utils/

Contains reusable helper utilities shared across the application.

### media/

Stores uploaded files such as attendance evidence when applicable.

### logs/

Contains application log files.

### tests/

Contains unit tests, integration tests, and API tests.

---

# 5. Layered Architecture

The backend follows a layered architecture that separates responsibilities between HTTP communication, business logic, persistence, and storage.

```text
HTTP Request

↓

Middleware

↓

Route Handler

↓

Service Layer

↓

Repository Layer

↓

ORM Layer

↓

PostgreSQL Database

↓

HTTP Response
```

Each layer communicates only with the layer directly beneath it.

This separation provides:

- maintainability;
- independent testing;
- reusable business logic;
- reduced coupling;
- improved scalability.

Business rules are implemented exclusively within the Service Layer, while Route Handlers remain responsible only for request processing and response generation.

# 6. Request Lifecycle

Every client request follows a consistent processing pipeline from the moment it reaches the backend until a response is returned.

Each architectural layer performs a specific responsibility before passing control to the next layer.

```
Client

↓

HTTP Request

↓

Middleware

↓

Route Handler

↓

Schema Validation

↓

Authentication & Authorization

↓

Service Layer

↓

Repository Layer

↓

Database

↓

Repository Layer

↓

Service Layer

↓

Response Model

↓

HTTP Response
```

This standardized request lifecycle ensures:

- consistent request processing;
- centralized validation;
- secure authentication;
- reusable business logic;
- predictable error handling;
- maintainable application architecture.

---

## 6.1 Client Request

A request begins when a client application sends an HTTP request to one of the backend API endpoints.

The request may include:

- URL parameters;
- query parameters;
- request body;
- uploaded files;
- authentication token.

The backend treats all client input as untrusted until validation has completed.

---

## 6.2 Middleware Processing

Before reaching the application logic, the request passes through the configured middleware pipeline.

Middleware performs cross-cutting responsibilities such as:

- request logging;
- CORS handling;
- request identification;
- exception handling;
- performance monitoring.

Middleware does not implement business logic.

---

## 6.3 Route Resolution

FastAPI matches the incoming request to the appropriate route handler.

The route handler is responsible for:

- receiving validated request data;
- resolving dependencies;
- invoking the appropriate service;
- returning the service response.

Route handlers should remain lightweight and contain minimal application logic.

---

## 6.4 Request Validation

Before business logic executes, FastAPI validates the request using Pydantic schemas.

Validation includes:

- required fields;
- data types;
- field constraints;
- request structure.

Requests that fail schema validation are rejected before reaching the service layer.

Business rule validation is performed separately by application services.

---

## 6.5 Authentication & Authorization

Protected endpoints require authentication before business operations may proceed.

Authentication responsibilities include:

- validating the JWT;
- identifying the authenticated user;
- verifying token validity.

Authorization determines whether the authenticated user has permission to perform the requested operation.

Only authorized requests continue to the business layer.

---

## 6.6 Service Execution

After validation succeeds, the request is delegated to the appropriate service.

The service layer is responsible for:

- business rule validation;
- workflow execution;
- coordination between services;
- repository interaction;
- transaction management.

Business services remain independent of HTTP implementation details.

---

## 6.7 Repository Operations

Services interact with repositories to access persistent data.

Repositories are responsible for:

- querying the database;
- creating records;
- updating records;
- deleting records where permitted.

Repositories should not implement business rules.

---

## 6.8 Database Interaction

Repositories communicate with PostgreSQL using SQLAlchemy's asynchronous ORM.

Database responsibilities include:

- persistent storage;
- transaction support;
- constraint enforcement;
- relationship management.

Database constraints complement—but do not replace—business validation performed by the service layer.

---

## 6.9 Response Generation

After the business operation completes successfully:

- repositories return data to services;
- services prepare the business result;
- route handlers serialize the response using Pydantic models.

The client receives a standardized API response.

---

## 6.10 Exception Handling

Errors may occur at any stage of request processing.

Exceptions are handled centrally to ensure:

- consistent error responses;
- appropriate HTTP status codes;
- standardized error structures;
- secure error reporting.

Internal implementation details should never be exposed to clients.

---

## 6.11 Transaction Completion

For operations that modify persistent data, database transactions are completed before the response is returned.

Successful operations commit the transaction.

Failed operations roll back all pending database changes to preserve data integrity.

---

## 6.12 Response Delivery

Once processing completes, the backend returns an HTTP response to the client.

Responses contain:

- operation status;
- response data when applicable;
- standardized error information when requests fail.

After the response is delivered, request-specific resources are released and the request lifecycle concludes.



# 7. Route Layer

The Route Layer provides the public REST API of the backend.

Route handlers act as the entry point for all client requests. Their responsibility is to receive HTTP requests, validate input, invoke the appropriate business service, and return standardized responses.

Route handlers must remain lightweight and must never contain business logic.

---

## 7.1 Responsibilities

The Route Layer is responsible for:

- receiving HTTP requests;
- validating request schemas;
- resolving dependencies;
- authenticating protected endpoints;
- authorizing the current user;
- invoking the appropriate service;
- returning standardized API responses.

Route handlers should never implement business workflows.

---

## 7.2 Router Organization

Routes are organized by business domain.

```text
app/api/

├── auth.py
├── users.py
├── events.py
├── attendance.py
├── presence.py
├── emergency_tickets.py
├── notifications.py
└── reports.py
```

Each router exposes endpoints for a single business domain.

---

## 7.3 Router Responsibilities

### auth.py

Responsible for:

- user login;
- current authenticated user;
- token validation.

---

### users.py

Responsible for:

- user management;
- administrator user operations;
- user profile retrieval.

---

### events.py

Responsible for:

- event creation;
- event updates;
- event activation;
- event completion;
- event retrieval.

---

### attendance.py

Responsible for:

- attendance check-in;
- attendance status;
- attendance check-out;
- attendance history.

---

### presence.py

Responsible for:

- location updates;
- presence status;
- current presence information.

---

### emergency_tickets.py

Responsible for:

- ticket submission;
- ticket retrieval;
- administrator review;
- approval and rejection.

---

### notifications.py

Responsible for:

- retrieving notifications;
- marking notifications as read.

---

### reports.py

Responsible for:

- attendance reports;
- event reports;
- administrative exports.

---

## 7.4 Request Processing

A route handler should follow the same processing sequence.

```
Receive Request

↓

Validate Schema

↓

Resolve Dependencies

↓

Authenticate User

↓

Authorize User

↓

Call Service

↓

Return Response
```

Each step has a clearly defined responsibility.

---

## 7.5 Validation

Request validation is performed using Pydantic request models.

Validation includes:

- required fields;
- data types;
- field constraints;
- request structure.

Business rule validation remains the responsibility of the Service Layer.

---

## 7.6 Dependency Injection

FastAPI dependency injection is used to obtain shared application resources.

Common dependencies include:

- authenticated user;
- database session;
- application configuration;
- reusable services.

Dependency injection promotes loose coupling and improves testability.

---

## 7.7 Response Handling

Route handlers return standardized response models.

Response models provide:

- consistent API contracts;
- automatic serialization;
- automatic OpenAPI documentation;
- predictable client behavior.

Route handlers should never manually construct business responses when a response model is available.

---

## 7.8 Route Design Principles

Every route handler should:

- remain thin;
- delegate business logic to services;
- avoid direct database access;
- avoid transaction management;
- avoid duplicate validation.

Following these principles ensures consistent application behavior and improves long-term maintainability.


# 8. Service Layer

The Service Layer is the core of the backend architecture.

It contains all business logic, coordinates application workflows, enforces business rules, and orchestrates interactions between repositories and other services.

Services remain independent of HTTP communication and database implementation details, making them reusable, testable, and maintainable.

---

## 8.1 Responsibilities

Services are responsible for:

- implementing business rules;
- validating business operations;
- coordinating application workflows;
- interacting with repositories;
- managing database transactions;
- invoking supporting services;
- generating business events.

Services must never depend on HTTP request or response objects.

---

## 8.2 Service Organization

Business logic is organized into independent domain services.

```text
app/services/

app/services/

├── auth_service.py
├── user_service.py
├── event_service.py
├── attendance_service.py
├── presence_service.py
├── boundary_service.py
├── emergency_ticket_service.py
├── volunteer_block_service.py
├── activity_service.py
├── activity_assignment_service.py
├── activity_review_service.py
├── activity_template_service.py
├── notification_service.py
├── activity_history_service.py
├── audit_log_service.py
└── report_service.py
```

Each service owns a single business domain and exposes well-defined operations for that domain.

---

## 8.3 Service Responsibilities

Each service has a clearly defined responsibility.

### Authentication Service

Responsible for:

- user authentication;
- password verification;
- JWT generation;
- JWT validation;
- current user retrieval.

---

### User Service

Responsible for:

- user management;
- user retrieval;
- user updates;
- role validation.

---

### Event Service

Responsible for:

- event creation;
- event updates;
- event lifecycle management;
- event activation;
- event completion.

---

### Attendance Service

Responsible for:

- attendance check-in;
- attendance validation;
- attendance status transitions;
- attendance check-out;
- attendance lifecycle management.

---

### Presence Service

Responsible for:

- processing location updates;
- maintaining presence state;
- recording presence history;
- coordinating boundary evaluation.

---

### Boundary Service

Responsible for:

- validating geographic boundaries;
- evaluating member locations;
- supporting GeoJSON boundaries;
- supporting captured GPS point validation.

Boundary calculations remain isolated within this service to encourage reuse and simplify future enhancements.

---

### Emergency Ticket Service

Responsible for:

- emergency ticket submission;
- administrator review;
- approval processing;
- rejection processing;
- ticket status management.

---

### Volunteer Block Service

Responsible for:

- volunteer participation restrictions;
- eligibility validation;
- block creation;
- block removal;
- participation enforcement.


### Activity Service

Responsible for:

- activity creation;
- activity updates;
- activity publication;
- activity cancellation;
- activity archival.

---

### Activity Assignment Service

Responsible for:

- assigning activities to volunteers;
- validating assignment conflicts;
- maintaining assignment status;
- tracking assignment lifecycle.
- starting assigned activities;
- submitting assignments for review;
- validating assignment completion requirements;
---

### Activity Review Service

Responsible for:

- reviewing submitted activities;
- requesting changes;
- verifying completed activities;
- preserving review history.

---

### Activity Template Service

Responsible for:

- template creation;
- template updates;
- template retrieval;
- generating activities from templates.
---

### Notification Service

Responsible for:

- notification generation;
- notification retrieval;
- notification status updates.

Notifications are generated automatically as part of business workflows.

---

### Activity History Service

Responsible for:

- recording user activity;
- maintaining chronological timelines;
- generating activity records for completed business operations.

---

### Audit Log Service

Responsible for:

- recording administrative actions;
- recording security-sensitive operations;
- maintaining immutable audit history.

---

### Report Service

Responsible for:

- attendance reporting;
- administrative summaries;
- data exports;
- statistical reporting.

Report generation should not contain business validation logic.

---

## 8.4 Service Collaboration

Business operations frequently require multiple services.

Services collaborate through clearly defined interfaces while maintaining ownership of their respective domains.

Example workflow:

```text
Activity Service

↓

Activity Assignment Service

↓

Activity Review Service

↓

Notification Service

↓

Activity History Service

↓

Audit Log Service
```

Each service performs its own responsibility before returning control to the initiating service.

---

## 8.5 Business Rule Enforcement

All business rules are enforced within the Service Layer.

Examples include:

- attendance eligibility;
- event state validation;
- boundary validation;
- emergency ticket eligibility;
- volunteer participation restrictions;
- authorization checks.

Business rules must never be duplicated across multiple architectural layers.

---

## 8.6 Transaction Coordination

Services coordinate database transactions involving multiple repositories.

Examples include:

- attendance check-in;
- attendance check-out;
- emergency ticket approval;
- volunteer block creation.

Related database updates should be committed as a single transaction to preserve consistency.

---

## 8.7 Repository Interaction

Services interact with repositories rather than communicating directly with the database.

Repositories provide persistence operations while services remain responsible for business decisions.

This separation reduces coupling between business logic and data storage.

---

## 8.8 Error Handling

Services detect and report business rule violations.

Examples include:

- attendance already exists;
- event not active;
- boundary validation failed;
- emergency ticket not found;
- volunteer currently blocked.

Services should raise application-specific exceptions that are translated into standardized API responses by the application's exception handling mechanism.

---

## 8.9 Service Design Principles

Every service should:

- have a single business responsibility;
- expose clear public methods;
- remain independent of HTTP concerns;
- remain independent of presentation logic;
- use dependency injection where appropriate;
- avoid unnecessary coupling with other services.

Well-defined service boundaries improve maintainability and simplify testing.

---

## 8.10 Testability

Services should be independently testable without requiring HTTP requests.

Dependencies should be injectable, allowing repositories and external components to be replaced with test doubles during automated testing.

Isolated services enable reliable unit testing while preserving consistent business behavior throughout the application.

# 9. Repository Layer

The Repository Layer is responsible for all interactions with the database.

Repositories provide a clean abstraction between the Service Layer and the persistence layer, allowing business logic to remain independent of database implementation details.

Repositories perform data access operations but must never implement business rules.

---

## 9.1 Responsibilities

Repositories are responsible for:

- retrieving persistent data;
- creating new records;
- updating existing records;
- deleting records where permitted;
- executing optimized database queries;
- managing ORM entities.

Repositories should never make business decisions.

---

## 9.2 Repository Organization

Repositories are organized by business domain.

```text
app/repositories/

app/repositories/

├── user_repository.py
├── event_repository.py
├── attendance_repository.py
├── presence_repository.py
├── emergency_ticket_repository.py
├── volunteer_block_repository.py
├── activity_repository.py
├── activity_assignment_repository.py
├── activity_evidence_repository.py
├── activity_review_repository.py
├── activity_template_repository.py
├── notification_repository.py
├── activity_history_repository.py
├── audit_log_repository.py
└── report_repository.py
```

Each repository manages persistence for a single business entity.

---

## 9.3 Repository Responsibilities

### User Repository

Responsible for:

- retrieving users;
- creating users;
- updating users;
- role lookups.

---

### Event Repository

Responsible for:

- event persistence;
- event retrieval;
- event updates;
- event lifecycle storage.

---

### Attendance Repository

Responsible for:

- attendance records;
- attendance lookups;
- attendance updates;
- attendance history retrieval.

---

### Presence Repository

Responsible for:

- presence logs;
- presence history;
- latest presence state.

---

### Emergency Ticket Repository

Responsible for:

- ticket persistence;
- ticket retrieval;
- ticket status updates.

---

### Volunteer Block Repository

Responsible for:

- active volunteer blocks;
- historical volunteer blocks;
- eligibility lookups.


### Activity Repository

Responsible for:

- creating activities;
- updating activity details;
- publishing activities;
- cancelling activities;
- archiving activities;
- retrieving activity lists.

---

### Activity Assignment Repository

Responsible for:

- creating assignments;
- retrieving volunteer assignments;
- updating assignment status;
- validating duplicate assignments.

---

### Activity Evidence Repository

Responsible for:

- storing evidence metadata;
- retrieving evidence;
- deleting evidence before submission.

---

### Activity Review Repository

Responsible for:

- storing review decisions;
- retrieving review history;
- retrieving pending reviews.

---

### Activity Template Repository

Responsible for:

- storing templates;
- storing template items;
- retrieving templates;
- applying templates.

---

### Notification Repository

Responsible for:

- notification persistence;
- unread notification retrieval;
- notification status updates.

---

### Activity History Repository

Responsible for:

- activity record persistence;
- chronological activity retrieval.

---

### Audit Log Repository

Responsible for:

- audit log persistence;
- administrative audit retrieval.

---

### Report Repository

Responsible for optimized reporting queries.

Reporting repositories may execute read-only queries specifically designed for reporting performance.

---

## 9.4 Repository Design Principles

Repositories should:

- expose simple persistence methods;
- avoid business validation;
- return domain entities;
- remain independent of HTTP concerns;
- use SQLAlchemy consistently.

Business workflows remain the responsibility of services.

---

## 9.5 ORM Interaction

Repositories interact with PostgreSQL using SQLAlchemy's asynchronous ORM.

The ORM provides:

- entity mapping;
- relationship management;
- query generation;
- transaction support.

Repositories should avoid exposing ORM implementation details to higher layers.

---

## 9.6 Query Optimization

Repositories should implement efficient database queries.

Examples include:

- indexed lookups;
- eager loading where appropriate;
- pagination;
- selective column retrieval;
- filtering at the database level.

Repositories should avoid unnecessary database round trips.

---

## 9.7 Transaction Participation

Repositories participate in transactions coordinated by the Service Layer.

Repositories should never independently commit or roll back transactions unless explicitly designed for isolated operations.

Transaction boundaries remain the responsibility of services.

---

## 9.8 Error Handling

Database exceptions should be translated into repository-level exceptions where appropriate.

Repository implementations should avoid exposing raw database errors to higher architectural layers.

Business error interpretation remains the responsibility of services.

---

## 9.9 Testability

Repositories should be independently testable.

Repository tests should verify:

- correct query execution;
- entity persistence;
- relationship handling;
- transaction participation.

Repository tests should not duplicate business rule tests.


# 10. Authentication Architecture

Authentication provides secure access to protected backend resources.

The authentication architecture verifies user identity, establishes trust between the client and the backend, and provides authorization information for protected operations.

Authentication is stateless and based on JSON Web Tokens (JWT).

---

## 10.1 Authentication Flow

Every authenticated request follows the same processing sequence.

```text
Client Login Request

↓

Authentication Route

↓

Authentication Service

↓

User Repository

↓

Password Verification

↓

JWT Generation

↓

Client Receives Token

↓

Authenticated API Requests

↓

JWT Validation

↓

Authorized Business Operation
```

The backend validates every protected request independently.

---

## 10.2 Login Process

The login process consists of:

1. receive user credentials;
2. validate request format;
3. retrieve user record;
4. verify password hash;
5. generate JWT;
6. return authentication response.

Passwords are never stored or transmitted in plain text.

---

## 10.3 JWT Authentication

JWT tokens contain the information required to identify the authenticated user.

Protected endpoints require a valid JWT before business processing begins.

Expired, invalid, or malformed tokens are rejected immediately.

---

## 10.4 Authentication Responsibilities

The Authentication Service is responsible for:

- verifying credentials;
- generating tokens;
- validating tokens;
- retrieving authenticated users;
- rejecting unauthorized requests.

Authentication services do not implement application business rules.

---

## 10.5 Authorization

After authentication succeeds, authorization determines whether the user has permission to perform the requested operation.

Authorization decisions are based on:

- authenticated identity;
- assigned role;
- requested operation.

Unauthorized requests are rejected before business logic executes.

---

## 10.6 Protected Resources

Authentication is required for all protected endpoints.

Examples include:

- event management;
- attendance operations;
- presence updates;
- emergency ticket operations;
- administrative reporting.

Public endpoints require no authentication unless explicitly specified.

---

## 10.7 Dependency Injection

Authentication uses FastAPI dependency injection.

Protected routes obtain the authenticated user through reusable authentication dependencies rather than implementing authentication logic directly.

This ensures consistent authentication across the application.

---

## 10.8 Security Principles

Authentication follows these principles:

- passwords are securely hashed;
- tokens are signed by the server;
- server validates every request;
- authentication is stateless;
- clients are never trusted without verification.

The backend remains the sole authority for user identity.

---

## 10.9 Error Handling

Authentication failures produce standardized error responses.

Examples include:

- invalid credentials;
- expired token;
- invalid token;
- unauthorized request;
- insufficient permissions.

Authentication failures prevent further request processing.

---

## 10.10 Audit Integration

Successful and failed authentication events may generate audit log records where appropriate.

Audit logging supports security monitoring and administrative accountability while remaining independent of authentication logic.


# 11. Event Management Architecture

The Event Management Architecture is responsible for the complete lifecycle of events.

It coordinates event creation, updates, activation, completion, cancellation, and availability while ensuring that all event state transitions comply with the application's business rules.

The backend remains the sole authority for determining an event's current state.

---

## 11.1 Architecture Overview

The Event Management module coordinates multiple architectural components.

```text
Client

↓

Event Route

↓

Event Service

↓

Event Repository

↓

PostgreSQL

↓

Notification Service (when applicable)

↓

Activity History Service

↓

Audit Log Service
```

Each component performs a single responsibility while collaborating through well-defined service interfaces.

---

## 11.2 Event Lifecycle

Every event progresses through a controlled lifecycle.

```text
DRAFT

↓

SCHEDULED

↓

ACTIVE

↓

COMPLETED

or

↓

CANCELLED
```

Only valid state transitions are permitted.

The backend validates every transition before updating persistent data.

---

## 11.3 Event Creation

Creating an event follows this workflow.

```text
Administrator Request

↓

Request Validation

↓

Authorization

↓

Event Service

↓

Business Validation

↓

Event Repository

↓

Database

↓

Audit Log

↓

Response
```

The Event Service validates:

- required event information;
- event schedule;
- boundary configuration;
- business constraints.

Successful creation generates the corresponding audit records.

---

## 11.4 Event Updates

Event modifications are coordinated by the Event Service.

The service validates:

- current event state;
- requested modifications;
- business constraints.

Only permitted fields may be modified based on the event's current lifecycle state.

---

## 11.5 Event Activation

When an event becomes active, the backend performs the required transition.

```text
Administrator

↓

Event Service

↓

State Validation

↓

Database Update

↓

Activity Generation

↓

Audit Logging
```

Only scheduled events may transition to the active state.

---

## 11.6 Event Completion

Completing an event finalizes attendance operations.

The backend:

- updates the event status;
- finalizes active attendance where applicable;
- records historical information;
- generates activity history;
- generates audit records.

Completed events become read-only except for administrative operations permitted by business rules.

---

## 11.7 Event Cancellation

Cancelled events remain preserved as historical records.

Cancellation:

- prevents new attendance;
- preserves historical data;
- records administrative actions.

Cancellation does not remove the event from the database.

---

## 11.8 Event Validation

The Event Service validates:

- event existence;
- event status;
- administrator permissions;
- scheduling constraints;
- boundary configuration.

Validation occurs before any database updates.

---

## 11.9 Service Collaboration

Event operations may interact with additional services.

```text
Event Service

↓

Notification Service

↓

Activity History Service

↓

Audit Log Service
```

Each supporting service performs its own responsibility independently.

---

## 11.10 Design Principles

The Event Management Architecture follows these principles:

- centralized lifecycle management;
- backend-controlled state transitions;
- immutable historical records;
- transactional consistency;
- independent service responsibilities.

These principles ensure reliable event management throughout the application.


# Activity Management Architecture

The Activity Management module extends the attendance system by allowing administrators to create, assign, monitor, and review volunteer activities during an event.

Activities are independent of attendance records and follow their own lifecycle from creation through review.

---

## Module Components

The Activity Management module consists of the following components:

- Activity Management
- Activity Assignment
- Evidence Management
- Activity Review
- Activity Templates

Each component is implemented as an independent service while following the project's layered architecture.

---

## High-Level Architecture

```
Administrator

      │

      ▼

Activity API

      │

      ▼

Activity Service

      │

      ▼

Repositories

      │

      ▼

PostgreSQL Database
```

---

## Activity Lifecycle

```
DRAFT

    │

    ▼

PUBLISHED

    │

    ▼

ASSIGNED

    │

    ▼

IN_PROGRESS

    │

    ▼

UNDER_REVIEW

    │

    ├────────────► VERIFIED

    │

    └────────────► NEEDS_CHANGES
                        │
                        ▼
                  IN_PROGRESS
```

---

## Assignment Architecture

One activity may be assigned to multiple volunteers.

Each assignment maintains an independent execution lifecycle.

```
Activity

      │

      ├────────────► Volunteer A

      │                  │

      │                  ▼

      │            Assignment A

      │

      ├────────────► Volunteer B

      │                  │

      │                  ▼

      │            Assignment B

      │

      └────────────► Volunteer C

                         │

                         ▼

                   Assignment C
```

Each assignment progresses independently through execution and review.

---

## Evidence Management

Evidence is attached directly to activity assignments.

Members may upload multiple evidence items while completing an assigned activity before submitting it for administrator review.

```
Assignment

      │

      ├──────────► Photo

      │

      ├──────────► Photo

      │

      └──────────► Video
```

This structure preserves the context of each evidence item.

---

## Review Workflow

Submitted activities are reviewed by an administrator.

```
UNDER_REVIEW

      │

      ├────────────► VERIFIED

      │

      └────────────► NEEDS_CHANGES
                            │
                            ▼
                      IN_PROGRESS
```

Each review action creates a new review record, preserving the complete review history.

---

## Template Architecture

Templates simplify activity creation for recurring event types.

```
Activity Template

        │

        ▼

Template Items

        │

        ▼

Generated Activities

        │

        ▼

Assignments
```

Generated activities become independent records. Updating a template affects only future activities created from it.

---

## Design Principles

The Activity Management module follows these principles:

- Activities belong to events.
- Activities are independent of attendance records.
- One activity may be assigned to multiple volunteers.
- Every volunteer maintains an independent execution lifecycle.
- Evidence belongs to activity assignments.
- Assignment submissions require evidence before review.
- Review history is permanently preserved.
- Templates generate new activities without modifying existing ones.


# 12. Attendance Architecture

The Attendance Architecture manages the complete attendance lifecycle for every participant.

It coordinates attendance validation, presence monitoring, boundary evaluation, emergency ticket workflows, volunteer eligibility, notifications, and historical record generation.

Attendance processing is entirely backend-controlled.

---

## 12.1 Architecture Overview

Attendance operations involve several collaborating services.

```text
Client

↓

Attendance Route

↓

Attendance Service

↓

Boundary Service

↓

Presence Service

↓

Repositories

↓

Database

↓

Notification Service

↓

Activity History Service

↓

Audit Log Service
```

Each service performs a specific responsibility while remaining independent of presentation concerns.

---

## 12.2 Attendance Lifecycle

Attendance progresses through controlled state transitions.

```text
Check In

↓

Presence Monitoring

↓

Boundary Evaluation

↓

Emergency Ticket (when required)

↓

Check Out

↓

Attendance Completed
```

Every transition is validated by the backend before being persisted.

---

## 12.3 Check-In Workflow

The check-in process follows this sequence.

```text
Member Request

↓

Authentication

↓

Attendance Service

↓

Business Validation

↓

Boundary Validation

↓

Attendance Repository

↓

Database

↓

Activity History

↓

Audit Log

↓

Response
```

The Attendance Service validates:

- user eligibility;
- event status;
- duplicate attendance;
- boundary compliance.

Only successful validations result in attendance creation.

---

## 12.4 Active Attendance

After check-in, attendance becomes active.

Active attendance enables:

- presence monitoring;
- boundary evaluation;
- emergency ticket processing;
- administrative supervision.

Only one active attendance record may exist for a participant within an event.

---

## 12.5 Presence Integration

Attendance and presence monitoring operate as separate architectural concerns.

Attendance Service is responsible for attendance lifecycle.

Presence Service is responsible for:

- location updates;
- presence state;
- historical presence records.

Attendance uses presence information but does not perform location processing itself.

---

## 12.6 Boundary Evaluation

Boundary validation is delegated to the Boundary Service.

```text
Location Update

↓

Boundary Service

↓

GeoJSON / Captured GPS Validation

↓

Presence Result

↓

Attendance Service
```

Separating boundary evaluation improves maintainability and allows future boundary strategies to be introduced without affecting attendance logic.

---

## 12.7 Emergency Ticket Integration

When a participant requires temporary leave, Attendance coordinates with the Emergency Ticket Service.

```text
Attendance Service

↓

Emergency Ticket Service

↓

Administrator Decision

↓

Attendance Update
```

The Attendance Service remains responsible for attendance state while the Emergency Ticket Service manages the approval workflow.

---

## 12.8 Attendance Completion

Attendance concludes when:

- the participant checks out;
- the event completes and automatic completion occurs where applicable.

Completion updates attendance records and generates associated historical records.

---

## 12.9 Supporting Services

Attendance operations automatically interact with supporting services.

```text
Attendance Service

↓

Notification Service

↓

Activity History Service

↓

Audit Log Service
```

Supporting services execute independently without modifying attendance business logic.

---

## 12.10 Transaction Management

Attendance operations frequently modify multiple entities.

Examples include:

- attendance records;
- presence logs;
- notifications;
- activity history;
- audit logs.

These related updates should be coordinated within a single transaction whenever consistency requires it.

---

## 12.11 Design Principles

The Attendance Architecture follows these principles:

- backend-controlled attendance lifecycle;
- independent presence monitoring;
- reusable boundary validation;
- transactional consistency;
- immutable historical records;
- modular service collaboration.

These principles provide a scalable foundation for attendance management while preserving clean separation between architectural components.

# 13. Presence Monitoring Architecture

The Presence Monitoring Architecture continuously evaluates a participant's location throughout an active attendance session.

It is responsible for determining whether a participant is currently inside or outside the configured event boundary while maintaining a complete historical record of presence changes.

Presence monitoring operates independently of attendance management while providing information required by the Attendance Service.

---

## 13.1 Architecture Overview

Presence monitoring coordinates several backend components.

```text
Client Location Update

↓

Presence Route

↓

Presence Service

↓

Boundary Service

↓

Presence Repository

↓

Database

↓

Notification Service

↓

Activity History Service

↓

Audit Log Service
```

Each service performs a single responsibility while remaining independent of other business domains.

---

## 13.2 Monitoring Lifecycle

Presence monitoring begins after successful attendance check-in.

```text
Check In

↓

Monitoring Active

↓

Location Updates

↓

Boundary Evaluation

↓

Presence State Changes

↓

Check Out

↓

Monitoring Ends
```

Monitoring remains active only while attendance is active.

---

## 13.3 Location Processing

Each location update follows the same workflow.

```text
Location Update

↓

Schema Validation

↓

Presence Service

↓

Boundary Service

↓

Presence Evaluation

↓

Presence Repository

↓

Supporting Services
```

The backend determines the participant's current presence state using the configured boundary definition.

---

## 13.4 Boundary Evaluation

The Presence Service delegates all geographic calculations to the Boundary Service.

The Boundary Service supports:

- GeoJSON boundaries;
- captured GPS point validation.

The Presence Service receives only the evaluation result and does not perform geographic calculations itself.

---

## 13.5 Presence State Management

The Presence Service maintains the participant's current presence state.

Possible state transitions include:

```text
CHECK_IN

↓

PRESENT

↓

LEFT

↓

RETURNED

↓

CHECK_OUT
```

Only valid state transitions are permitted.

Each transition is validated before being persisted.

---

## 13.6 Presence History

Every presence state transition is recorded.

Historical records include:

- previous state;
- new state;
- timestamp;
- related attendance;
- related event.

Historical records provide a complete chronological timeline of participant movement throughout the event.

---

## 13.7 Service Collaboration

Presence monitoring collaborates with multiple supporting services.

```text
Presence Service

↓

Notification Service

↓

Activity History Service

↓

Audit Log Service
```

Each supporting service performs its own responsibility without modifying presence business logic.

---

## 13.8 Performance Considerations

Presence monitoring may receive frequent location updates.

The architecture should:

- minimize unnecessary database operations;
- avoid duplicate presence records;
- evaluate only meaningful state changes;
- optimize frequently executed queries.

Performance optimizations must not compromise business rule correctness.

---

## 13.9 Error Handling

Presence monitoring gracefully handles:

- invalid location updates;
- missing attendance records;
- invalid event states;
- boundary evaluation failures.

Errors should return standardized API responses without exposing internal implementation details.

---

## 13.10 Design Principles

The Presence Monitoring Architecture follows these principles:

- continuous backend evaluation;
- independent boundary processing;
- immutable presence history;
- reusable location validation;
- modular service collaboration;
- scalable processing for continuous updates.


# 14. Boundary Validation Architecture

The Boundary Validation Architecture provides a centralized mechanism for determining whether a participant's reported location satisfies the configured event boundary.

Boundary validation is isolated within a dedicated service so that geographic calculations remain reusable and independent of attendance or presence workflows.

---

## 14.1 Architecture Overview

Boundary validation operates as a shared backend component.

```text
Presence Service

↓

Boundary Service

↓

Boundary Repository

↓

Boundary Definition

↓

Validation Result
```

Other services consume only the validation result rather than implementing geographic calculations.

---

## 14.2 Supported Boundary Types

The architecture supports multiple boundary definitions.

Examples include:

- GeoJSON boundaries;
- captured GPS point boundaries.

The validation strategy is selected according to the event configuration.

The calling services remain unaware of the underlying implementation.

---

## 14.3 Validation Workflow

Each validation request follows a consistent sequence.

```text
Location Coordinates

↓

Boundary Service

↓

Load Event Boundary

↓

Evaluate Position

↓

Determine Result

↓

Return Validation Response
```

The Boundary Service performs all geographic processing internally.

---

## 14.4 Boundary Service Responsibilities

The Boundary Service is responsible for:

- loading boundary definitions;
- validating participant locations;
- selecting the appropriate validation strategy;
- returning standardized validation results.

The service does not modify attendance or presence records.

---

## 14.5 Service Integration

Boundary validation is used by multiple business modules.

```text
Attendance Service

↓

Boundary Service

↑

Presence Service
```

This shared architecture eliminates duplicated geographic logic across the application.

---

## 14.6 Validation Results

Boundary evaluation produces a standardized result that may include:

- validation outcome;
- boundary type;
- evaluation timestamp;
- supporting metadata where applicable.

Business services determine how the result affects application workflows.

---

## 14.7 Extensibility

The Boundary Service is designed to support additional validation strategies in the future.

New boundary implementations should integrate without requiring changes to:

- Attendance Service;
- Presence Service;
- Route Layer.

This preserves architectural stability while allowing future expansion.

---

## 14.8 Performance Considerations

Boundary evaluation should:

- minimize computational overhead;
- reuse loaded boundary definitions where appropriate;
- avoid redundant evaluations;
- remain efficient for frequent location updates.

Performance optimizations must preserve validation accuracy.

---

## 14.9 Error Handling

Boundary validation handles:

- missing boundary definitions;
- invalid geographic data;
- unsupported boundary types;
- malformed boundary configurations.

Errors are translated into standardized application exceptions.

---

## 14.10 Design Principles

The Boundary Validation Architecture follows these principles:

- centralized geographic processing;
- reusable validation logic;
- independent service design;
- implementation abstraction;
- scalable validation strategies;
- consistent evaluation results.

# 15. Emergency Ticket Architecture

The Emergency Ticket Architecture manages temporary leave requests submitted by participants during an active attendance session.

It coordinates ticket submission, administrator review, approval decisions, and attendance updates while maintaining complete historical records of every request.

The Emergency Ticket Service owns the entire emergency ticket lifecycle.

---

## 15.1 Architecture Overview

Emergency ticket processing involves multiple collaborating services.

```text
Member

↓

Emergency Ticket Route

↓

Emergency Ticket Service

↓

Emergency Ticket Repository

↓

Database

↓

Notification Service

↓

Activity History Service

↓

Audit Log Service
```

Each service performs an independent responsibility while remaining loosely coupled to the rest of the system.

---

## 15.2 Ticket Submission Workflow

Submitting an emergency ticket follows a standardized workflow.

```text
Active Attendance

↓

Submit Ticket

↓

Schema Validation

↓

Business Validation

↓

Emergency Ticket Repository

↓

Database

↓

Notification Generation

↓

Activity History

↓

Audit Log
```

Only participants with an active attendance session may submit emergency tickets.

---

## 15.3 Administrative Review

Emergency tickets require administrator review before reaching a final decision.

```text
Administrator

↓

Emergency Ticket Service

↓

Retrieve Ticket

↓

Business Validation

↓

Decision

↓

Database Update

↓

Supporting Services
```

The Emergency Ticket Service validates:

- ticket existence;
- current ticket status;
- administrator permissions;
- attendance eligibility.

---

## 15.4 Ticket Lifecycle

Emergency tickets follow a controlled lifecycle.

```text
PENDING

├── APPROVED

├── REJECTED

└── CANCELLED
```

Only valid state transitions are permitted.

Completed ticket decisions are recorded permanently.

---

## 15.5 Attendance Integration

Emergency tickets integrate directly with the Attendance Service.

```text
Attendance Service

↓

Emergency Ticket Service

↓

Decision

↓

Attendance Update
```

The Emergency Ticket Service manages the approval workflow.

The Attendance Service remains responsible for attendance state management.

---

## 15.6 Notification Integration

Ticket operations automatically generate notifications where appropriate.

Examples include:

- ticket submitted;
- ticket approved;
- ticket rejected;
- ticket cancelled.

Notification generation is delegated to the Notification Service.

---

## 15.7 Historical Records

Emergency ticket operations automatically generate:

- activity history;
- audit log entries.

These supporting records preserve the complete history of each ticket.

---

## 15.8 Transaction Management

Ticket processing may update multiple entities.

Examples include:

- emergency ticket;
- attendance record;
- notifications;
- activity history;
- audit logs.

These updates should be coordinated within a single transaction whenever consistency requires it.

---

## 15.9 Error Handling

The Emergency Ticket Architecture handles:

- ticket not found;
- invalid ticket state;
- duplicate requests;
- unauthorized review;
- invalid attendance state.

Errors are translated into standardized API responses.

---

## 15.10 Design Principles

The Emergency Ticket Architecture follows these principles:

- backend-controlled workflow;
- centralized approval logic;
- immutable historical records;
- modular service collaboration;
- transactional consistency;
- independent business responsibilities.

# 16. Volunteer Block Architecture

The Volunteer Block Architecture manages temporary participation restrictions applied to volunteers.

It ensures that participation eligibility is evaluated consistently before attendance is created while preserving complete administrative history.

The Volunteer Block Service is responsible for enforcing volunteer participation restrictions throughout the application.

---

## 16.1 Architecture Overview

Volunteer block processing is coordinated through dedicated backend services.

```text
Administrator

↓

Volunteer Block Route

↓

Volunteer Block Service

↓

Volunteer Block Repository

↓

Database

↓

Notification Service

↓

Activity History Service

↓

Audit Log Service
```

Each architectural component performs a single responsibility.

---

## 16.2 Block Creation Workflow

Applying a volunteer block follows this workflow.

```text
Administrator Request

↓

Authorization

↓

Volunteer Block Service

↓

Business Validation

↓

Volunteer Block Repository

↓

Database

↓

Supporting Services
```

Only authorized administrators may create volunteer blocks.

---

## 16.3 Eligibility Evaluation

Participation eligibility is evaluated before attendance is created.

```text
Attendance Request

↓

Attendance Service

↓

Volunteer Block Service

↓

Eligibility Check

↓

Allow

or

Reject
```

Attendance processing depends only on the eligibility result.

The Attendance Service does not manage volunteer block records directly.

---

## 16.4 Block Lifecycle

Volunteer blocks follow a controlled lifecycle.

```text
ACTIVE

↓

REMOVED
```

Historical block records remain permanently stored after removal.

---

## 16.5 Attendance Integration

Volunteer eligibility is verified before:

- attendance check-in;
- event participation;
- other protected participation workflows where applicable.

The Volunteer Block Service determines eligibility.

The Attendance Service enforces the decision.

---

## 16.6 Administrative Management

Volunteer block administration includes:

- block creation;
- block retrieval;
- block removal;
- historical review.

Administrative operations are performed through dedicated backend services.

---

## 16.7 Supporting Services

Volunteer block operations automatically generate:

- notifications;
- activity history;
- audit log records.

These supporting services remain independent from volunteer block management.

---

## 16.8 Transaction Management

Volunteer block operations may modify multiple entities.

Examples include:

- volunteer block records;
- notifications;
- activity history;
- audit logs.

Related updates should be committed atomically.

---

## 16.9 Error Handling

Volunteer block processing handles:

- volunteer not found;
- duplicate active block;
- invalid block state;
- unauthorized operations;
- missing volunteer block.

Errors are returned using the application's standardized error format.

---

## 16.10 Design Principles

The Volunteer Block Architecture follows these principles:

- centralized eligibility management;
- backend-controlled enforcement;
- immutable historical records;
- independent service collaboration;
- transactional consistency;
- reusable participation validation.

# 17. Notification Architecture

The Notification Architecture is responsible for informing users about significant business events throughout the application.

Notifications are generated automatically by backend services as part of normal business workflows. The backend determines when notifications are created, who receives them, and their current status.

Notification generation is centralized to ensure consistent user communication across the system.

---

## 17.1 Architecture Overview

Notification processing is shared across multiple business domains.

```text
Business Service

↓

Notification Service

↓

Notification Repository

↓

Database

↓

Client Retrieves Notifications
```

Business services request notification generation without needing to understand how notifications are stored or managed.

---

## 17.2 Notification Sources

Notifications may be generated by:

- Event Service
- Attendance Service
- Presence Service
- Emergency Ticket Service
- Volunteer Block Service

Each service remains responsible only for initiating notification requests.

The Notification Service owns notification persistence.

---

## 17.3 Notification Workflow

Every notification follows the same processing sequence.

```text
Business Event

↓

Notification Service

↓

Create Notification

↓

Notification Repository

↓

Database

↓

Available to User
```

Notification creation occurs automatically after successful completion of the initiating business operation.

---

## 17.4 Notification Lifecycle

Notifications follow a simple lifecycle.

```text
UNREAD

↓

READ
```

Only valid lifecycle transitions are permitted.

Read notifications remain available for future reference.

---

## 17.5 Notification Ownership

Every notification belongs to exactly one user.

Notifications may optionally reference:

- event;
- attendance record;
- emergency ticket;
- volunteer block.

Ownership is determined by the originating business workflow.

---

## 17.6 Service Integration

Notification generation integrates with multiple backend services.

```text
Attendance Service

↓

Notification Service

↑

Emergency Ticket Service

↑

Volunteer Block Service

↑

Event Service
```

The Notification Service remains independent from the business logic of the calling services.

---

## 17.7 Notification Retrieval

Users retrieve notifications through the Notification Route.

The Notification Service is responsible for:

- retrieving notifications;
- filtering notifications;
- updating notification status.

Business services do not access notification data directly.

---

## 17.8 Performance Considerations

Notification queries should:

- retrieve unread notifications efficiently;
- support pagination;
- minimize unnecessary joins;
- use indexed lookup columns.

Notification generation should not significantly increase request processing time.

---

## 17.9 Error Handling

Notification processing handles:

- invalid notification identifiers;
- unauthorized notification access;
- missing notification records.

Errors are translated into standardized API responses.

---

## 17.10 Design Principles

The Notification Architecture follows these principles:

- centralized notification generation;
- independent notification management;
- immutable notification history;
- reusable notification workflows;
- modular service integration.

# 18. Activity History Architecture

The Activity History Architecture maintains a chronological timeline of significant business events associated with each user.

Activity history provides users and administrators with a historical view of completed actions while remaining separate from administrative audit logs.

Activity history is generated automatically by backend services.

---

## 18.1 Architecture Overview

Activity history is generated by multiple business modules.

```text
Business Service

↓

Activity History Service

↓

Activity History Repository

↓

Database

↓

Client Timeline
```

The Activity History Service provides a centralized mechanism for recording business events.

---

## 18.2 Activity Sources

Activity records may be generated by:

- Attendance Service;
- Presence Service;
- Event Service;
- Emergency Ticket Service;
- Volunteer Block Service.

Business services initiate activity generation after successful completion of business operations.

---

## 18.3 Activity Workflow

Every activity record follows the same processing sequence.

```text
Business Event

↓

Activity History Service

↓

Generate Activity

↓

Repository

↓

Database
```

Activity generation occurs automatically and requires no additional client interaction.

---

## 18.4 Activity Timeline

Activity history maintains a chronological sequence of completed business events.

Examples include:

- checked in;
- checked out;
- left event boundary;
- returned to event boundary;
- emergency ticket submitted;
- emergency ticket approved;
- volunteer block removed.

Historical ordering is determined using server-generated timestamps.

---

## 18.5 Activity Ownership

Every activity record belongs to exactly one user.

Records may optionally reference:

- event;
- attendance record;
- emergency ticket.

Activity ownership is determined by the originating business operation.

---

## 18.6 Service Integration

Activity history integrates with all major business modules.

```text
Attendance Service

↓

Activity History Service

↑

Presence Service

↑

Emergency Ticket Service

↑

Volunteer Block Service
```

Business services remain independent from activity persistence.

---

## 18.7 Historical Preservation

Activity records are permanent historical records.

Normal application operations should not modify or delete historical activity.

Historical integrity supports reporting and user timelines.

---

## 18.8 Performance Considerations

Activity queries should:

- support pagination;
- retrieve chronological records efficiently;
- use indexed timestamps;
- avoid unnecessary joins.

Frequently accessed activity timelines should remain performant.

---

## 18.9 Error Handling

Activity retrieval handles:

- invalid users;
- unauthorized access;
- invalid query parameters.

Responses follow the application's standardized error format.

---

## 18.10 Design Principles

The Activity History Architecture follows these principles:

- centralized activity generation;
- immutable historical records;
- chronological consistency;
- independent service integration;
- reusable business event recording.


# 19. Audit Logging Architecture

The Audit Logging Architecture records administrative and security-sensitive operations performed within the backend.

Audit logs provide accountability, troubleshooting, and compliance support while remaining separate from user-facing activity history.

The Audit Log Service owns all audit record generation.

---

## 19.1 Architecture Overview

Audit logging is shared across administrative workflows.

```text
Business Service

↓

Audit Log Service

↓

Audit Log Repository

↓

Database

↓

Administrative Review
```

Audit logging occurs automatically after significant administrative or security-related operations.

---

## 19.2 Audit Sources

Audit records may be generated by:

- Authentication Service;
- Event Service;
- Attendance Service;
- Emergency Ticket Service;
- Volunteer Block Service.

Administrative operations should generate audit records where appropriate.

---

## 19.3 Audit Workflow

Every audit record follows a consistent workflow.

```text
Administrative Action

↓

Audit Log Service

↓

Create Audit Record

↓

Repository

↓

Database
```

Audit generation occurs automatically after successful completion of the initiating operation.

---

## 19.4 Recorded Information

Audit records may include:

- actor;
- action performed;
- affected entity;
- timestamp;
- contextual metadata.

Sensitive information must never be recorded.

---

## 19.5 Administrative Access

Audit records are intended for authorized administrative users only.

Normal users must not access audit history.

Authorization is enforced before audit retrieval.

---

## 19.6 Service Integration

Audit logging integrates with all administrative business services.

```text
Event Service

↓

Audit Log Service

↑

Attendance Service

↑

Emergency Ticket Service

↑

Volunteer Block Service
```

Audit logging remains independent from business workflow implementation.

---

## 19.7 Historical Preservation

Audit records are immutable.

Historical audit information must remain available for administrative review.

Normal application operations should not modify audit history.

---

## 19.8 Performance Considerations

Audit queries should:

- support filtering;
- support pagination;
- optimize chronological retrieval;
- minimize impact on business operations.

Audit logging should not significantly delay request processing.

---

## 19.9 Error Handling

Audit operations handle:

- unauthorized access;
- invalid audit identifiers;
- invalid query parameters.

Errors follow the standardized API response format.

---

## 19.10 Design Principles

The Audit Logging Architecture follows these principles:

- centralized audit generation;
- immutable audit history;
- secure administrative access;
- independent service integration;
- reliable accountability.

# 20. Middleware

Middleware provides cross-cutting functionality that is applied to every incoming request before it reaches the Route Layer and before the response is returned to the client.

Middleware centralizes infrastructure concerns while keeping business logic isolated within the Service Layer.

---

## 20.1 Responsibilities

Middleware is responsible for:

- request processing;
- response processing;
- request logging;
- CORS handling;
- exception handling;
- request tracing.

Middleware must never implement business rules.

---

## 20.2 Middleware Pipeline

Every request passes through the middleware pipeline.

```text
HTTP Request

↓

Logging Middleware

↓

CORS Middleware

↓

Exception Middleware

↓

Route Handler

↓

HTTP Response
```

Each middleware component performs an independent responsibility.

---

## 20.3 Logging Middleware

Logging middleware records request and response information for operational monitoring.

Typical information includes:

- request method;
- request path;
- response status;
- processing duration.

Sensitive information must not be logged.

---

## 20.4 CORS Middleware

CORS middleware controls which client applications may communicate with the backend.

CORS configuration should be managed centrally and remain consistent across all routes.

---

## 20.5 Exception Middleware

Exception middleware intercepts unhandled exceptions and converts them into standardized API responses.

Application errors should be returned consistently regardless of where they originate.

---

## 20.6 Design Principles

Middleware should:

- remain stateless;
- execute quickly;
- avoid business logic;
- remain reusable;
- be independently testable.

# 21. Validation Strategy

Validation protects the integrity of application data by ensuring that only valid requests reach the business layer.

Validation is performed at multiple architectural layers.

---

## 21.1 Validation Layers

The backend uses a layered validation approach.

```text
Client Input

↓

Schema Validation

↓

Authentication

↓

Authorization

↓

Business Validation

↓

Database Constraints
```

Each layer validates a different aspect of the request.

---

## 21.2 Schema Validation

Schema validation is performed using Pydantic.

Schema validation verifies:

- required fields;
- data types;
- value constraints;
- request structure.

Invalid requests are rejected before business processing begins.

---

## 21.3 Business Validation

Business validation is performed by the Service Layer.

Examples include:

- attendance eligibility;
- event lifecycle validation;
- volunteer eligibility;
- emergency ticket rules;
- boundary evaluation.

---

## 21.4 Database Constraints

Database constraints provide the final layer of protection.

Examples include:

- primary keys;
- foreign keys;
- unique constraints;
- check constraints.

Database constraints complement application validation but do not replace it.

---

## 21.5 Validation Principles

Validation should be:

- centralized;
- deterministic;
- reusable;
- consistent;
- performed before persistence.

# 22. Transaction Management

Transactions ensure that related database operations either complete successfully together or are rolled back together.

The backend coordinates transactions within the Service Layer.

---

## 22.1 Transaction Boundaries

A transaction begins when a business operation starts and completes after all related persistence operations succeed.

```text
Business Operation

↓

Database Updates

↓

Commit

or

Rollback
```

---

## 22.2 Transaction Coordination

Services coordinate transactions involving multiple repositories.

Examples include:

- attendance check-in;
- emergency ticket approval;
- volunteer block creation;
- event completion.

---

## 22.3 Rollback Strategy

If any operation fails:

- pending changes are rolled back;
- database consistency is preserved;
- the client receives an appropriate error response.

---

## 22.4 Design Principles

Transactions should be:

- atomic;
- consistent;
- isolated;
- durable;
- limited to the smallest practical scope.

# 23. Error Handling

The backend uses centralized error handling to provide consistent responses across all application modules.

Errors are translated into standardized API responses before being returned to clients.

---

## 23.1 Error Sources

Errors may originate from:

- schema validation;
- authentication;
- authorization;
- business validation;
- repositories;
- infrastructure.

---

## 23.2 Error Processing

```text
Application Error

↓

Exception Handler

↓

Standardized API Response

↓

Client
```

---

## 23.3 Error Categories

Common categories include:

- validation errors;
- authentication errors;
- authorization errors;
- business rule violations;
- resource not found;
- unexpected server errors.

---

## 23.4 Design Principles

Error handling should:

- be centralized;
- remain consistent;
- avoid exposing internal implementation details;
- provide meaningful client feedback.

# 24. Dependency Injection

The backend uses FastAPI's dependency injection system to provide reusable application resources.

Dependency injection reduces coupling between architectural components.

---

## 24.1 Common Dependencies

Examples include:

- authenticated user;
- database session;
- application configuration;
- shared services.

---

## 24.2 Benefits

Dependency injection provides:

- loose coupling;
- improved testability;
- code reuse;
- centralized resource management.

---

## 24.3 Design Principles

Dependencies should:

- remain reusable;
- remain stateless where practical;
- expose clearly defined interfaces;
- simplify testing.

# 25. Async Processing

The backend uses asynchronous processing to improve scalability and efficiently handle concurrent requests.

---

## 25.1 Async Operations

Asynchronous execution is used for:

- database operations;
- file operations;
- external service communication.

Blocking operations should be avoided whenever possible.

---

## 25.2 Design Principles

Async code should:

- avoid blocking the event loop;
- use asynchronous libraries;
- await all I/O operations;
- maintain predictable execution flow.

---

## 25.3 Benefits

Asynchronous processing provides:

- improved throughput;
- better resource utilization;
- greater scalability;
- responsive request handling.

# 26. File Storage

The backend manages uploaded files separately from business data.

Persistent file references are stored in the database while file content is stored within the configured storage location.

---

## 26.1 Responsibilities

File storage is responsible for:

- storing uploaded files;
- retrieving stored files;
- maintaining file references;
- preserving file integrity.

---

## 26.2 Storage Strategy

---

## Activity Evidence Storage

Activity evidence is stored separately from attendance evidence.

Each uploaded file is associated with an activity assignment and referenced by its metadata in the database.

Example directory structure:

```text
media/

├── attendance/
│
└── activities/
    ├── photos/
    └── videos/
```

Only file metadata is stored in PostgreSQL.

The physical files remain in the media storage directory while database records maintain the relationship between activities, progress updates, and evidence.

Evidence files are optimized before storage to reduce disk usage while maintaining sufficient quality for administrative review.

---

## 26.3 Design Principles

File storage should:

- preserve integrity;
- prevent filename conflicts;
- support future storage providers;
- remain independent of business services.

# 27. Logging

Logging provides operational visibility into backend execution.

Logs support troubleshooting, monitoring, and operational analysis.

---

## 27.1 Logging Scope

The backend logs:

- application startup;
- request processing;
- significant business events;
- unexpected failures.

Sensitive information must never be logged.

---

## 27.2 Logging Principles

Logging should:

- remain consistent;
- avoid duplication;
- provide sufficient operational context;
- support troubleshooting.

Business event logging remains separate from audit logging.

# 28. Security Architecture

Security is integrated throughout the backend architecture.

Every architectural layer contributes to protecting application data and business operations.

---

## 28.1 Security Components

Security includes:

- authentication;
- authorization;
- password hashing;
- JWT validation;
- schema validation;
- business validation.

---

## 28.2 Defense in Depth

The backend applies multiple security layers.

```text
Client

↓

Authentication

↓

Authorization

↓

Validation

↓

Business Rules

↓

Database Constraints
```

Each layer contributes to overall system security.

---

## 28.3 Security Principles

The backend follows these principles:

- least privilege;
- backend authority;
- secure defaults;
- defense in depth;
- server-side validation.

# 29. Performance Considerations

### Activity Module Performance

- Activities are indexed by event, status, category, and priority.
- Assignment queries are indexed by volunteer and activity.
- Progress updates are retrieved in chronological order using indexed timestamps.
- Evidence metadata is indexed for efficient review retrieval.
- Activity reports are generated on demand rather than stored.
- Template application creates activities in bulk to reduce repeated database operations.

---

## 29.1 Performance Strategies

Examples include:

- efficient database queries;
- indexed lookups;
- pagination;
- asynchronous processing;
- minimizing unnecessary database operations.

---

## 29.2 Resource Utilization

Application resources should be used efficiently.

Examples include:

- connection pooling;
- optimized transactions;
- minimizing memory usage.

---

## 29.3 Design Principles

Performance improvements should:

- preserve correctness;
- remain measurable;
- maintain readability;
- avoid premature optimization.

# 30. Scalability

The backend architecture is designed to accommodate future growth while minimizing architectural changes.

Scalability is achieved through modular design and clear separation of responsibilities.

---

## 30.1 Architectural Scalability

The architecture supports:

- additional business modules;
- new API endpoints;
- expanded reporting;
- future integrations;
- increased user activity.

---

## 30.2 Modular Growth

New functionality should integrate using existing architectural patterns.

Existing services should remain stable while new services are introduced independently.

---

## 30.3 Design Principles

### Activity Layer Scalability

The Activity module is designed to scale independently of the attendance module.

Scalability is achieved through:

- independent services for activity management;
- normalized database relationships;
- indexed assignment and review queries;
- optimized evidence storage;
- reusable activity templates;
- on-demand report generation.


# Phase 4 Backend Architecture Extension

## Objectives
## Architectural Continuity
## Production Hardening Layer
## Background Processing Architecture
### Immediate Background Tasks
### Scheduled Background Tasks
## Synchronization Architecture
## Spoofing Detection Architecture
## Backup Architecture
## Service Collaboration
## Design Principles
## Phase 4 Summary

# 31. Phase 4 Backend Architecture Extension

Phase 4 extends the backend architecture by introducing production hardening capabilities.

Unlike previous phases, Phase 4 does not introduce new business domains. Instead, it strengthens the operational characteristics of the backend through reliable synchronization, comprehensive auditing, security monitoring, backup management, and background processing.

The existing layered architecture established during Phases 1–3 remains unchanged.

---

## 31.1 Objectives

The Phase 4 backend architecture is designed to:

- support reliable offline synchronization;
- improve production readiness;
- centralize audit logging;
- detect suspicious client behaviour;
- automate operational tasks;
- support backup and recovery;
- preserve backward compatibility;
- maintain modular service boundaries.

---

## 31.2 Architectural Continuity

Phase 4 preserves all architectural principles established during previous phases.

The following remain unchanged:

- layered architecture;
- service-oriented business logic;
- repository abstraction;
- transactional database operations;
- JWT authentication;
- asynchronous request processing;
- domain-oriented modules.

Production hardening is implemented by extending existing services and infrastructure rather than redesigning the backend.

---

## 31.3 Production Hardening Layer

A dedicated Production Hardening Layer is introduced to provide shared operational capabilities across the backend.

```
Business Services

        │

        ▼

Production Hardening Layer

        │

        ├── Synchronization Service

        ├── Spoofing Detection Service

        ├── Audit Log Service

        ├── Backup Service

        └── Background Processing
```

The Production Hardening Layer provides infrastructure services without owning business workflows.

---

## 31.4 Background Processing Architecture

Certain backend responsibilities execute outside the normal request-response lifecycle.

Background processing is divided into two independent categories:

- Immediate Background Tasks
- Scheduled Background Tasks

This separation improves maintainability, scalability, and operational troubleshooting.

---

### 31.4.1 Immediate Background Tasks

Immediate background tasks execute immediately after successful business operations.

Typical responsibilities include:

- audit log persistence;
- notification delivery;
- synchronization processing;
- security event recording;
- future asynchronous business operations.

These tasks should not delay client responses while maintaining operational consistency.

---

### 31.4.2 Scheduled Background Tasks

Scheduled background tasks execute independently of user requests.

Typical responsibilities include:

- backup execution;
- backup verification;
- audit retention cleanup;
- log maintenance;
- scheduled operational maintenance;
- future scheduled jobs.

Scheduled processing remains isolated from request handling.

---

## 31.5 Synchronization Architecture

Offline synchronization is coordinated through a dedicated Synchronization Service.

Responsibilities include:

- receiving synchronization requests;
- validating synchronization payloads;
- processing queued operations;
- resolving synchronization conflicts;
- coordinating affected business services;
- returning synchronization results.

The backend remains the authoritative source of truth.

The frontend maintains the offline queue using IndexedDB.

---

## 31.6 Spoofing Detection Architecture

Spoofing detection is implemented as a dedicated backend service.

The Spoofing Detection Service evaluates:

- mock location detection;
- impossible travel;
- abnormal GPS movement;
- emulator detection where supported;
- device time manipulation;
- rooted or jailbroken devices where supported;
- repeated suspicious behaviour.

Detection results generate security events rather than directly enforcing administrative actions.

Administrative review remains responsible for enforcement decisions.

---

## 31.7 Backup Architecture

Backup management is coordinated through a dedicated Backup Service.

Responsibilities include:

- initiating backups;
- tracking backup status;
- validating backup completion;
- verifying backup integrity;
- coordinating recovery procedures;
- maintaining backup metadata.

Backup files remain external to PostgreSQL while operational metadata remains stored within the application database.

---

## 31.8 Service Collaboration

Phase 4 introduces additional collaboration between existing business services and production services.

Example request flow:

```
Attendance Service

        │

        ▼

Presence Service

        │

        ▼

Spoofing Detection Service

        │

        ▼

Audit Log Service

        │

        ▼

Immediate Background Processing
```

Synchronization workflow:

```
Synchronization Service

        │

        ▼

Attendance Service

        │

        ▼

Presence Service

        │

        ▼

Activity Service
```

Scheduled workflow:

```
Scheduler

        │

        ▼

Backup Service

        │

        ▼

Backup Verification

        │

        ▼

Backup Metadata
```

Each service continues to own only its respective business or operational responsibility.

---

## 31.9 Design Principles

Phase 4 follows the following architectural principles:

- extend existing architecture rather than redesigning it;
- preserve single responsibility across services;
- separate business services from production infrastructure;
- isolate request processing from scheduled processing;
- maintain backend authority;
- preserve transactional consistency;
- support independent testing;
- support future scalability;
- remain deployment-platform agnostic.

---

## 31.10 Phase 4 Summary

Phase 4 extends the backend architecture with production hardening capabilities while preserving the layered, modular, and service-oriented architecture established throughout Phases 1–3.

The resulting architecture improves operational reliability, security, auditability, recoverability, and maintainability without altering existing business workflows or API contracts.


# 32. Conclusion

The backend architecture establishes a modular, layered, and service-oriented foundation for the InnoTech Hub Attendance system.

By separating HTTP communication, business logic, persistence, and infrastructure concerns, the architecture promotes maintainability, scalability, and long-term reliability.

The architecture emphasizes:

- backend authority over business decisions;
- clear separation of responsibilities;
- reusable service-oriented components;
- transactional consistency;
- centralized validation;
- secure authentication and authorization;
- comprehensive historical record keeping;
- extensibility for future enhancements.

The architecture additionally supports:

- activity management;
- volunteer activity assignment;
- progress tracking;
- evidence management;
- administrator review workflow;
- reusable activity templates.

The modular architecture allows future Activity Layer enhancements to be implemented without affecting the existing attendance and presence monitoring modules.

Following these architectural principles ensures that the backend remains consistent with the project's API contracts, database schema, and business rules while providing a stable platform for future development.



