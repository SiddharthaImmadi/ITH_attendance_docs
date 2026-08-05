# changelog.md

> Follows the spirit of [Keep a Changelog](https://keepachangelog.com/): newest entry on top,
> grouped as **Added / Changed / Fixed / Removed**. Every entry should be small enough to read in
> ten seconds. This is a human (and agent) log of what actually shipped — not a task list (that's
> `progress.md`) and not a proposal log (that's `enhancements.md`).

## How to use this file

- Add an entry whenever a PR merges into `main`.
- If a change touches `API_contract.md`, say so explicitly in the entry and link the section.
- Use the same `type(scope)` prefixes as commit messages (`feat`, `fix`, `chore`, `docs`).
- Version numbers are informal for now (`v0.1.0`, `v0.2.0`, ...) — bump the minor version each time
  a Phase 1 feature (from `PRD.md §5`) becomes usable end-to-end; bump patch for fixes.

---

## [Unreleased]

### Added
- Nothing yet.

---

---

## [v0.3.0] — 2026-08-06

### Added

- `docs(production)` Phase 4 Production Hardening documentation completed for backend and frontend.
- `docs(production)` Offline-first architecture documented.
- `docs(production)` Automatic synchronization architecture documented.
- `docs(production)` Global synchronization status workflow documented.
- `docs(production)` Audit Log management documented.
- `docs(production)` Security Monitoring documented.
- `docs(production)` Backup Management documented.
- `docs(production)` Production deployment guidance documented.
- `docs(production)` Production verification workflow documented.
- `docs(production)` Rollback strategy documented.
- `docs(production)` Operational monitoring guidance documented.

### Changed

- `docs(prd)` `PRD.md` extended with Phase 4 Production Hardening requirements.
- `docs(architecture)` `system_architecture.md` extended with production infrastructure, synchronization, audit logging, security monitoring, backup management, deployment, and operational monitoring architecture.
- `docs(database)` `backend_database_schema.md` extended with production support tables including synchronization, audit logging, security monitoring, backup management, deployment, and operational metadata.
- `docs(backend)` `backend_architecture.md` extended with production architecture.
- `docs(backend)` `backend_services.md` extended with synchronization, audit, security, backup, deployment, and monitoring services.
- `docs(backend)` `backend_api_implementation.md` extended with Phase 4 implementation guidance.
- `docs(backend)` `backend_testing.md` extended with production testing strategy.
- `docs(backend)` `backend_environment.md` extended with production configuration guidance.
- `docs(backend)` `backend_authentication.md` extended with production authentication and security guidance.
- `docs(backend)` `backend_todo.md` extended with Phase 4 implementation milestones.
- `docs(backend)` `rules_backend.md` extended with production implementation rules.
- `docs(frontend)` `frontend_ui_spec.md` extended with production interfaces and workflows.
- `docs(frontend)` `frontend_routing.md` extended with production routing.
- `docs(frontend)` `frontend_state_management.md` extended with offline queue, synchronization, audit, and security state management.
- `docs(frontend)` `frontend_component_guidelines.md` extended with production component architecture.
- `docs(frontend)` `frontend_design_system.md` extended with production design standards.
- `docs(frontend)` `rules_frontend.md` extended with production implementation rules.
- `docs(frontend)` `frontend_todo.md` extended with the complete Phase 4 implementation roadmap.
- `docs(deployment)` `deployment_guide.md` extended with production deployment, backup, rollback, monitoring, and verification guidance.
- `docs(progress)` `progress.md` updated to reflect completion of Phase 4 documentation.

### Fixed

- `docs(production)` Documentation standardized around an offline-first architecture with automatic synchronization.
- `docs(production)` Audit Logs and Security Monitoring consolidated into a unified administrator interface while preserving independent backend data models.
- `docs(production)` Synchronization workflow clarified with ordered queue processing and automatic retry behavior.
- `docs(production)` Backup workflow standardized around administrator-controlled operations with automatic verification.
- `docs(production)` Deployment guidance made hosting-provider independent.
- `docs(production)` Production terminology standardized across backend and frontend documentation.

### Removed

- `docs(production)` Phase 4 deployment placeholders replaced with complete production deployment guidance.

## [v0.2.0] — 2026-07-29

### Added

- `docs(activity)` Phase 3 Activity Layer documentation completed for backend and frontend.
- `docs(activity)` Activity Management architecture and workflows documented.
- `docs(activity)` Activity Assignment workflow documented.
- `docs(activity)` Activity Progress Tracking documented.
- `docs(activity)` Activity Evidence management documented.
- `docs(activity)` Activity Review and resubmission workflow documented.
- `docs(activity)` Activity Template system documented.
- `docs(activity)` Activity History behavior documented.
- `docs(activity)` Activity reporting requirements documented.
- `docs(activity)` Activity assignment notification behavior documented.
- `docs(activity)` Supported offline activity synchronization behavior documented.
- `docs(backend)` Phase 3 backend testing strategy documented.
- `docs(backend)` Phase 3 backend implementation roadmap documented.
- `docs(frontend)` Phase 3 frontend UI, routing, state, component, design, rules, and implementation guidance documented.

### Changed

- `docs(prd)` `PRD.md` extended with Phase 3 Activity Layer requirements.
- `docs(api)` `API_contract.md` extended with the approved Activity Layer API.
- `docs(database)` `backend_database_schema.md` extended with Phase 3 tables and relationships.
- `docs(backend)` `backend_architecture.md` extended with Activity Layer architecture.
- `docs(backend)` `backend_services.md` extended with Activity Layer services.
- `docs(backend)` `backend_api_implementation.md` extended with Phase 3 implementation guidance.
- `docs(backend)` `backend_testing.md` extended with Phase 3 testing strategy.
- `docs(backend)` `backend_todo.md` extended with Phase 3 implementation milestones.
- `docs(backend)` `backend_authentication.md` extended with Phase 3 authorization rules.
- `docs(backend)` `backend_environment.md` extended with Activity Layer configuration.
- `docs(backend)` `rules_backend.md` extended with Activity Layer business rules.
- `docs(frontend)` `frontend_ui_spec.md` extended with Phase 3 Activity Layer screens and workflows.
- `docs(frontend)` `frontend_routing.md` extended with Activity Layer navigation and route behavior.
- `docs(frontend)` `frontend_state_management.md` extended with Activity, Assignment, Progress, Evidence, Review, Template, Report, and offline state guidance.
- `docs(frontend)` `frontend_component_guidelines.md` extended with Activity Layer component architecture.
- `docs(frontend)` `frontend_design_system.md` extended with Activity Layer visual standards.
- `docs(frontend)` `rules_frontend.md` extended with Phase 3 frontend behavior and implementation rules.
- `docs(frontend)` `frontend_todo.md` extended with the complete Activity Management implementation checklist.
- `docs(progress)` `progress.md` updated to reflect completion of both backend and frontend Phase 3 documentation.

### Fixed

- `docs(activity)` Activity Review workflow standardized around `NEEDS_CHANGES` and `VERIFIED`.
- `docs(activity)` Removed the need for a separate `REJECTED` state from the Activity Review workflow.
- `docs(activity)` Activity submission behavior clarified: completed submissions are read-only and remain viewable.
- `docs(activity)` Evidence limits standardized to a maximum of 10 photographs and 2 videos, with each video under 1 minute.
- `docs(activity)` Template behavior clarified so generated activities remain independent records.
- `docs(activity)` Offline behavior clarified so supported pending data is stored locally and synchronized when connectivity returns.
- `docs(frontend)` Frontend documentation structure aligned with the finalized Activity Layer workflows.
- `docs(backend)` Backend documentation flow aligned with the implementation sequence.

### Removed

- `docs(activity)` Separate `REJECTED` outcome from the Phase 3 Activity Review workflow.

---

## [v0.1.2] — 2026-07-20

### Changed
- `API_contract.md` consolidated into a single, comprehensive markdown file. All 12 endpoint
  specifications and error codes now live in one place with a table of contents, organized by
  section (Authentication, Sessions, Attendance/Check-in, Reports, Error Codes). Markdown format
  is more readable than separate JSON files.

### Removed
- Individual `api-contract-*.json` files are now deprecated in favor of the consolidated
  `API_contract.md`. Developers and agents should reference only `API_contract.md`.

---

## [v0.1.1] — 2026-07-20

### Changed
- `API_contract.md` replaced with an index. The full contract now lives in 12 separate
  `api-contract-*.json` files, one per purpose (admin login, member login, get current user,
  session creation, session list, session detail, session close, GPS check-in, live photo capture,
  attendance history, report export, error code reference) so each is unambiguous and directly
  machine-readable by Claude Code / Kiro.
- GPS check-in and live photo capture are documented as separate files (`08` and `09`) even though
  they are one physical API request, per request from the project owner for clarity.

---

## [v0.1.0] — 2026-07-20

### Added
- Initial documentation set created: `context.md`, `PRD.md`, `system_architecture.md`,
  `API_contract.md`, `rules_backend.md`, `rules_frontend.md`, `deployment_guide.md`,
  `changelog.md`, `enhancements.md`, `development_roadmap.md`, `progress.md`.
- Architecture decisions locked in: React/TS/Tailwind/shadcn frontend, FastAPI + PostgreSQL
  backend, local-only deployment for Phase 1, monorepo with `main`/`frontend`/`backend` branches.

### Changed
- N/A (first entry)

### Fixed
- N/A (first entry)
