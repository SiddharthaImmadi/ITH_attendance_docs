# progress.md — Current Status

> Living tracker. Update this at the end of every working session — this is what an agent
> (Claude Code, Kiro) should read to know "where did we leave off" without re-reading every other
> doc from scratch.

## Current phase
**Phase 1 — Core MVP** (see `development_roadmap.md`)

## Current milestone
**Milestone 0 — Environment & scaffolding** — not yet started.

## Snapshot (as of 2026-07-20)

| Milestone | Status |
|---|---|
| 0. Environment & scaffolding | Not started |
| 1. Auth | Not started |
| 2. Session creation | Not started |
| 3. Check-in | Not started |
| 4. Reporting | Not started |
| 5. Polish pass | Not started |

## What's done so far
- Read and internalized all three source documents (Pitch, Rulebook, Working Book).
- Made key architecture decisions: React/TS/Tailwind/shadcn frontend (Flutter deferred), FastAPI +
  PostgreSQL backend, local-only deployment, monorepo with `main`/`frontend`/`backend` branches.
- Full documentation set written (this file plus the 10 others listed in `context.md §5`).

## What's next
1. Create Repo A (public, docs) and Repo B (private, monorepo with `main`/`frontend`/`backend`
   branches).
2. Push this documentation set to Repo A's root and mirror it under Repo B's `/docs`.
3. Scaffold Milestone 0 (see `development_roadmap.md §2`) on both branches.

## Blockers
None currently.

## Updated (2026-07-20, post-consolidation)

**API Contract consolidation complete:**
- All 12 endpoint specifications now consolidated into one `API_contract.md` file
- Markdown format is cleaner and easier to navigate than separate JSON files
- Individual `api-contract-*.json` files are deprecated (can be ignored/deleted)

**Collaboration guides added:**
- `COLLABORATION_GUIDE.md` — how frontend and backend developers coordinate
- `FIRST_STEPS.md` — day-by-day checklist for the frontend developer's first week

## Notes for next session
- No code exists yet — everything below Milestone 0 is still theoretical and should be validated
  against reality as it's built (update `enhancements.md` if something in these docs turns out to
  be wrong once real code exists).
