# Backend API Implementation

This document describes how the backend APIs are implemented within the InnoTech Hub Attendance System.

Unlike the API contracts, which define endpoint specifications, request schemas, and response models, this document explains how requests are processed internally through the application's architectural layers.

It provides implementation guidance for backend developers while remaining independent of framework-specific code.

---

# 1. Overview

The backend follows a layered implementation model in which every API request passes through a consistent execution pipeline before a response is returned.

Business rules are implemented entirely on the server. Clients submit requests and receive responses but never determine business outcomes.

The implementation strategy emphasizes:

- thin route handlers;
- centralized business logic;
- reusable services;
- repository-based persistence;
- transactional consistency;
- standardized validation;
- comprehensive historical tracking.

Every API follows the same high-level processing pattern.

```text
Client Request

↓

Route Layer

↓

Authentication

↓

Authorization

↓

Schema Validation

↓

Service Layer

↓

Repository Layer

↓

Database

↓

Supporting Services

↓

Response
```

Supporting services include:

- Notification Service
- Activity History Service
- Audit Log Service

These services operate independently from the primary business operation while remaining part of the overall request workflow.

---

# 2. Implementation Principles

The backend implementation follows several architectural principles that remain consistent across every API.

---

## 2.1 Thin Route Handlers

Route handlers are responsible only for HTTP communication.

Their responsibilities include:

- receiving requests;
- validating request schemas;
- authenticating users;
- authorizing access;
- invoking service methods;
- returning responses.

Route handlers do not contain business rules.

---

## 2.2 Service-Oriented Business Logic

Business logic resides exclusively within the Service Layer.

Services are responsible for:

- enforcing business rules;
- coordinating repositories;
- validating business conditions;
- managing transactions;
- invoking supporting services.

Each service owns a clearly defined business domain.

---

## 2.3 Repository-Based Persistence

Repositories provide all database access.

Repositories are responsible for:

- retrieving records;
- creating records;
- updating records;
- deleting records where permitted;
- executing optimized queries.

Business services never communicate directly with the database.

---

## 2.4 Backend Authority

The backend is the single source of truth.

Clients may submit requests and data, but the backend determines:

- whether requests are valid;
- whether operations are permitted;
- attendance state;
- presence state;
- volunteer eligibility;
- event status;
- emergency ticket outcomes.

Business decisions are never delegated to the client.

---

## 2.5 Transactional Consistency

Operations affecting multiple entities are executed within controlled transactions.

Examples include:

- attendance check-in;
- attendance check-out;
- emergency ticket approval;
- volunteer block creation;
- event completion.

If any required operation fails, the transaction is rolled back to preserve data consistency.

---

## 2.6 Standardized Validation

Validation occurs in multiple stages.

```text
Request

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

Each validation layer protects a different aspect of the system.

---

## 2.7 Supporting Services

Many business operations automatically trigger supporting services.

These include:

- notification generation;
- activity history recording;
- audit log creation.

Supporting services remain independent from the initiating business operation.

---

## 2.8 Error Handling

Business errors are converted into standardized API responses.

Internal implementation details are never exposed to clients.

Errors remain consistent regardless of which service generates them.

---

## 2.9 Response Generation

Only successful business operations produce successful responses.

Responses are generated after:

- validation;
- transaction completion;
- supporting service execution.

Clients receive standardized response models defined by the API contracts.

---

## 2.10 Design Principles

Every API implementation follows these principles:

- separation of concerns;
- backend authority;
- reusable services;
- repository abstraction;
- transactional integrity;
- centralized validation;
- consistent error handling;
- maintainable implementation.

# 3. Authentication Implementation

Authentication verifies user identity before allowing access to protected application resources.

The Authentication Service is responsible for validating credentials, generating authentication tokens, validating existing tokens, and providing authenticated user information.

Authentication is implemented as a reusable foundation shared by every protected API.

---

## 3.1 Authentication Flow

Every authentication request follows a consistent execution flow.

```text
Client Request

↓

Authentication Route

↓

Schema Validation

↓

Authentication Service

↓

User Repository

↓

Password Verification

↓

JWT Generation

↓

Response
```

For authenticated requests, the flow changes slightly.

```text
Client Request

↓

JWT Verification

↓

Authenticated User

↓

Protected Route

↓

Business Service

↓

Response
```

---

## 3.2 Login Implementation

The login process authenticates a user using their registered credentials.

Implementation flow:

```text
Login Request

↓

Validate Request Schema

↓

Retrieve User

↓

Verify Password

↓

Generate JWT

↓

Return User Information
```

The Authentication Service performs the following operations:

- validate request data;
- retrieve the user account;
- verify password hash;
- validate account eligibility;
- generate authentication token;
- construct authentication response.

Authentication succeeds only after every validation step passes successfully.

---

## 3.3 Token Validation

Protected endpoints require JWT authentication.

Every authenticated request performs the following sequence.

```text
HTTP Request

↓

Extract JWT

↓

Verify Signature

↓

Validate Expiration

↓

Retrieve User

↓

Attach User Context

↓

Continue Request
```

If token validation fails, request processing terminates immediately.

Business services are never executed for unauthenticated requests.

---

## 3.4 Authorization

Authentication confirms identity.

Authorization determines what the authenticated user may perform.

Authorization is evaluated before business processing begins.

Examples include:

- administrator-only endpoints;
- member-only operations;
- event ownership validation;
- administrative approval operations.

Authorization failures immediately terminate request processing.

---

## 3.5 User Context

After successful authentication, the authenticated user becomes part of the request context.

The user context includes information required by downstream services.

Examples include:

- user identifier;
- user role;
- authentication status.

Business services receive authenticated user information through dependency injection rather than directly interacting with authentication components.

---

## 3.6 Authentication Service Responsibilities

The Authentication Service is responsible for:

- credential verification;
- password validation;
- JWT generation;
- JWT validation;
- user retrieval;
- authentication response construction.

The Authentication Service is not responsible for authorization decisions outside authentication.

---

## 3.7 Repository Interaction

Authentication communicates with the persistence layer through the User Repository.

```text
Authentication Service

↓

User Repository

↓

