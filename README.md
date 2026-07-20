# Attendance & Activity Tracking Application — Documentation

Public repository containing all project documentation, requirements, and architectural decisions for the InnoTech-Hub Attendance & Activity Tracking Application (Phase 1 MVP).

## Quick Navigation

### Getting Started
- **[context.md](context.md)** — Start here. Project overview, purpose, team, repo structure, key terms.
- **[FIRST_STEPS.md](FIRST_STEPS.md)** — For the frontend developer: day-by-day checklist for week 1.
- **[COLLABORATION_GUIDE.md](COLLABORATION_GUIDE.md)** — How frontend and backend developers coordinate daily.

### Product & Requirements
- **[PRD.md](PRD.md)** — Phase 1 scope: what must ship, what's explicitly out of scope.
- **[development_roadmap.md](development_roadmap.md)** — Phases 1–4 breakdown, Phase 1 milestones.

### Technical Design
- **[system_architecture.md](system_architecture.md)** — Tech stack (React/FastAPI/PostgreSQL), data model, folder layout, branching.
- **[API_contract.md](API_contract.md)** — All 11 endpoints: request/response shapes, error codes, validation rules.

### Development Rules
- **[rules_backend.md](rules_backend.md)** — Backend conventions, non-negotiable security rules.
- **[rules_frontend.md](rules_frontend.md)** — Frontend conventions, "premium feel" guidelines.

### Operations
- **[deployment_guide.md](deployment_guide.md)** — Local setup: prerequisites, environment, running both frontend and backend.
- **[progress.md](progress.md)** — Living tracker of what's done, what's in progress, blockers.
- **[changelog.md](changelog.md)** — Version history and changes shipped.
- **[enhancements.md](enhancements.md)** — Deferred ideas and agent-suggested improvements.

### Source Documents
- **[Attendance_Application_Pitch.pptx](source-docs/Attendance_Application_Pitch.pptx)** — Product concept, 4-phase roadmap, suggested architecture.
- **[Attendance_Application_Rulebook.pdf](source-docs/Attendance_Application_Rulebook.pdf)** — Governance, compliance, and conduct rules (v1.0).
- **[Attendance_Application_Working_Book.pdf](source-docs/Attendance_Application_Working_Book.pdf)** — Functional workflows, operations manual, implementation checklists (v1.0).

## Key Facts

| Aspect | Value |
|---|---|
| **Phase** | 1 (Core MVP) |
| **Frontend** | React + TypeScript + Tailwind + shadcn/ui |
| **Backend** | FastAPI + PostgreSQL + SQLAlchemy |
| **Deployment** | Local only (Phase 1) |
| **Git structure** | Monorepo: `main`, `frontend`, `backend` branches |
| **Docs owner** | Public (read-only); code in private Repo B |

## How to Use This Repo

1. **Frontend developer:** Read `context.md`, then `FIRST_STEPS.md`.
2. **Backend developer:** Read `context.md`, then `rules_backend.md` and `API_contract.md`.
3. **Both:** Review `COLLABORATION_GUIDE.md` for daily sync.
4. **Project owner/auditor:** Start with `PRD.md` and `development_roadmap.md`.

## Status

- ✅ Documentation complete
- ✅ Architecture locked in
- ✅ API contract finalized
- ⏳ Code scaffolding not yet started (Milestone 0)

Last updated: 2026-07-20
