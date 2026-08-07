# enhancements.md — Suggested Changes & Deferred Ideas

> When an AI agent (Claude Code, Kiro) or a developer notices a better approach, a missing
> feature, or a piece of Phase 2+ scope while working in Phase 1, it goes **here**, not directly
> into the code. This keeps agents from silently expanding scope or changing architecture
> decisions mid-flight. A developer reviews this list and marks entries Accepted/Rejected/Deferred.

## How to add an entry

Append a new row to the table below (don't rewrite history). Keep the suggestion itself to 1-2
sentences — put longer reasoning in the Rationale column.

| Date | Suggested by | Area | Suggestion | Rationale | Status |
|---|---|---|---|---|---|
| 2026-07-20 | Claude (docs setup) | Backend | Add `check-out` + duration calculation | Full Working Book workflow includes it, but Pitch slide 8 scopes it to Phase 2 | Deferred — Phase 2 |
| 2026-07-20 | Claude (docs setup) | Backend | Add left-venue / returned presence monitoring | Requires periodic location polling, out of Phase 1's single check-in scope | Deferred — Phase 2 |
| 2026-07-20 | Claude (docs setup) | Backend | Add activity submission + reviewer approval flow | Depends on sessions/attendance being stable first (Working Book §7) | Deferred — Phase 3 |
| 2026-07-20 | Claude (docs setup) | Backend | Add offline check-in queue/sync | Needs a working online flow first; adds significant complexity for two new devs | Deferred — Phase 4 |
| 2026-07-20 | Claude (docs setup) | Backend | Add full audit log table (append-only action history) | Rulebook requires it eventually, but Phase 1 already gets minimal traceability via `created_at`/`check_in_time` fields | Deferred — Phase 4 |
| 2026-07-20 | Claude (docs setup) | Backend | Add correction/dispute request workflow | Depends on there being real attendance data to dispute first | Deferred — Phase 4 |
| 2026-07-20 | Claude (docs setup) | Product | Consider adding face-matching to photo evidence | Rulebook §7.2 explicitly says this "must remain subject to consent, legal review, bias testing and human oversight" — not a Phase 1 or even a lightweight decision | Deferred — needs separate legal/product review before any phase |
| 2026-07-20 | Claude (docs setup) | Frontend | Consider adding Flutter mobile client | Explicitly deferred per project decision — web-first for now | Deferred — later phase |

## Status definitions

| Status | Meaning |
|---|---|
| Proposed | Just logged, not yet reviewed |
| Accepted | Approved — move into `development_roadmap.md` under the right phase |
| Rejected | Reviewed and declined — keep the row so it isn't re-suggested without context |
| Deferred — Phase N | Good idea, correct idea even, just not now |


Future Enhancement

Volunteer Progress Tracking

Description:
Allow members to create multiple progress updates while completing an assigned activity.

Potential Features:
- Progress timeline
- Progress journal
- Intermediate status updates
- Progress comments
- Progress-specific evidence
- Administrator progress monitoring
- Real-time activity tracking

Reason Deferred:
Not required for the current volunteer management workflow.
Can be introduced in a future version if operational requirements evolve.