Database
```

The repository performs:

- user lookup;
- account retrieval;
- credential retrieval.

Password verification remains within the Authentication Service.

---

## 3.8 Supporting Services

Authentication operations may generate supporting records where appropriate.

Examples include:

- activity history;
- audit log entries.

Successful authentication and significant security-related events may be recorded for operational visibility.

Notification generation is not required for normal authentication operations.

---

## 3.9 Error Handling

Authentication processing handles:

- invalid credentials;
- expired tokens;
- invalid tokens;
- unauthorized access;
- missing authentication.

Authentication failures produce standardized API responses.

Sensitive implementation details are never exposed.

---

## 3.10 Design Principles

Authentication implementation follows these principles:

- backend-controlled authentication;
- secure credential verification;
- stateless JWT authentication;
- centralized authentication logic;
- reusable authentication services;
- standardized authorization;
- secure error handling.

# 4. User Management Implementation

The User Management Implementation is responsible for retrieving and managing user information required throughout the attendance system.

User management provides identity information while preserving the separation between authentication and business operations.

Business services retrieve user information through the User Service.

---

## 4.1 User Management Flow

User-related requests follow a standardized execution flow.

```text
Client Request

↓

User Route

↓

Authentication

↓

Authorization

↓

User Service

↓

User Repository

↓

Database

↓

Response
```

Every request is authenticated before user information is processed.

---

## 4.2 User Retrieval

User retrieval provides the authenticated user's information or retrieves user records required by administrative operations.

Implementation flow:

```text
Authenticated Request

↓

User Service

↓

Business Validation

↓

User Repository

↓

Database

↓

Response
```

The User Service validates that the requested operation is permitted before retrieving user information.

---

## 4.3 User Validation

The User Service validates:

- user existence;
- account availability;
- authorization requirements;
- requested operation.

Business processing stops immediately if validation fails.

---

## 4.4 Repository Interaction

All user persistence operations are performed through the User Repository.

```text
User Service

↓

User Repository

↓

Database
```

The repository is responsible for:

- retrieving users;
- searching users;
- updating permitted user information;
- performing optimized user queries.

Business validation remains within the User Service.

---

## 4.5 Supporting Services

User management operations may generate:

- activity history;
- audit log entries;
- notifications where appropriate.

Supporting services execute independently after successful completion of the primary operation.

---

## 4.6 Transaction Management

Operations affecting multiple user-related entities are coordinated within a single transaction.

Simple retrieval operations do not require transactional coordination.

---

## 4.7 Error Handling

User management handles:

- user not found;
- unauthorized access;
- invalid requests;
- validation failures.

All errors follow the standardized API response format.

---

## 4.8 Design Principles

User management follows these principles:

- centralized user operations;
- repository abstraction;
- backend authority;
- reusable business services;
- standardized validation;
- consistent error handling.

# 5. Event Management Implementation

The Event Management Implementation is responsible for creating, managing, activating, updating, and completing events throughout their lifecycle.

The Event Service owns all event-related business rules and coordinates interactions with supporting services while maintaining the event as the central entity of the attendance system.

---

## 5.1 Event Processing Flow

Every event-related request follows a standardized implementation flow.

```text
Client Request

↓

Event Route

↓

Authentication

↓

Authorization

↓

Schema Validation

↓

Event Service

↓

Business Validation

↓

Event Repository

↓

Database

↓

Supporting Services

↓

Response
```

The Event Service acts as the central coordinator for all event operations.

---

## 5.2 Event Creation

Event creation initializes a new attendance event.

Implementation flow:

```text
Create Event Request

↓

Validate Request

↓

Validate Business Rules

↓

Create Event

↓

Persist Event

↓

Generate Activity

↓

Generate Audit Log

↓

Return Event
```

The Event Service performs the following responsibilities:

- validate event information;
- validate event schedule;
- validate boundary configuration;
- assign event ownership;
- initialize event status;
- persist the event.

The backend determines the initial lifecycle state.

---

## 5.3 Event Retrieval

Event retrieval provides event information for members and administrators.

Implementation flow:

```text
Retrieve Event

↓

Authentication

↓

Authorization

↓

Repository Query

↓

Construct Response
```

The Event Service determines:

- accessible events;
- event visibility;
- lifecycle information;
- participation eligibility.

Repository queries remain isolated from business rules.

---

## 5.4 Event Updates

Authorized administrators may update eligible events.

Implementation flow:

```text
Update Request

↓

Retrieve Event

↓

Validate Current State

↓

Validate Changes

↓

Persist Updates

↓

Supporting Services

↓

Response
```

The Event Service validates whether requested modifications are permitted based on the current event lifecycle.

Only valid updates are persisted.

---

## 5.5 Event Lifecycle Management

The backend controls every event lifecycle transition.

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

Lifecycle transitions are validated before persistence.

Invalid transitions are rejected before database updates occur.

---

## 5.6 Attendance Integration

The Event Service coordinates with the Attendance Service during participation workflows.

```text
Attendance Request

↓

Attendance Service

↓

Event Service

↓

Validate Event

↓

Continue Attendance
```

The Event Service determines:

- event availability;
- event status;
- participation eligibility.

Attendance processing continues only after successful validation.

---

## 5.7 Repository Interaction

The Event Service communicates exclusively through the Event Repository.

```text
Event Service

↓

Event Repository

↓

Database
```

The repository performs:

- event retrieval;
- event persistence;
- event updates;
- event search;
- lifecycle queries.

Business validation remains within the Event Service.

---

## 5.8 Supporting Services

Successful event operations may generate:

- notifications;
- activity history;
- audit log records.

Supporting services execute independently after successful completion of the primary operation.

---

## 5.9 Transaction Management

Operations affecting multiple entities are coordinated within a transaction.

Examples include:

- event creation;
- event activation;
- event completion;
- event cancellation.

The transaction is committed only after all required operations succeed.

---

## 5.10 Error Handling

Event processing handles:

- event not found;
- invalid lifecycle transition;
- invalid schedule;
- unauthorized operation;
- validation failures.

Errors are returned using standardized API responses.

---

## 5.11 Design Principles

Event implementation follows these principles:

- backend-controlled lifecycle;
- centralized business rules;
- repository abstraction;
- transactional consistency;
- reusable services;
- standardized validation.

# 6. Attendance Implementation

The Attendance Implementation manages participant attendance throughout an event.

The Attendance Service coordinates attendance creation, attendance completion, presence monitoring integration, emergency ticket integration, and attendance history.

Attendance serves as the primary business workflow within the application.

---

## 6.1 Attendance Processing Flow

Attendance requests follow a consistent execution pipeline.

```text
Client Request

↓

Attendance Route

↓

Authentication

↓

Authorization

↓

Schema Validation

↓

Attendance Service

↓

Business Validation

↓

Supporting Services

↓

Attendance Repository

↓

Database

↓

Response
```

Supporting services are invoked automatically where required.

---

## 6.2 Check-In Implementation

Check-in creates a new attendance record for an eligible participant.

Implementation flow:

```text
Check-In Request

↓

Validate User

↓

Validate Event

↓

Validate Eligibility

↓

Boundary Validation

