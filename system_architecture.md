# system_architecture.md — Technical Architecture (Phase 1)

## 1. Stack decision

| Layer | Choice | Why |
|---|---|---|
| Frontend | **React + TypeScript + Vite**, styled with **Tailwind CSS** + **shadcn/ui** (Radix primitives), motion via **Framer Motion** | Chosen for a rich, premium-feeling UI (per project decision) with a huge learning-resource base — good fit for a first-time frontend developer working with an AI agent. Flutter is deferred to a later phase. |
| Backend | **FastAPI** (Python) | Matches the original pitch/architecture; async-friendly, auto-generates OpenAPI docs which doubles as a live version of `API_contract.md`. |
| Database | **PostgreSQL** | Project decision (overrides the pitch's Excel-only suggestion) — a real relational database is easier for two beginners to reason about than direct spreadsheet I/O, and it still supports Excel export as a report format rather than as the system of record. |
| ORM / migrations | **SQLAlchemy** + **Alembic** | Standard pairing with FastAPI; migrations give a reviewable history of schema changes. |
| Auth | **JWT** (access token), passwords hashed with **bcrypt/argon2** | Simple, stateless, well-documented for beginners. |
| Reporting | **openpyxl** | Generates the Excel export from PostgreSQL query results on demand. |
| Dev environment | Local only (see `deployment_guide.md`) | Per project decision — cloud deployment is future work. |

This stack fulfills the same architectural intent as the pitch deck's "Flutter + FastAPI + Excel"
diagram (slide 7), with Flutter replaced by a web frontend for now and Excel demoted from storage
to export format, and PostgreSQL added as the system of record.

## 2. High-level component diagram

```
┌─────────────────────────┐        HTTPS/JSON        ┌──────────────────────────┐
│   React + TS Frontend   │ ───────────────────────▶ │      FastAPI Backend     │
│  (Vite, Tailwind,       │ ◀─────────────────────── │  (business rules, auth,  │
│   shadcn/ui)            │        JWT-authed         │   validation)            │
└─────────────────────────┘                           └───────────┬──────────────┘
                                                                    │ SQL (SQLAlchemy)
                                                                    ▼
                                                        ┌──────────────────────────┐
                                                        │       PostgreSQL         │
                                                        │  (system of record)      │
                                                        └───────────┬──────────────┘
                                                                    │ on-demand export
                                                                    ▼
                                                        ┌──────────────────────────┐
                                                        │  openpyxl .xlsx report   │
                                                        └──────────────────────────┘
```

Device camera and GPS are accessed via standard browser APIs (`navigator.geolocation`,
`getUserMedia`) from the frontend — no native mobile code in Phase 1.

## 3. Data model (Phase 1 entities)

Scoped down from the full entity list in Working Book §12.2. Only what Phase 1 needs:

### `users`
| Column | Type | Notes |
|---|---|---|
| id | UUID (PK) | |
| full_name | text | |
| email | text, unique | login identifier |
| password_hash | text | never expose via API |
| role | enum(`admin`, `member`) | Phase 1 has only these two |
| created_at | timestamptz | |

### `sessions`
| Column | Type | Notes |
|---|---|---|
| id | UUID (PK) | |
| title | text | |
| purpose | text, nullable | |
| created_by | UUID (FK → users.id) | must be an admin |
| date | date | |
| start_time | timestamptz | |
| end_time | timestamptz | |
| grace_period_minutes | int | default e.g. 10 |
| venue_lat | double precision | |
| venue_lng | double precision | |
| radius_meters | int | |
| status | enum(`scheduled`, `open`, `closed`) | simplified from full state model |
| created_at | timestamptz | |

### `attendance_records`
| Column | Type | Notes |
|---|---|---|
| id | UUID (PK) | |
| session_id | UUID (FK → sessions.id) | |
| member_id | UUID (FK → users.id) | |
| check_in_time | timestamptz | server time, not client-submitted |
| check_in_lat | double precision | |
| check_in_lng | double precision | |
| distance_meters | double precision | computed server-side (Haversine) |
| gps_accuracy_meters | double precision, nullable | |
| photo_reference | text | storage path/key, see §5 |
| final_status | enum(`present`, `late`, `pending_verification`, `rejected`) | |
| rejection_reason | text, nullable | |
| created_at | timestamptz | |

**Unique constraint** on (`session_id`, `member_id`) to enforce FR-14 (no duplicate check-in) at
the database level, not just in application code.

Phase 2+ will add: `presence_events`, `activities`, `correction_requests`, `audit_log` — do not
create these tables yet (see `PRD.md §3` and `enhancements.md`).

## 4. Status values (Phase 1 subset)

| Status | Meaning | Set on Phase 1 |
|---|---|---|
| Scheduled | Session created, not yet open | `sessions.status` |
| Open | Members may check in | `sessions.status` |
| Closed | Admin closed it / time window passed | `sessions.status` |
| Present | Checked in on time, inside radius | `attendance_records.final_status` |
| Late | Checked in after grace period, inside radius | `attendance_records.final_status` |
| Pending Verification | GPS accuracy too low to confidently accept/reject | `attendance_records.final_status` |
| Rejected | Outside radius with acceptable accuracy, or failed validation | `attendance_records.final_status` |

Full status vocabulary (including Phase 2+ states like Left Venue, Returned, Offline Sync Pending)
is documented in the Rulebook §2.2 for future reference — don't implement the extra states yet.

## 5. Photo evidence storage

- Stored outside the database as files (local disk under a `media/` folder for Phase 1, keyed by
  UUID filename); `attendance_records.photo_reference` stores the relative path.
- Working Book §13.1 explicitly warns against embedding large photos directly in Excel/DB — this
  matches that guidance.
- Never store raw uploaded files from anything other than a live camera capture; enforce
  "live capture only" at the frontend (no file-picker fallback) per Rulebook §7.2.

## 6. Monorepo folder layout (Repo B)

```
attendance-app-dev/
├── docs/                  # mirrored from Repo A
├── backend/
│   ├── app/
│   │   ├── api/           # FastAPI routers (auth, sessions, attendance, reports)
│   │   ├── models/         # SQLAlchemy models
│   │   ├── schemas/         # Pydantic request/response schemas
│   │   ├── services/        # business rules (distance calc, status calc)
│   │   ├── core/            # config, security/JWT, db session
│   │   └── main.py
│   ├── alembic/            # migrations
│   ├── tests/
│   └── requirements.txt
├── frontend/
│   ├── src/
│   │   ├── pages/           # route-level views (Login, AdminDashboard, MemberCheckIn, etc.)
│   │   ├── components/       # reusable UI (shadcn-based)
│   │   ├── lib/              # API client, auth helpers
│   │   └── main.tsx
│   ├── index.html
│   └── package.json
└── README.md
```

## 7. Branching model

- `main` — integration branch; always runnable end-to-end against the current
  `API_contract.md`.
- `backend` — backend developer's working branch → PR into `main`.
- `frontend` — frontend developer's working branch → PR into `main`.

Because both branches build against a shared contract, **`API_contract.md` changes must be agreed
and merged to `main` before either side codes against the new shape** — this is the one place
strict coordination matters even with a simple two-branch model.

## 8. Suggested API surface

See `API_contract.md` for the authoritative, detailed contract. Summary:

| Method | Path | Purpose |
|---|---|---|
| POST | `/auth/login` | Authenticate, return JWT |
| GET | `/me` | Current user profile |
| POST | `/sessions` | Admin: create session |
| GET | `/sessions` | List sessions (role-scoped) |
| GET | `/sessions/{id}` | Session detail + check-ins (admin) |
| PATCH | `/sessions/{id}/close` | Admin: close session |
| POST | `/attendance/check-in` | Member: submit check-in |
| GET | `/attendance/history` | Member: own check-in history |
| GET | `/reports/attendance.xlsx` | Admin: export Excel report |

**## 9. Phase 2 Architecture Extension**

Phase 2 builds upon the Phase 1 architecture without replacing any existing architectural
decisions. The overall system structure, technology stack, repository organization, and
communication flow remain unchanged. This phase focuses on implementing the documented
architecture and introducing additional business capabilities required by the project roadmap.

### Phase 2 objectives

The primary objectives of Phase 2 are:

- Complete backend implementation using the architecture defined in this document.
- Develop the React frontend that consumes the FastAPI backend.
- Implement secure authentication and authorization.
- Build the attendance workflow with business rule validation.
- Generate attendance reports from PostgreSQL data.
- Establish a stable integration between frontend, backend, and database.

### Architectural continuity

The following architectural decisions remain unchanged from Phase 1:

- React + TypeScript continues as the frontend technology.
- FastAPI remains the backend framework.
- PostgreSQL continues as the system of record.
- SQLAlchemy and Alembic remain responsible for data persistence and schema migrations.
- JWT continues to provide stateless authentication.
- Excel files remain an export format only and are never treated as the primary data source.

Phase 2 extends the implementation of these components rather than introducing new
architectural technologies.

### Repository evolution

The repository structure introduced in Phase 1 remains valid throughout Phase 2.

Instead of reorganizing the project, existing modules will gradually be populated with
production-ready implementations. New source files may be added inside existing folders as
features are implemented, but the overall project organization remains stable.

This approach ensures that documentation, implementation, and team collaboration continue to
reference a consistent repository structure throughout development.

### Implementation approach

Phase 2 follows an incremental implementation strategy.

Development progresses through small, reviewable milestones where documentation is reviewed
before implementation begins. Each completed milestone is validated before the next milestone
starts, allowing both frontend and backend development to evolve together while remaining
aligned with the shared API contract.

Future phases may extend the architecture with additional capabilities described in
`enhancements.md`, but those features are intentionally outside the scope of Phase 2.

## 10. Phase 3 Architecture Extension

Phase 3 extends the existing attendance platform by introducing the Activity Management Layer.

The architecture introduced during Phases 1 and 2 remains unchanged. Phase 3 adds new business capabilities without modifying the established technology stack, repository organization, communication flow, authentication model, or existing API architecture.

The Activity Layer is implemented as an extension of the existing system rather than as an independent subsystem.

---

### 10.1 Phase 3 Objectives

The primary architectural objectives of Phase 3 are:

- Introduce structured activity management.
- Support administrator-managed activity assignments.
- Track activity progress throughout its lifecycle.
- Support activity evidence submission.
- Introduce activity review and verification workflows.
- Provide reusable activity templates.
- Generate activity-specific reports.
- Preserve complete activity history.
- Maintain backward compatibility with existing attendance functionality.

---

### 10.2 Architectural Continuity

Phase 3 preserves all architectural decisions established during earlier phases.

The following remain unchanged:

- React + TypeScript remains the frontend technology.
- FastAPI remains the backend framework.
- PostgreSQL remains the system of record.
- SQLAlchemy and Alembic continue managing persistence and migrations.
- JWT remains responsible for authentication.
- Existing attendance workflows remain unchanged.
- Existing API endpoints remain valid.
- Existing repository organization remains unchanged.

Phase 3 introduces additional modules rather than replacing existing architecture.

---

### 10.3 Activity Layer Architecture

The Activity Layer operates alongside the existing Attendance Layer.

Conceptually the backend becomes:

```
Authentication

↓

Event Management

↓

Attendance Management

↓

Presence Monitoring

↓

Activity Management

↓

Reporting
```

The Activity Layer communicates with existing services through clearly defined interfaces while remaining logically independent.

Attendance data and Activity data are related but remain separate business domains.

---

### 10.4 Service Architecture

Phase 3 introduces additional backend services dedicated to Activity Management.

Typical responsibilities include:

Activity Service

- activity lifecycle
- activity retrieval
- activity validation

Assignment Service

- volunteer assignment
- assignment validation
- assignment conflict detection

Progress Service

- progress updates
- progress history

Evidence Service

- evidence validation
- evidence management

Review Service

- review workflow
- verification
- correction requests

Template Service

- template management
- template application

Report Service

- activity reporting
- export generation

Each service owns its business rules and communicates through clearly defined interfaces.

---

### 10.5 Activity Workflow Architecture

The Activity lifecycle follows the documented business workflow.

Typical progression:

```
Activity Created

↓

Published

↓

Assigned

↓

In Progress

↓

Submitted

↓

Needs Changes (optional)

↓

Resubmitted (optional)

↓

Verified
```

State transitions are validated by backend services before changes are committed.

---

### 10.6 Evidence Architecture

Activity evidence is treated as an extension of Activity progress rather than as an independent subsystem.

The architecture supports:

- photograph evidence
- video evidence
- evidence metadata
- evidence validation
- evidence history

Evidence files remain external to the database while metadata remains stored within PostgreSQL.

---

### 10.7 Activity History Architecture

Activity History provides an immutable historical view of completed work.

The architecture supports:

- chronological history
- submission history
- review history
- progress history
- evidence history

Historical records remain read-only after creation.

---

### 10.8 Reporting Architecture

Activity reporting extends the existing reporting subsystem.

Reports retrieve information from PostgreSQL and generate structured outputs without modifying operational data.

Attendance reporting and Activity reporting remain independent reporting capabilities while sharing common infrastructure.

---

### 10.9 Architectural Principles

Phase 3 follows the following architectural principles:

- Preserve backward compatibility.
- Extend existing architecture instead of replacing it.
- Maintain separation between Attendance and Activity domains.
- Keep business rules centralized in services.
- Keep API routes thin.
- Use transactional operations where multiple entities are modified.
- Preserve complete historical information.
- Generate reports from the database rather than maintaining redundant data.

---

### 10.10 Phase 3 Summary

Phase 3 extends the existing architecture with a dedicated Activity Management Layer while preserving all architectural decisions established during earlier phases.

The resulting architecture continues to support incremental development, independent testing, clear service boundaries, and future extensibility without disrupting the existing Attendance platform.

## 11. Phase 4 Architecture Extension

Phase 4 focuses on preparing the system for production deployment.

Unlike previous phases, Phase 4 does not introduce new business features. Instead, it strengthens the reliability, security, recoverability, observability, and operational readiness of the existing system.

The architecture established during Phases 1–3 remains unchanged. Phase 4 extends the platform by introducing production hardening capabilities.

---

### 11.1 Phase 4 Objectives

The primary architectural objectives of Phase 4 are:

- support reliable offline operation;
- automatically synchronize offline data;
- improve system security through spoofing detection;
- provide comprehensive audit logging;
- improve operational monitoring;
- support backup and disaster recovery;
- prepare the system for production deployment;
- preserve backward compatibility with all previous phases.

---

### 11.2 Architectural Continuity

Phase 4 preserves all previous architectural decisions.

The following remain unchanged:

- React + TypeScript frontend
- FastAPI backend
- PostgreSQL system of record
- SQLAlchemy ORM
- Alembic migrations
- JWT authentication
- Attendance architecture
- Presence architecture
- Activity Management architecture

Production hardening extends the existing architecture rather than replacing it.

---

### 11.3 Production Hardening Architecture

The overall platform architecture evolves into:

```
Authentication

↓

Event Management

↓

Attendance Management

↓

Presence Monitoring

↓

Activity Management

↓

Production Hardening

↓

Reporting
```

Production Hardening provides shared infrastructure services used throughout the application rather than introducing an independent business module.

---

### 11.4 Offline Synchronization Architecture

Offline Synchronization enables supported operations to continue during temporary network interruptions.

The architecture consists of:

- offline request queue;
- local operation storage;
- automatic synchronization;
- synchronization monitoring;
- conflict detection;
- synchronization recovery.

The backend remains the authoritative source of truth.

Synchronization should occur automatically when connectivity is restored.

---

### 11.5 Offline Queue Architecture

Pending offline operations are stored locally using IndexedDB.

The offline queue is responsible for:

- preserving pending operations;
- maintaining operation order where required;
- tracking synchronization status;
- retrying failed synchronization;
- removing completed operations after successful synchronization.

The queue remains independent of business modules while supporting Attendance, Presence, Activity Management, and future capabilities.

---

### 11.6 Conflict Resolution Architecture

Synchronization follows a server-authoritative model.

If synchronized data conflicts with server state:

- server state remains authoritative;
- conflicting operations are rejected or reconciled according to documented business rules;
- local state is updated using the confirmed server response.

Conflict resolution remains centralized and consistent across all supported modules.

---

### 11.7 Background Processing Architecture

Certain production operations execute asynchronously through a dedicated background processing layer.

Typical responsibilities include:

- audit log persistence;
- synchronization processing;
- notification delivery;
- scheduled maintenance tasks;
- backup scheduling;
- future production services.

The architecture intentionally separates background processing from synchronous request handling to improve responsiveness and scalability.

Implementation technology remains an implementation decision rather than an architectural requirement.

---

### 11.8 Audit Logging Architecture

Audit Logging provides a centralized, immutable record of important system operations.

Audit logging should capture:

- authentication events;
- authorization events;
- attendance operations;
- activity operations;
- administrative operations;
- reporting operations;
- configuration changes;
- backup and recovery events;
- deployment-related administrative actions.

Audit logging operates asynchronously through the background processing layer.

Audit records are immutable and available only through authorized administrative interfaces.

Audit retention follows a configurable policy.

---

### 11.9 Anti-Spoofing Architecture

The Anti-Spoofing subsystem assists in identifying potentially suspicious behaviour.

Detection mechanisms may include:

- mock location detection;
- impossible travel analysis;
- abnormal GPS movement;
- emulator detection where supported;
- device time manipulation detection;
- repeated suspicious behaviour analysis;
- rooted or jailbroken device detection where supported.

Detection results are treated as security indicators rather than definitive proof of misconduct.

The architecture supports administrator review before enforcement decisions are made.

---

### 11.10 Backup and Recovery Architecture

The platform supports comprehensive recovery from operational failures.

The backup architecture includes:

- PostgreSQL database backups;
- uploaded evidence and media backups;
- configurable backup schedules;
- configurable retention policies;
- backup verification;
- recovery validation;
- rollback support.

Backup and recovery procedures are considered part of normal production operations.

---

### 11.11 Production Deployment Architecture

Phase 4 prepares the application for production deployment.

The deployment architecture supports:

- environment-specific configuration;
- secure secret management;
- production logging;
- health monitoring;
- rollback procedures;
- deployment verification;
- scalable deployment strategies.

The architecture remains deployment-platform agnostic.

Specific deployment technologies are implementation decisions documented separately.

---

### 11.12 Architectural Principles

Phase 4 follows the following principles:

- preserve existing architecture;
- extend rather than redesign;
- backend remains authoritative;
- production services remain modular;
- background processing remains isolated from request handling;
- audit records remain immutable;
- synchronization remains reliable and recoverable;
- backups remain verifiable;
- deployments remain repeatable;
- rollback procedures remain available.

---

### 11.13 Phase 4 Summary

Phase 4 extends the existing architecture with production hardening capabilities while preserving the architectural decisions established throughout Phases 1–3.

The resulting architecture improves operational reliability, security, recoverability, and deployment readiness without introducing new user-facing functionality or altering the existing business workflows.

