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