↓

Create Attendance

↓

Initialize Presence

↓

Supporting Services

↓

Response
```

The Attendance Service validates:

- authenticated participant;
- event availability;
- attendance eligibility;
- duplicate attendance;
- boundary compliance.

Attendance is created only after all validation succeeds.

---

## 6.3 Active Attendance

An active attendance represents a participant currently attending an active event.

During active attendance, the Attendance Service coordinates with:

- Presence Service;
- Boundary Service;
- Emergency Ticket Service;
- Notification Service.

Attendance remains active until completion.

---

## 6.4 Check-Out Implementation

Check-out completes an active attendance record.

Implementation flow:

```text
Check-Out Request

↓

Retrieve Attendance

↓

Validate State

↓

Complete Attendance

↓

Update Database

↓

Supporting Services

↓

Response
```

The Attendance Service determines:

- attendance eligibility;
- completion state;
- attendance duration;
- final attendance information.

The backend calculates attendance information using server-controlled data.

---

## 6.5 Presence Integration

Active attendance continuously interacts with the Presence Service.

```text
Attendance Service

↓

Presence Service

↓

Boundary Service

↓

Presence Repository

↓

Database
```

The Attendance Service receives validated presence information from the Presence Service.

Attendance processing does not evaluate boundaries directly.

---

## 6.6 Emergency Ticket Integration

Emergency ticket workflows integrate with active attendance.

```text
Attendance Service

↓

Emergency Ticket Service

↓

Decision

↓

Attendance Update
```

Attendance changes resulting from emergency ticket decisions are coordinated through the Attendance Service.

---

## 6.7 Repository Interaction

Attendance persistence is performed exclusively through the Attendance Repository.

```text
Attendance Service

↓

Attendance Repository

↓

Database
```

Repository responsibilities include:

- attendance creation;
- attendance retrieval;
- attendance updates;
- attendance history queries.

Business validation remains within the Attendance Service.

---

## 6.8 Supporting Services

Attendance operations may automatically generate:

- notifications;
- activity history;
- audit logs.

Supporting services execute after successful attendance operations.

---

## 6.9 Transaction Management

Attendance operations commonly affect multiple entities.

Examples include:

- attendance record;
- presence record;
- notifications;
- activity history;
- audit logs.

These updates are coordinated within a single transaction where consistency requires it.

---

## 6.10 Error Handling

Attendance processing handles:

- duplicate attendance;
- invalid attendance state;
- event unavailable;
- attendance not found;
- participation restrictions;
- validation failures.

Errors follow standardized API response formats.

---

## 6.11 Design Principles

Attendance implementation follows these principles:

- backend-controlled attendance lifecycle;
- centralized business rules;
- reusable service coordination;
- repository abstraction;
- transactional consistency;
- standardized validation.

# 7. Presence Monitoring Implementation

The Presence Monitoring Implementation continuously tracks the location of participants after successful check-in.

The Presence Service is responsible for processing location updates, coordinating boundary validation, maintaining presence states, and recording presence history throughout an active attendance session.

Presence monitoring operates only while an attendance record remains active.

---

## 7.1 Presence Monitoring Flow

Every location update follows a standardized implementation flow.

```text
Location Update

↓

Presence Route

↓

Authentication

↓

Authorization

↓

Schema Validation

↓

Presence Service

↓

Boundary Service

↓

Business Validation

↓

Presence Repository

↓

Database

↓

Supporting Services

↓

Response
```

The Presence Service coordinates all location processing while delegating boundary evaluation to the Boundary Service.

---

## 7.2 Location Update Processing

Each submitted location update is processed independently.

Implementation flow:

```text
Receive Location

↓

Validate Attendance

↓

Validate Location Data

↓

Boundary Validation

↓

Determine Presence State

↓

Persist Presence Event

↓

Generate Supporting Records

↓

Response
```

The Presence Service validates:

- authenticated participant;
- active attendance;
- valid location payload;
- event availability.

Only valid location updates continue to boundary evaluation.

---

## 7.3 Presence State Management

The Presence Service determines the participant's current presence state.

```text
CHECK_IN

↓

PRESENCE_OK

↓

LEFT

↓

RETURNED

↓

CHECK_OUT
```

Presence state transitions are controlled exclusively by backend business logic.

Invalid transitions are rejected before persistence.

---

## 7.4 Presence History

Every significant presence transition is recorded.

Examples include:

- participant checked in;
- participant left the event boundary;
- participant returned to the event boundary;
- participant checked out.

Presence history provides a complete chronological record of attendance activity.

Historical records remain immutable after creation.

---

## 7.5 Attendance Integration

Presence monitoring operates as an extension of active attendance.

```text
Attendance Service

↓

Presence Service

↓

Boundary Service

↓

Presence Repository
```

The Attendance Service owns the attendance lifecycle.

The Presence Service owns location processing.

The Boundary Service owns boundary evaluation.

Each service maintains a single responsibility.

---

## 7.6 Repository Interaction

Presence data is persisted through the Presence Repository.

```text
Presence Service

↓

Presence Repository

↓

Database
```

Repository responsibilities include:

- recording presence events;
- retrieving presence history;
- retrieving current presence state;
- optimized chronological queries.

Business decisions remain within the Presence Service.

---

## 7.7 Supporting Services

Presence events may automatically generate:

- notifications;
- activity history;
- audit log entries.

Examples include:

- participant leaves the event boundary;
- participant returns to the event boundary;
- significant presence state changes.

Supporting services execute after successful presence processing.

---

## 7.8 Transaction Management

Location processing may update multiple entities.

Examples include:

- presence records;
- attendance status;
- notifications;
- activity history;
- audit logs.

Related updates are coordinated within a single transaction whenever consistency requires it.

---

## 7.9 Error Handling

Presence processing handles:

- attendance not found;
- inactive attendance;
- invalid location data;
- invalid presence state;
- validation failures.

Errors are returned using standardized API responses.

---

## 7.10 Design Principles

Presence Monitoring Implementation follows these principles:

- continuous backend-controlled monitoring;
- independent location processing;
- delegated boundary evaluation;
- immutable presence history;
- reusable service coordination;
- transactional consistency.

# 8. Boundary Validation Implementation

The Boundary Validation Implementation determines whether a participant's reported location satisfies the event's configured attendance boundary.

Boundary evaluation is centralized within the Boundary Service to ensure consistent validation across all attendance workflows.

The Boundary Service does not manage attendance or presence records directly.

Its sole responsibility is evaluating location data against the configured event boundary.

---

## 8.1 Boundary Validation Flow

Boundary validation follows a dedicated processing pipeline.

```text
Presence Service

↓

