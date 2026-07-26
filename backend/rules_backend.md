# rules_backend.md — Backend Development Rules

> For the backend developer and any agent (Claude Code, Kiro) writing backend code in `/backend`.
> These rules exist to keep a beginner-authored codebase consistent and to enforce the
> non-negotiable security/compliance rules from the Rulebook.

## 1. Project layout

Follow the structure in `system_architecture.md §6`. Don't add new top-level folders under
`backend/app/` without updating that doc.

## 2. Non-negotiable rules (from the Rulebook — do not "optimize" these away)

1. **Server time only.** Never use a client-submitted timestamp for check-in validation. Read the
   time on the server when the request arrives. (Rulebook §7.3)
2. **No trusting client-computed status.** The client may show a preview, but `final_status` is
   always computed server-side from stored coordinates/time, never accepted as input. (Working
   Book §13.1: "never trust client-only decisions.")
3. **Duplicate check-in is a database-level constraint**, not just an application check (unique
   constraint on `session_id + member_id` — see `system_architecture.md §3`). Application-level
   checks can race; the DB constraint can't.
4. **Passwords are hashed** (bcrypt or argon2 via `passlib`), never logged, never returned in any
   API response.
5. **Photo evidence must be a live capture.** The backend cannot enforce "was this photo really
   just taken," but it must reject anything that isn't a valid image upload accompanying a
   check-in request, and the frontend must never offer a file-picker fallback (see
   `rules_frontend.md`).
6. **GPS accuracy matters as much as distance.** A check-in with poor accuracy should not be
   silently accepted or rejected — mark it `pending_verification` (Working Book §6.2). Don't
   collapse this into a binary inside/outside check.

## 3. Tech conventions

| Concern | Convention |
|---|---|
| Framework | FastAPI, async route handlers where they touch the DB |
| ORM | SQLAlchemy 2.x (declarative models in `app/models/`) |
| Validation | Pydantic schemas in `app/schemas/` — one `Create`, one `Read` (and `Update` if needed) per resource, never reuse ORM models directly as response models |
| Migrations | Alembic — every model change ships with a migration in the same commit |
| Config/secrets | `pydantic-settings`, values from environment variables only (`.env` for local dev, **never committed** — see §5) |
| Business logic | Lives in `app/services/`, not in route handlers — route handlers parse input, call a service, return the response |
| Distance calc | Haversine formula, implemented once in `app/services/geo.py`, unit-tested with known coordinate pairs |
| Linting/formatting | `ruff` + `black`, run before every commit |
| Testing | `pytest`, at minimum one test per acceptance criterion in `PRD.md §7` (F01–F04, F10) |

## 4. API discipline

- Every endpoint must match `API_contract.md` exactly (path, method, request/response shape,
  status codes). If you need to deviate, update the contract file first, in the same PR.
- Use FastAPI's automatic OpenAPI docs (`/docs`) to sanity-check your implementation against the
  contract — they should never drift.
- Return the standard error shape from `API_contract.md §1` for every error path; don't let
  unhandled exceptions leak stack traces to the client.

## 5. Secrets & environment

- `DATABASE_URL`, `JWT_SECRET_KEY`, and any other secret live in `backend/.env` (git-ignored).
- Commit a `backend/.env.example` with placeholder values so the other developer/agent knows what
  variables are needed, per `deployment_guide.md`.
- Never put real secrets in Repo A (public docs repo) — not even in an example.

## 6. Git / commit conventions (backend branch)

- Work happens on the `backend` branch (see `system_architecture.md §7`); open a PR into `main`
  when a feature is complete and tested.
- Commit messages: `type(scope): summary`, e.g. `feat(attendance): add duplicate check-in guard`,
  `fix(auth): correct JWT expiry`.
- Every PR into `main` that touches an endpoint must link to (or restate) the relevant section of
  `API_contract.md`.
- If a migration is included, say so explicitly in the PR description (`Includes Alembic
  migration: yes/no`).

## 7. When you (the agent) notice something worth changing

If you spot a better approach than what's specified here — e.g., a different validation library,
a different status-calculation order — don't just implement it silently. Add an entry to
`enhancements.md` describing the suggestion and rationale, and only implement it after the
developer approves, unless it's a trivial non-behavioral fix (typo, formatting).

