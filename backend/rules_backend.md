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