Boundary Service

↓

Retrieve Event Boundary

↓

Evaluate Location

↓

Validation Result

↓

Presence Service
```

The Boundary Service returns only the validation outcome.

Subsequent business actions are performed by the calling service.

---

## 8.2 Boundary Retrieval

Before evaluation begins, the Boundary Service retrieves the event's configured boundary.

Implementation flow:

```text
Event Identifier

↓

Boundary Service

↓

Event Repository

↓

Boundary Definition
```

The retrieved boundary becomes the reference for validation.

Boundary configuration is treated as read-only during validation.

---

## 8.3 Boundary Evaluation

The Boundary Service evaluates submitted location data against the configured event boundary.

Implementation flow:

```text
Location Data

↓

Boundary Service

↓

Boundary Evaluation

↓

Validation Result
```

The evaluation determines whether the submitted location satisfies the configured boundary requirements.

The implementation remains independent of the boundary representation.

---

## 8.4 Validation Results

Boundary evaluation returns one of two outcomes.

```text
Location

↓

Boundary Evaluation

↓

Inside Boundary

or

Outside Boundary
```

The Boundary Service does not determine attendance status or presence state.

It returns only the validation result.

---

## 8.5 Presence Integration

The Presence Service consumes boundary evaluation results.

```text
Presence Service

↓

Boundary Service

↓

Validation Result

↓

Presence Decision
```

Presence state changes remain the responsibility of the Presence Service.

Boundary evaluation remains isolated within the Boundary Service.

---

## 8.6 Event Integration

The Event Service provides boundary configuration during attendance operations.

```text
Attendance Service

↓

Event Service

↓

Boundary Service
```

Boundary definitions are managed through event administration.

Boundary validation consumes those definitions without modifying them.

---

## 8.7 Repository Interaction

The Boundary Service retrieves required boundary information through repository abstractions.

```text
Boundary Service

↓

Event Repository

↓

Database
```

The repository provides:

- boundary configuration;
- event boundary retrieval;
- boundary metadata.

Business evaluation remains within the Boundary Service.

---

## 8.8 Transaction Management

Boundary evaluation itself does not modify persistent data.

Transactions are managed by the calling business service when subsequent updates become necessary.

This separation keeps boundary validation lightweight and reusable.

---

## 8.9 Error Handling

Boundary validation handles:

- event not found;
- missing boundary definition;
- invalid location data;
- malformed boundary configuration.

Errors are propagated to the calling service using standardized application exceptions.

---

## 8.10 Design Principles

Boundary Validation Implementation follows these principles:

- centralized boundary evaluation;
- reusable validation logic;
- backend-controlled decisions;
- separation from attendance management;
- repository abstraction;
- implementation independence from boundary representation.

# 9. Emergency Ticket Implementation

The Emergency Ticket Implementation manages temporary leave requests submitted by participants during an active attendance.

The Emergency Ticket Service coordinates request creation, administrative review, approval decisions, attendance updates, and historical record generation while maintaining a controlled ticket lifecycle.

Emergency ticket processing is initiated only for participants with an active attendance.

---

## 9.1 Emergency Ticket Processing Flow

Every emergency ticket request follows a standardized implementation flow.

```text
Client Request

↓

Emergency Ticket Route

↓

Authentication

↓

Authorization

↓

Schema Validation

↓

Emergency Ticket Service

↓

Business Validation

↓

Emergency Ticket Repository

↓

Database

↓

Supporting Services

↓

Response
```

The Emergency Ticket Service coordinates all ticket-related business operations.

---

## 9.2 Ticket Submission

Participants may submit an emergency ticket while actively attending an event.

Implementation flow:

```text
Submit Ticket

↓

Validate Attendance

↓

Validate Request

↓

Create Ticket

↓

Persist Ticket

↓

Generate Notifications

↓

Generate Activity

↓

Generate Audit Log

↓

Response
```

The Emergency Ticket Service validates:

- authenticated participant;
- active attendance;
- request eligibility;
- duplicate pending requests;
- request content.

Only valid requests are persisted.

---

## 9.3 Administrative Review

Administrators review submitted emergency tickets.

Implementation flow:

```text
Administrator Action

↓

Retrieve Ticket

↓

Validate Status

↓

Review Request

↓

Decision

↓

Persist Decision

↓

Supporting Services

↓

Response
```

The service validates:

- ticket existence;
- ticket ownership where applicable;
- administrative authorization;
- current ticket status.

Only pending tickets may be reviewed.

---

## 9.4 Ticket Lifecycle

Emergency tickets follow a controlled lifecycle.

```text
PENDING

├── APPROVED

├── REJECTED

└── CANCELLED
```

Lifecycle transitions are validated before persistence.

Completed decisions become permanent historical records.

---

## 9.5 Attendance Integration

Emergency ticket decisions integrate with attendance processing.

```text
Emergency Ticket Service

↓

Attendance Service

↓

Attendance Update

↓

Database
```

The Emergency Ticket Service determines the review outcome.

The Attendance Service updates attendance information where required.

Each service maintains its own responsibility.

---

## 9.6 Repository Interaction

Emergency ticket persistence is performed exclusively through the Emergency Ticket Repository.

```text
Emergency Ticket Service

↓

Emergency Ticket Repository

↓

Database
```

Repository responsibilities include:

- ticket creation;
- ticket retrieval;
- ticket updates;
- ticket history queries.

Business validation remains within the Emergency Ticket Service.

---

## 9.7 Supporting Services

Emergency ticket operations automatically generate supporting records.

These may include:

- notifications;
- activity history;
- audit logs.

Supporting services execute after successful ticket processing.

---

## 9.8 Transaction Management

Emergency ticket operations may update multiple entities.

Examples include:

- emergency ticket;
- attendance record;
- notifications;
- activity history;
- audit logs.

These updates are coordinated within a single transaction whenever consistency requires it.

---

## 9.9 Error Handling

Emergency ticket processing handles:

- attendance not found;
- ticket not found;
- duplicate pending requests;
- invalid ticket state;
- unauthorized review;
- validation failures.

Errors follow the standardized API response format.

---

## 9.10 Design Principles

Emergency Ticket Implementation follows these principles:

- centralized approval workflow;
- backend-controlled decisions;
- immutable historical records;
- reusable business services;
- transactional consistency;
- standardized validation.

# 10. Volunteer Block Implementation

The Volunteer Block Implementation manages temporary participation restrictions applied to volunteers.

The Volunteer Block Service determines participation eligibility before attendance processing begins and maintains historical records of administrative decisions.

Volunteer blocks are enforced entirely by the backend.

---

## 10.1 Volunteer Block Processing Flow

Volunteer block operations follow a standardized implementation flow.

```text
Administrator Request