## 8. Explicitly out of scope for Phase 1 backend

Do not build, even if it seems easy to add "while you're in there": offline sync endpoints,
presence/left-venue tracking, activity endpoints, correction/approval workflow, audit log table,
multi-role permission matrix beyond admin/member. These are documented in `PRD.md §3` and
`development_roadmap.md` for later phases.

## 9. Phase 2 Backend Rules

> Phase 2 extends the attendance system with continuous attendance monitoring, early check-out approval, and attendance summaries. These rules supplement the Phase 1 rules and do not replace them.

### 9.1 Attendance Lifecycle

The backend is the single source of truth for the attendance lifecycle.

Attendance progresses through backend-controlled states only.

Example lifecycle:

```
Check-in

↓

Active Attendance

↓

(Optional) Early Check-out Request

↓

Administrator Decision

↓

Approved → Complete Check-out

OR

Rejected → Continue Attendance

↓

Attendance Completed
```

The frontend must never determine attendance state transitions.

---

### 9.2 Presence Monitoring

Presence monitoring must be calculated by the backend.

The backend is responsible for:

- determining whether a member is inside or outside the attendance area;
- recording presence events;
- maintaining chronological attendance history.

The frontend is responsible only for displaying backend results.

---

### 9.3 Attendance Timeline

All attendance events must be recorded chronologically.

Examples include:

- Check In
- Entered Venue
- Left Venue
- Returned
- Early Check-out Requested
- Early Check-out Approved
- Early Check-out Rejected
- Check Out

Timeline records must be generated by backend services rather than reconstructed by the frontend.

---

### 9.4 Early Check-out Approval

Early check-out requests require administrator approval.

Rules:

- Members must provide a reason.
- Approval decisions are made only by administrators.
- Members cannot complete early check-out until approval is granted.
- Rejected requests return the member to Active Attendance.

The backend must enforce these rules regardless of frontend behavior.

---

### 9.5 Attendance Completion

Attendance is considered complete only after the backend successfully processes check-out.

The backend must calculate:

- attendance duration;
- final attendance status;
- check-out timestamp;
- attendance summary.

These values must never be supplied by the client.

---

### 9.6 Business Logic Location

All attendance-related business logic belongs in `app/services/`.

Examples include:

- attendance monitoring;
- presence evaluation;
- early check-out validation;
- attendance summary generation;
- administrator approval workflow.

Route handlers should only:

- validate requests;
- call services;
- return responses.

---

### 9.7 Query Efficiency

Attendance monitoring endpoints may be called frequently.

Backend implementations should:

- minimize unnecessary database queries;
- return only required fields;
- paginate large collections where appropriate;
- avoid N+1 query patterns.

Performance improvements must not change API behavior.

---

### 9.8 API Consistency

All new Phase 2 endpoints must follow the existing API conventions.

Requirements:

- consistent request validation;
- consistent response models;
- standard error format;
- predictable HTTP status codes.

Update `API_contract.md` before implementing any new endpoint.

---

### 9.9 Error Handling

Attendance workflow errors should return clear business-level error responses.

Examples include:

- attendance already completed;
- approval pending;
- approval rejected;
- invalid attendance state;
- session not active;
- monitoring unavailable.

Avoid exposing internal implementation details.

---

### 9.10 Database Changes

Every new attendance feature requiring persistent storage must include:

- SQLAlchemy model updates;
- Alembic migration;
- updated schemas;
- updated service layer;
- corresponding tests.

Schema changes must never be made without migrations.

---

### 9.11 Testing Requirements

Each new attendance feature should include automated tests covering:

- successful workflow;
- invalid workflow transitions;
- permission enforcement;
- validation failures;
- edge cases;
- service-layer business logic.

Business rules should be tested independently from API routes.

---

### 9.12 Phase 2 Development Principles

During Phase 2 development:

- The backend remains the source of truth.
- Business rules belong in services.
- Clients are never trusted to compute attendance outcomes.
- Administrator approval is always enforced server-side.
- Every model change includes a migration.
- Every API change updates the API contract before implementation.