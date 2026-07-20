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
- Nothing yet since v0.1.2.

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