↓

Volunteer Block Route

↓

Authentication

↓

Authorization

↓

Schema Validation

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

↓

Response
```

The Volunteer Block Service owns all volunteer block business rules.

---

## 10.2 Block Creation

Administrators create volunteer blocks for eligible participants.

Implementation flow:

```text
Create Block

↓

Validate Administrator

↓

Validate Volunteer

↓

Validate Existing Blocks

↓

Create Block

↓

Persist Block

↓

Supporting Services

↓

Response
```

The Volunteer Block Service validates:

- administrator authorization;
- volunteer existence;
- duplicate active blocks;
- business eligibility.

Only valid blocks are persisted.

---

## 10.3 Eligibility Evaluation

Volunteer eligibility is evaluated before attendance creation.

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

The Volunteer Block Service returns only the eligibility decision.

Attendance processing consumes the result.

---

## 10.4 Block Lifecycle

Volunteer blocks follow a simple lifecycle.

```text
ACTIVE

↓

REMOVED
```

Historical block records remain permanently available after removal.

---

## 10.5 Attendance Integration

Attendance processing depends on volunteer eligibility.

```text
Attendance Service

↓

Volunteer Block Service

↓

Eligibility Result

↓

Attendance Processing
```

The Attendance Service does not evaluate volunteer blocks directly.

Eligibility remains centralized within the Volunteer Block Service.

---

## 10.6 Repository Interaction

Volunteer block persistence is handled through the Volunteer Block Repository.

```text
Volunteer Block Service

↓

Volunteer Block Repository

↓

Database
```

Repository responsibilities include:

- block creation;
- block retrieval;
- active block queries;
- block history.

Business rules remain within the Volunteer Block Service.

---

## 10.7 Supporting Services

Volunteer block operations may generate:

- notifications;
- activity history;
- audit logs.

Supporting services execute independently after successful completion of the primary operation.

---

## 10.8 Transaction Management

Volunteer block operations may affect multiple entities.

Examples include:

- volunteer block records;
- notifications;
- activity history;
- audit logs.

Related updates are coordinated within a single transaction.

---

## 10.9 Error Handling

Volunteer block processing handles:

- volunteer not found;
- duplicate active block;
- unauthorized operation;
- invalid block state;
- validation failures.

Errors are returned using standardized API responses.

---

## 10.10 Design Principles

Volunteer Block Implementation follows these principles:

- centralized eligibility management;
- backend-controlled enforcement;
- reusable service coordination;
- repository abstraction;
- transactional consistency;
- standardized validation.

# 11. Notification Implementation

The Notification Implementation provides a centralized mechanism for informing users about significant business events.

Business services request notification generation without managing notification persistence directly.

The Notification Service owns notification creation, retrieval, and status management.

---

## 11.1 Notification Processing Flow

Notification generation follows a common implementation pattern.

```text
Business Service

↓

Notification Service

↓

Notification Repository

↓

Database

↓

Response
```

Business services remain independent of notification storage.

---

## 11.2 Notification Generation

Notifications are created after successful completion of business operations.

Examples include:

- attendance completed;
- participant left the event boundary;
- participant returned to the event boundary;
- emergency ticket approved;
- emergency ticket rejected;
- volunteer block created;
- volunteer block removed.

Notification generation occurs after the primary business operation succeeds.

---

## 11.3 Notification Retrieval

Users retrieve notifications through dedicated notification endpoints.

Implementation flow:

```text
Retrieve Notifications

↓

Notification Service

↓

Notification Repository

↓

Database

↓

Response
```

The Notification Service supports:

- notification retrieval;
- unread notification queries;
- notification status updates.

---

## 11.4 Repository Interaction

Notification persistence is handled through the Notification Repository.

Repository responsibilities include:

- notification creation;
- notification retrieval;
- notification updates;
- notification history queries.

Business services never interact with notification storage directly.

---

## 11.5 Transaction Management

Notification generation participates in the parent business transaction when consistency requires it.

Notification failures should not leave partially completed business operations.

---

## 11.6 Error Handling

Notification processing handles:

- notification not found;
- unauthorized access;
- invalid notification state.

Errors are returned using standardized application responses.

---

## 11.7 Design Principles

Notification Implementation follows these principles:

- centralized notification management;
- reusable notification generation;
- repository abstraction;
- standardized processing;
- independent service coordination.

# 12. Activity History Implementation

The Activity History Implementation records significant business events performed throughout the attendance lifecycle.

Activity history provides a chronological timeline of user actions while remaining separate from administrative audit logging.

The Activity History Service automatically records business events without requiring additional client interaction.

---

## 12.1 Activity Recording Flow

Activity recording follows a centralized implementation flow.

```text
Business Service

↓

Activity History Service

↓

Activity History Repository

↓

Database

↓

Response
```

Business services request activity recording after successful completion of a business operation.

The Activity History Service owns activity persistence.

---

## 12.2 Activity Generation

Activity records are generated automatically.

Examples include:

- attendance check-in;
- attendance check-out;
- participant left the event boundary;
- participant returned to the event boundary;
- emergency ticket submitted;
- emergency ticket approved;
- emergency ticket rejected;
- volunteer block removed.

Activities are generated only after successful completion of the associated business operation.

---

## 12.3 Timeline Construction

Activity history provides a chronological view of completed business events.

Implementation flow:

```text
Retrieve Timeline

↓

Activity History Service

↓

Activity History Repository

↓

Database

↓

Chronological Timeline

↓

Response
```

Timeline ordering is determined using server-generated timestamps.

Historical ordering remains consistent across all business modules.

---

## 12.4 Repository Interaction

Activity persistence is handled exclusively through the Activity History Repository.

```text
Activity History Service

↓

Activity History Repository

↓

Database
```

Repository responsibilities include:

- activity creation;
- activity retrieval;
- timeline queries;
- chronological sorting.

Business services remain independent of persistence logic.

---

## 12.5 Transaction Management

Activity history generation participates in the parent business transaction whenever consistency requires it.

Activity records should never exist for business operations that fail.

---

## 12.6 Error Handling

Activity history processing handles:

- invalid user;
- activity not found;
- unauthorized access;
- invalid query parameters.

Errors are returned using standardized API responses.

---

## 12.7 Design Principles

Activity History Implementation follows these principles:

- centralized activity generation;
- immutable historical records;
- chronological consistency;
- repository abstraction;
- reusable service integration.

# 13. Audit Logging Implementation

The Audit Logging Implementation records administrative and security-sensitive operations performed within the backend.

Audit logs provide accountability, operational traceability, and administrative oversight while remaining separate from user-facing activity history.

The Audit Log Service automatically records qualifying operations.

---

## 13.1 Audit Logging Flow

Audit logging follows a standardized implementation flow.

```text
Business Service

