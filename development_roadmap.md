# development_roadmap.md — Phase Plan & Phase 1 Milestones

## 1. Full roadmap (from Pitch, slide 8)

| Phase | Focus | Status |
|---|---|---|
| **Phase 1 — Core MVP** | Admin login, member login, session creation, GPS check-in, live photo, Excel export | **In progress** (current phase) |
| Phase 2 — Monitoring | Inside/outside tracking, left-venue alerts, check-out, duration calculation | Not started |
| Phase 3 — Activity layer | Member activity updates, evidence review, filters, extra report sheets | Not started |
| Phase 4 — Production hardening | Offline sync, audit logs, spoofing checks, backups, deployment | Not started |

Only Phase 1 is broken into milestones below. Later phases get their own milestone breakdown once
Phase 1 is stable — don't pre-plan their internals now (avoids the plan going stale before it's
relevant).

## 2. Phase 1 milestones

Both developers are new to shipping projects and to collaborative git — milestones are
deliberately small and sequenced so each one produces something demoable.

### Milestone 0 — Environment & scaffolding (both)
- Repo B created with `main`/`frontend`/`backend` branches.
- Backend: FastAPI skeleton boots, connects to local PostgreSQL, one health-check endpoint.
- Frontend: Vite + React + Tailwind + shadcn scaffolded, boots to a blank styled page.
- **Definition of done:** `deployment_guide.md` steps work end-to-end on both developers' machines.

### Milestone 1 — Auth (backend-led, frontend follows)
- Backend: `users` table + migration, `POST /auth/login`, `GET /me`, JWT issuing/verification.
- Frontend: login page, stores JWT, redirects based on role.
- **Definition of done:** Admin and member (seeded manually) can log in and see a role-appropriate
  blank dashboard.

### Milestone 2 — Session creation (backend-led, frontend follows)
- Backend: `sessions` table + migration, `POST /sessions`, `GET /sessions`, `GET /sessions/{id}`.
- Frontend: admin "Create Session" form, session list view.
- **Definition of done:** Admin can create a session through the UI and see it in a list.

### Milestone 3 — Check-in (both, tightly coupled — coordinate closely)
- Backend: `attendance_records` table + migration, Haversine distance service, `POST
  /attendance/check-in` with full validation (radius, time window, duplicate, accuracy).
- Frontend: member session view, live camera capture flow, GPS permission flow, check-in
  submission with success/error states.
- **Definition of done:** F01–F04 from `PRD.md §7` all pass manually.

### Milestone 4 — Reporting
- Backend: `GET /reports/attendance.xlsx` using openpyxl.
- Frontend: "Export" button on admin session detail view.
- **Definition of done:** F10 from `PRD.md §7` passes; exported file opens correctly in Excel/Sheets.

### Milestone 5 — Polish pass
- Frontend: apply the "premium feel" guidance from `rules_frontend.md §3` across all Phase 1
  screens — this is deliberately its own milestone rather than done piecemeal, so it doesn't get
  skipped under time pressure.
- Both: fix anything on the acceptance-criteria list that's still failing.
- **Definition of done:** A full walkthrough (login → create session → check in as member → export
  report) works without errors and looks intentional, not placeholder.

## 3. Dependencies between frontend and backend

| Frontend needs from backend | Backend needs from frontend |
|---|---|
| Stable `API_contract.md` before building a screen against it | None structurally — backend can be built and tested (via `/docs` or a REST client) ahead of the UI |
| Seeded admin/member accounts for manual testing | Realistic expectations on what browser APIs can/can't guarantee (e.g., GPS accuracy varies) |

Because the backend can be developed and tested independently (via FastAPI's `/docs` or a tool
like Postman/Insomnia), it's reasonable for the backend developer to run slightly ahead on each
milestone. The frontend developer should not block on a fully-polished backend — build against the
documented contract and adjust once the real endpoint is ready.

## 4. Exit criteria for Phase 1 (moving to Phase 2)

- All acceptance criteria in `PRD.md §7` pass.
- `progress.md` shows all Milestone 0–5 items complete.
- A short demo (per Milestone 5) can be given without manual workarounds.
- Any deferred ideas relevant to Phase 2 are captured in `enhancements.md` for review before
  Phase 2 planning begins.
