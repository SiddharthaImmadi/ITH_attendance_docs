# Context.md — Project Context for Humans and Agents

> Read this file first. It orients any developer or AI agent (Claude Code, Kiro) joining this
> project on what it is, why it exists, who is building it, and how the repos are organized.
> Every other doc in this set assumes you've read this one.

## 1. What this project is

**InnoTech-Hub Attendance & Activity Tracking Application** — a verified attendance system that
replaces manual/paper attendance with evidence-backed check-ins: GPS geofence + live (camera)
photo + time-window validation, with clean report exports for admins.

Source material (in repo A, `/source-docs`):

- `Attendance_Application_Pitch.pptx` — product concept and 4-phase roadmap
- `Attendance_Application_Rulebook.pdf` — governance/compliance rules (v1.0, July 2026)
- `Attendance_Application_Working_Book.pdf` — functional workflow and technical workbook (v1.0, July 2026)

Everything in this doc set is derived from those three sources. If a rule here ever seems to
conflict with them, the source documents win — flag the conflict in `enhancements.md` rather than
silently resolving it.

## 2. Why it exists

Manual attendance has three problems the product is built to solve (Pitch, slide 2):

1. **Proxy attendance** — sheets can be signed by someone else.
2. **No venue proof** — a checkmark doesn't prove someone stayed at the location.
3. **Slow reporting** — Excel reports assembled after the fact, missing context.

The fix: every attendance record carries identity + GPS + timestamp + photo evidence, captured
live, validated server-side.

## 3. Who is building it

Two developers, both new to shipping a full project and new to collaborative git workflows. Both
are using AI coding agents (**Claude Code** and **Kiro**) as pair-programmers. This documentation
set exists specifically so those agents have enough standing context to make good decisions
without either developer having to re-explain the project in every session.

| Role | Owns | Primary docs to read |
|---|---|---|
| Backend developer | API, database, business rules, auth | `rules_backend.md`, `API_contract.md`, `system_architecture.md` |
| Frontend developer | Web UI, API integration, UX | `rules_frontend.md`, `API_contract.md`, `system_architecture.md` |
| Both | Scope, priorities, definitions | `PRD.md`, `development_roadmap.md`, `progress.md` |

Neither developer builds Flutter yet — **Flutter is explicitly a later-phase concern.** Phase 1 is
being built as a **web application** (see `system_architecture.md` for the chosen stack) so both
developers can learn one ecosystem (JS/TS + Python) before adding a mobile client.

## 4. Repository layout

Two repositories:

### Repo A — `attendance-app-docs` (public, read-only for outsiders)

- Public so anyone can see what's being built and why.
- **Not editable** by outside users — only the two developers (and the owner) can push.
- Contains only documentation: this doc set, plus the three original source files under
  `/source-docs`.
- Nothing here should ever contain secrets, credentials, or connection strings.

### Repo B — `attendance-app-dev` (private, developers + agents only)

- Private. Only the two developers have access (and, through them, their local Claude Code / Kiro
  agent sessions).
- Mirrors the full doc set from Repo A under `/docs` so agents working in Repo B have the same
  context without switching repos.
- Contains the actual application code.

**Branching model** (see also `system_architecture.md §7` for the full workflow):

- `main` — integration branch. Only merged code that both sides have agreed works against the
  current `API_contract.md`. This is what gets run locally end-to-end.
- `backend` — the backend developer's long-lived working branch. All backend commits land here
  first, then get PR'd into `main`.
- `frontend` — the frontend developer's long-lived working branch. All frontend commits land here
  first, then get PR'd into `main`.

Rule of thumb: if a change affects the contract between frontend and backend (new endpoint,
changed field, new status value), update `API_contract.md` on `main` *before* either branch builds
against it, and note the change in `changelog.md`.

## 5. How the documentation set fits together

| File | Answers |
|---|---|
| `context.md` (this file) | What is this, who's building it, how are repos organized? |
| `PRD.md` | What exactly must Phase 1 do? What's explicitly out of scope? |
| `system_architecture.md` | What's the stack, the data model, the folder layout? |
| `API_contract.md` | What are the exact endpoints, request/response shapes? |
| `rules_backend.md` | How should backend code be written and reviewed? |
| `rules_frontend.md` | How should frontend code be written and reviewed? |
| `development_roadmap.md` | What order do we build things in, and what's "done" mean? |
| `deployment_guide.md` | How do I run this on my machine? |
| `progress.md` | What's actually done right now, as of today? |
| `changelog.md` | What changed, release by release? |
| `enhancements.md` | What did an agent or developer suggest that we deliberately deferred? |

## 6. Key terms (from the Rulebook and Working Book, kept consistent across all docs)

| Term | Meaning |
|---|---|
| Session | A defined attendance period (class, event, shift) with a venue, radius, and time window. |
| Venue radius | Permitted geographic boundary measured from configured venue coordinates. |
| Check-in | Verified action recording arrival and starting the attendance period. |
| Check-out | Verified action recording departure and ending the attendance period. |
| Evidence | Live photo + GPS coordinates + timestamp + device info backing a record. |
| Exception | An authorized, documented departure from a standard rule. |
| Pending Verification | Status when a required check is inconclusive and needs human review. |

Full status list and definitions live in `system_architecture.md §4` (Data Model) — this table is
just the minimum vocabulary needed to read the other docs.

## 7. Ground rules for agents working in this repo

1. Phase 1 scope is defined in `PRD.md` — do not implement Phase 2–4 features (monitoring,
   activity tracking, offline sync, audit logs) unless explicitly asked. If you think of one
   anyway, log it in `enhancements.md` instead of building it.
2. Never trust client-submitted location, time, or status. Server-side validation is mandatory
   (Rulebook §6.1, §7.1) — see `rules_backend.md`.
3. Photo evidence must come from a live camera capture, never a gallery/file upload (Rulebook
   §7.2).
4. Any change to an API endpoint or data field must be reflected in `API_contract.md` in the same
   pull request.
5. Update `progress.md` and, where relevant, `changelog.md` at the end of a working session —
   don't leave it to the next person to reconstruct what happened.