↓

Audit Log Service

↓

Audit Log Repository

↓

Database

↓

Response
```

Business services initiate audit generation after successful completion of qualifying operations.

---

## 13.2 Audit Generation

Audit records are generated automatically for significant administrative operations.

Examples include:

- administrator authentication;
- event creation;
- event updates;
- event cancellation;
- emergency ticket review;
- volunteer block creation;
- volunteer block removal.

Audit generation occurs after successful completion of the primary business operation.

---

## 13.3 Administrative Review

Audit records provide historical visibility into administrative operations.

Implementation flow:

```text
Administrative Request

↓

Audit Log Service

↓

Audit Log Repository

↓

Database

↓

Response
```

Only authorized administrators may retrieve audit information.

Authorization is validated before retrieval.

---

## 13.4 Repository Interaction

Audit persistence is performed through the Audit Log Repository.

```text
Audit Log Service

↓

Audit Log Repository

↓

Database
```

Repository responsibilities include:

- audit creation;
- audit retrieval;
- audit history queries;
- chronological queries.

Business services remain independent of persistence implementation.

---

## 13.5 Transaction Management

Audit records participate in the parent business transaction when consistency requires it.

Administrative history should accurately reflect completed operations.

---

## 13.6 Error Handling

Audit processing handles:

- unauthorized access;
- audit record not found;
- invalid query parameters.

Errors are translated into standardized API responses.

---

## 13.7 Design Principles

Audit Logging Implementation follows these principles:

- centralized audit generation;
- immutable administrative history;
- secure administrative access;
- repository abstraction;
- reusable service integration.

# 14. Report Implementation

The Report Implementation generates attendance reports for administrators.

Reports provide summarized attendance information while remaining independent from operational business workflows.

The Report Service coordinates data retrieval, report generation, and response construction.

---

## 14.1 Report Generation Flow

Report generation follows a standardized implementation flow.

```text
Administrator Request

↓

Report Route

↓

Authentication

↓

Authorization

↓

Report Service

↓

Repository Queries

↓

Generate Report

↓

Response
```

The Report Service coordinates report generation without modifying business data.

---

## 14.2 Attendance Report Generation

Attendance reports summarize participation for completed or active events.

Implementation flow:

```text
Generate Report

↓

Retrieve Event

↓

Retrieve Attendance

↓

Validate Data

↓

Construct Report

↓

Return Report
```

The Report Service determines:

- report eligibility;
- report completeness;
- report structure.

Business data remains unchanged during report generation.

---

## 14.3 Repository Interaction

Report generation may retrieve information from multiple repositories.

Examples include:

- Event Repository;
- Attendance Repository;
- User Repository.

Repositories remain responsible only for data retrieval.

The Report Service performs report construction.

---

## 14.4 Response Construction

Reports may be returned using application-supported response formats.

The implementation remains independent of the output representation.

Report generation focuses on assembling validated business information rather than presentation details.

---

## 14.5 Performance Considerations

Report generation should:

- minimize unnecessary queries;
- retrieve only required data;
- support efficient large-event processing;
- avoid impacting operational workloads.

Where appropriate, report generation should use optimized repository queries.

---

## 14.6 Error Handling

Report processing handles:

- event not found;
- unauthorized access;
- insufficient report data;
- invalid request parameters.

Errors follow standardized API response formats.

---

## 14.7 Design Principles

Report Implementation follows these principles:

- read-only report generation;
- repository abstraction;
- centralized report construction;
- reusable reporting logic;
- optimized data retrieval.

# 15. Repository Usage

Repositories provide the persistence layer for the backend implementation.

They encapsulate all database interactions, allowing business services to remain independent of the underlying database technology.

Every repository is responsible only for data access.

Business rules remain within the Service Layer.

---

## 15.1 Repository Responsibilities

Repositories perform operations such as:

- retrieving records;
- creating records;
- updating records;
- deleting records where permitted;
- executing optimized queries.

Repositories must not contain business validation.

---

## 15.2 Repository Interaction Flow

Every persistence operation follows the same interaction pattern.

```text
Route

↓

Service

↓

Repository

↓

Database

↓

Repository

↓

Service

↓

Response
```

Business services coordinate repositories without exposing database implementation details.

---

## 15.3 Repository Organization

The backend maintains separate repositories for each business domain.

Examples include:

- User Repository;
- Event Repository;
- Attendance Repository;
- Presence Repository;
- Emergency Ticket Repository;
- Volunteer Block Repository;
- Notification Repository;
- Activity History Repository;
- Audit Log Repository;
- Report Repository.

Each repository manages only its own aggregate.

---

## 15.4 Query Optimization

Repositories are responsible for:

- efficient filtering;
- indexed lookups;
- pagination support;
- optimized joins where required.

Performance improvements should remain transparent to business services.

---

## 15.5 Repository Design Principles

Repository implementation follows these principles:

- single responsibility;
- database abstraction;
- reusable queries;
- optimized persistence;
- maintainable implementation.

# 16. Transaction Strategy

Transactions ensure that related database operations are committed as a single logical unit.

The Service Layer coordinates transactions while repositories remain focused on persistence.

---

## 16.1 Transaction Flow

```text
Business Operation

↓

Begin Transaction

↓

Repository Operations

↓

Supporting Services

↓

Commit

or

Rollback
```

The transaction completes only after all required operations succeed.

---

## 16.2 Coordinated Operations

Examples of transactional operations include:

- event creation;
- attendance check-in;
- attendance check-out;
- emergency ticket approval;
- volunteer block creation;
- event completion.

Each operation may involve multiple repositories.

---

## 16.3 Rollback Strategy

If any required operation fails:

- pending database changes are rolled back;
- data consistency is preserved;
- no partial updates remain.

Rollback protects the integrity of business workflows.

---

## 16.4 Transaction Scope

Transactions should:

- remain as short as practical;
- update only required entities;
- avoid unnecessary database locks;
- preserve consistency.

---

## 16.5 Design Principles

Transaction management follows these principles:

- atomic operations;
- consistent persistence;
- isolated execution;
- durable commits;
- predictable rollback behavior.

# 17. Validation Strategy

Validation ensures that only legitimate requests reach the business layer.

Validation occurs progressively throughout request processing.

---

## 17.1 Validation Pipeline

Every request follows the same validation sequence.

```text
Request

↓

Schema Validation

↓

Authentication

↓

Authorization

↓

Business Validation

↓

Repository Operations

↓

Database Constraints
```

Each validation layer protects a different aspect of the application.

---

## 17.2 Schema Validation

Schema validation verifies:

- request structure;
- required fields;
- supported data types;
- field constraints.

Invalid requests are rejected before business processing begins.

---

## 17.3 Business Validation

Business validation is performed within the Service Layer.

Examples include:

- attendance eligibility;
- event lifecycle validation;
- volunteer participation rules;
- emergency ticket eligibility;
- boundary compliance.

Business validation determines whether the requested operation is permitted.

---

## 17.4 Database Validation

Database constraints provide the final layer of protection.

Examples include:

- primary keys;
- foreign keys;
- unique constraints;
- check constraints.

Application validation complements database validation.

---

## 17.5 Validation Principles

Validation follows these principles:

- layered validation;
- backend authority;
- deterministic processing;
- reusable validation logic;
- consistent enforcement.

# 18. Error Handling

The backend provides centralized error handling to ensure consistent responses across every API.

Errors are translated into standardized responses before being returned to clients.

---

## 18.1 Error Flow

```text
Application Error

↓

Exception Handler

↓

Standardized Response

↓

Client
```

Internal implementation details remain hidden from clients.

---

## 18.2 Error Categories

Typical error categories include:

- validation failures;
- authentication failures;
- authorization failures;
- business rule violations;
- resource not found;
- conflict conditions;
- unexpected server errors.

Each category maps to standardized application responses.

---

## 18.3 Service-Level Errors

Business services raise application-specific exceptions.

Route handlers do not implement business-specific error handling.

Centralized exception processing converts service exceptions into API responses.

---

## 18.4 Logging

Unexpected failures should generate operational log entries.

Application logs support:

- troubleshooting;
- monitoring;
- operational diagnostics.

Sensitive information must never appear in logs.

---

## 18.5 Design Principles

Error handling follows these principles:

- centralized exception handling;
- consistent responses;
- secure error reporting;
- meaningful client feedback;
- operational visibility.

# 19. Testing Strategy

The backend implementation is designed to support comprehensive automated testing.

Testing verifies business correctness while ensuring long-term maintainability.

---

## 19.1 Testing Levels

Testing includes:

- unit testing;
- integration testing;
- API testing;
- repository testing;
- business workflow testing.

Each level validates a different aspect of the implementation.

---

## 19.2 Unit Testing

Unit tests verify individual services independently.

Examples include:

- Event Service;
- Attendance Service;
- Presence Service;
- Boundary Service;
- Emergency Ticket Service;
- Volunteer Block Service.

Dependencies should be isolated where appropriate.

---

## 19.3 Integration Testing

Integration tests verify collaboration between:

- services;
- repositories;
- database;
- supporting services.

Integration testing validates complete business workflows.

---

## 19.4 API Testing

API testing verifies:

- endpoint behavior;
- authentication;
- authorization;
- request validation;
- response models;
- error responses.

API tests ensure compliance with the published API contracts.

---

## 19.5 Workflow Testing

End-to-end business workflows should be validated.

Examples include:

- event lifecycle;
- attendance lifecycle;
- presence monitoring;
- emergency ticket approval;
- volunteer block enforcement;
- notification generation.

Testing should verify both successful and failure scenarios.

---

## 19.6 Design Principles

Testing follows these principles:

- repeatable execution;
- isolated tests;
- predictable outcomes;
- comprehensive workflow coverage;
- maintainable test suites.

# 20. Conclusion

The Backend API Implementation establishes a consistent approach for implementing every API within the InnoTech Hub Attendance System.

By separating HTTP communication, business logic, persistence, validation, and supporting services, the implementation remains modular, maintainable, and scalable.

The implementation emphasizes:

- thin route handlers;
- centralized business services;
- repository-based persistence;
- backend-controlled business decisions;
- transactional consistency;
- standardized validation;
- centralized error handling;
- comprehensive historical record keeping.

This implementation guide complements the API contracts, backend architecture, database schema, and business rules by describing how requests move through the backend while preserving clear separation of responsibilities.

Following these implementation principles ensures that future development remains consistent with the project's architectural standards, business requirements, and long-term maintainability goals.

# 21. Phase 3 Activity Layer Implementation

Phase 3 extends the backend by introducing the Activity Layer.

The Activity Layer allows administrators to create activities, assign volunteers, collect evidence for assigned work, review completed assignments, and reuse activity templates.

The implementation follows the same layered architecture used throughout the attendance system.

The backend remains the single source of truth for all activity-related business decisions.

---

## 21.1 Activity Processing Flow

Every activity request follows the standard backend pipeline.

```text
Client Request

↓

Activity Route

↓

Authentication

↓

Authorization

↓

Schema Validation

↓

Activity Service

↓

Business Validation

↓

Repository Layer

↓

Database

↓

Supporting Services

↓

Response
```

Supporting services include:

- Notification Service
- Activity History Service
- Audit Log Service

---

## 21.2 Activity Creation

Activity creation is performed only by administrators.

Implementation flow:

```text
Create Activity

↓

Validate Administrator

↓

Validate Event

↓

Validate Activity Data

↓

Create Activity

↓

Persist Activity

↓

Supporting Services

↓

Response
```

The Activity Service validates:

- administrator permissions;
- event existence;
- activity information;
- activity category;
- activity priority.

Activities are created in the **Draft** state.

---

## 21.3 Activity Publication

Only Draft activities may be published.

Implementation flow:

```text
Retrieve Activity

↓

Validate Current State

↓

Publish Activity

↓

Persist Changes

↓

Response
```

Only published activities may receive volunteer assignments.

---

## 21.4 Activity Assignment

Administrators assign one or more volunteers to a published activity.

Implementation flow:

```text
Retrieve Activity

↓

Validate Activity

↓

Validate Volunteer

↓

Check Duplicate Assignment

↓

Create Assignment

↓

Supporting Services

↓

Response
```

Assignment validation prevents duplicate assignments for the same volunteer.

---

## 21.6 Evidence Processing

Evidence is attached directly to activity assignments.

Implementation flow:

```text
Upload Evidence

↓

Validate Assignment

↓

Validate Submission Limits

↓

Optimize File

↓

Store File

↓

Persist Metadata

↓

Response
```

The backend validates:

- supported file type;
- maximum photo limit;
- maximum video limit;
- maximum video duration.

Only metadata is stored in the database.

---

## 21.7 Activity Submission

After completing the activity, volunteers submit it for administrator review.

Implementation flow:

```text
Validate Assignment

↓

Validate Evidence

↓

Update Assignment Status

↓

Notify Administrator

↓

Response
```

Once submitted:

- the assignment becomes read-only until review completion or a "Needs Changes" decision;
- uploaded evidence becomes immutable.

---

## 21.8 Review Processing

Administrators review submitted assignments.

Implementation flow:

```text
Retrieve Submission

↓

Review Evidence

↓

Decision

↓

Persist Review

↓

Supporting Services

↓

Response
```

Review decisions:

- VERIFIED
- NEEDS_CHANGES

Every review creates a new review history record.

---

## 21.9 Template Processing

Templates simplify recurring activity creation.

Implementation flow:

```text
Select Template

↓

Retrieve Template

↓

Generate Activities

↓

Persist Activities

↓

Response
```

Generated activities become independent records.

Updating a template does not modify previously generated activities.

---

## 21.10 Transaction Management

The following operations execute within a single transaction:

- activity creation;
- volunteer assignment;
- activity submission;
- review completion;
- template application.

Rollback occurs if any required operation fails.

---

## 21.11 Error Handling

Activity processing handles:

- activity not found;
- invalid activity state;
- duplicate assignment;
- assignment not found;
- evidence limit exceeded;
- invalid review state;
- template not found.

Errors follow the standardized API response model.

---

## 21.12 Design Principles

The Activity Layer follows these principles:

- backend-controlled business decisions;
- centralized business logic;
- evidence belongs to activity assignments;
- immutable review history;
- reusable templates;
- transactional consistency;
- repository abstraction;
- standardized validation;
- thin route handlers.

# 22. Phase 4 Production Hardening Implementation

Phase 4 extends the backend implementation by introducing production hardening capabilities while preserving the implementation architecture established during Phases 1–3.

Unlike previous phases, Phase 4 focuses on operational reliability rather than introducing new business functionality.

The backend remains the authoritative source of truth for every operation.

---

## 22.1 Production Hardening Flow

Production hardening extends the existing request pipeline.

```text
Client Request

↓

Authentication

↓

Authorization

↓

Schema Validation

↓

Spoofing Detection

↓

Business Validation

↓

Business Service

↓

Repository Layer

↓

Database

↓

Immediate Background Processing

↓

Response
```

Immediate Background Processing may include:

- audit logging;
- notification generation;
- synchronization result generation;
- security event recording.

The request lifecycle remains consistent across all APIs.

---

## 22.2 Offline Synchronization Implementation

Offline operations are synchronized through a dedicated Synchronization Service.

The frontend maintains the offline queue using IndexedDB.

When connectivity is restored, the frontend submits the entire synchronization queue in a single request.

Implementation flow:

```text
Synchronization Request

↓

Synchronization Service

↓

Validate Queue

↓

Replay Operations

↓

Business Services

↓

Database

↓

Synchronization Result

↓

Response
```

Each synchronized operation follows the same implementation pipeline as an online request.

Business rules are never duplicated for synchronization processing.

---

## 22.3 Synchronization Validation

Before replaying queued operations, the backend validates:

- request integrity;
- operation ordering where required;
- authenticated user;
- authorization;
- payload validity.

Invalid operations are rejected individually while allowing valid operations to continue where appropriate.

The backend determines the final synchronization outcome.

---

## 22.4 Spoofing Detection Implementation

Spoofing detection occurs before business validation.

Implementation flow:

```text
Incoming Request

↓

Spoofing Detection Service

↓

Evaluate Device

↓

Evaluate Location

↓

Generate Assessment

↓

Continue Processing
```

The service evaluates:

- mock GPS;
- impossible travel;
- abnormal movement;
- emulator detection where supported;
- device time manipulation;
- rooted or jailbroken devices where supported.

The service returns a detailed assessment object describing all detected indicators.

Detection itself does not perform enforcement.

Administrative decisions remain separate from detection.

---

## 22.5 Audit Logging Implementation

Audit logging is coordinated through the Immediate Background Processing layer.

Implementation flow:

```text
Successful Business Operation

↓

Immediate Background Processing

↓

AuditLogService

↓

Audit Repository

↓

Database
```

Only business-significant operations generate audit records.

Audit records remain immutable after creation.

Business responses should not be delayed by audit persistence.

---

## 22.6 Backup Implementation

The Backup Service supports both scheduled and administrator-initiated backups.

Manual backup flow:

```text
Administrator Request

↓

Authentication

↓

Authorization

↓

Backup Service

↓

Generate Backup

↓

Verify Backup

↓

Persist Metadata

↓

Response
```

Scheduled backup flow:

```text
Scheduler

↓

Backup Service

↓

Generate Backup

↓

Verify Backup

↓

Persist Metadata
```

Backup files remain external to PostgreSQL.

Only backup metadata is stored within the application database.

---

## 22.7 Background Processing Implementation

Background processing is divided into two independent execution paths.

### Immediate Background Processing

Executed immediately after successful business operations.

Responsibilities include:

- audit logging;
- notification generation;
- synchronization completion;
- security event recording.

---

### Scheduled Background Processing

Executed independently of user requests.

Responsibilities include:

- scheduled backups;
- backup verification;
- audit retention cleanup;
- operational maintenance;
- future scheduled jobs.

The two execution paths remain isolated from each other.

---

## 22.8 Service Collaboration

Production hardening services collaborate with existing business services while preserving service ownership.

Example attendance workflow:

```text
Attendance Service

↓

Presence Service

↓

Spoofing Detection Service

↓

Repository

↓

Immediate Background Processing

↓

Response
```

Synchronization workflow:

```text
Synchronization Service

↓

Attendance Service

↓

Presence Service

↓

Activity Service

↓

Synchronization Result
```

Business services remain responsible for business rules.

Production services provide operational capabilities.

---

## 22.9 Error Handling

Production hardening introduces additional standardized errors.

Examples include:

```python
SYNC_CONFLICT

INVALID_SYNC_QUEUE

SPOOFING_DETECTED

BACKUP_FAILED

BACKUP_VERIFICATION_FAILED
```

These errors follow the same standardized response model used throughout the backend.

---

## 22.10 Implementation Principles

Phase 4 implementation follows these principles:

- preserve existing implementation architecture;
- backend remains authoritative;
- synchronization reuses existing business services;
- spoofing detection precedes business processing;
- audit logging executes asynchronously;
- backups support both scheduled and manual execution;
- immediate and scheduled background processing remain isolated;
- business rules remain centralized;
- implementation remains independently testable.

---

## 22.11 Phase 4 Summary

Phase 4 extends the backend implementation with production hardening capabilities while preserving the implementation patterns established throughout previous phases.

The resulting implementation improves reliability, synchronization, security monitoring, auditability, recoverability, and operational readiness without changing existing API contracts or business workflows.