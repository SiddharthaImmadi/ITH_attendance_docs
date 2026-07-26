# PRD.md — Product Requirements Document (Phase 1 / Core MVP)

> Scope: this PRD covers **Phase 1 only**, as defined on the pitch deck's roadmap slide. Phases
> 2–4 are summarized at the bottom for context but are explicitly **out of scope** for current
> development. See `development_roadmap.md` for how phases sequence over time.

## 1. Product summary

A web application where an **Admin** creates attendance sessions (title, date, time window, venue
coordinates + radius) and a **Member** checks in to an open session by submitting their GPS
location and a live-captured photo. The system validates the submission server-side and produces
an Excel export of the results.

> "Best first version: Admin + Member + GPS check-in + live photo + Excel export." — Pitch, slide 8

## 2. Goals (Phase 1)

1. Replace manual sign-in sheets with a verified digital check-in.
2. Prove a member was physically at the venue at check-in time (GPS geofence).
3. Prove the check-in was a live action, not a forwarded photo (live camera capture).
4. Give admins a clean, exportable record of who checked in, when, and from where.

## 3. Non-goals (Phase 1)

Explicitly deferred — do not build these now (see `enhancements.md` if an agent proposes one):

- Live/continuous venue monitoring, "left venue" / "returned" tracking (Phase 2)
- Check-out, duration calculation (Phase 2)
- Member activity submissions and review (Phase 3)
- Offline mode / sync queue (Phase 4)
- Audit logs, spoofing/anti-fraud detection beyond basic duplicate-check blocking (Phase 4)
- Corrections/dispute workflow (Phase 4)
- Flutter mobile app (later phase — Phase 1 is web-only)
- Any AI-powered features (pitch explicitly states "No AI API is required for the MVP")

## 4. User roles (Phase 1)

Per Rulebook §3 and Working Book §2, the full role model has 6 actors; Phase 1 only needs two:

| Role | Phase 1 permissions |
|---|---|
| **Admin** | Log in, create a session (title, date, start/end time, venue lat/long, radius), view list of sessions, view check-ins for a session, export Excel report. |
| **Member** | Log in, view open session(s) they're eligible for, submit check-in (GPS + live photo), view their own check-in confirmation/status. |

Coordinator, Auditor, and System Administrator roles from the source docs are **not** implemented
in Phase 1.

## 5. Functional requirements

### 5.1 Authentication
- FR-1: Admin and Member can log in with credentials (email/username + password).
- FR-2: Passwords are never stored in plaintext (see `rules_backend.md`).
- FR-3: Sessions are authenticated via JWT (see `API_contract.md`).
- FR-4: A logged-in user only sees data/actions permitted for their role.

### 5.2 Session management (Admin)
- FR-5: Admin can create a session with: title, purpose (optional text), date, start time, end
  time, grace period (minutes), venue latitude/longitude, allowed radius (meters).
- FR-6: A session has a status: `Scheduled` → `Open` → `Closed` (simplified from the full state
  model in Working Book §3.2 — Left Venue/Returned/Checked Out states are Phase 2).
- FR-7: Admin can view a list of their sessions and each session's checked-in members.
- FR-8: Admin can manually close a session.

### 5.3 Check-in (Member)
- FR-9: Member can see sessions currently in `Open` status that they're eligible for.
- FR-10: Member check-in requires: device GPS coordinates + a live-captured photo (no gallery
  upload — Rulebook §7.2).
- FR-11: Server calculates distance from venue coordinates (Haversine formula, Working Book §6.1)
  and accepts the check-in only if within `radius` (+ a small configurable accuracy tolerance).
- FR-12: A check-in outside the radius is rejected with a clear reason, or stored as `Pending
  Verification` if the rejection is due to poor GPS accuracy rather than confirmed distance
  (Working Book §6.2).
- FR-13: Check-in is only accepted while the session is `Open` and within its time window; a
  check-in after start time + grace period is marked `Late` rather than `Present`.
- FR-14: A member cannot check in twice for the same session (duplicate-check, Rulebook §6.1.5).
- FR-15: All timestamps used for validation are server time, not client-submitted time (Rulebook
  §7.3 "Device timestamp mismatch → use trusted server time").

### 5.4 Reporting
- FR-16: Admin can export an Excel workbook for a session containing at minimum an **Attendance
  Summary** sheet (Member ID, Name, Session, Date, Check-in time, Distance, Final Status) and a
  **Photo Evidence** reference sheet (Working Book §11.2, scoped down from the full 6-sheet
  structure — Presence Events/Activities/Corrections sheets are Phase 2+).
- FR-17: Exports are generated on demand from the PostgreSQL database (not hand-maintained files).

## 6. Non-functional requirements

| Area | Requirement |
|---|---|
| Security | Role-based access; passwords hashed; JWT-protected endpoints; no secrets in client code or in Repo A. |
| Privacy | Location and photo evidence only collected for sessions that require it; not exposed to any role beyond Admin. |
| Reliability | Duplicate check-in attempts must never create a second record. |
| Usability | A first-time member should be able to check in with visible, clear guidance and error messages (Working Book §14.2). |
| Auditability (lightweight) | Every check-in record stores who/when/where — full audit-log tooling is Phase 4, but the underlying data must not be silently overwritten. |

## 7. Acceptance criteria (from Working Book §14.1, filtered to Phase 1)

| ID | Scenario | Expected result |
|---|---|---|
| F01 | Eligible member checks in inside radius and within time | Check-in recorded once, correct timestamp, status Present/Late as appropriate |
| F02 | Member attempts duplicate check-in | Duplicate blocked; original record unchanged |
| F03 | Member checks in outside radius | Rejected or `Pending Verification` per rule |
| F04 | GPS accuracy is poor | Retry prompt or `Pending Verification`, not silent accept/reject |
| F10 | Excel report generated | Totals and columns match what's in the database for that session |

F05–F09 (left/return, early checkout, offline sync, corrections, unauthorized report access) are
Phase 2–4 acceptance criteria and are listed here only so they aren't accidentally lost — track
them in `development_roadmap.md`.

## 8. Phase 2–4 summary (context only, not current scope)

| Phase | Focus |
|---|---|
| Phase 2 — Monitoring | Inside/outside tracking, left-venue alerts, check-out, duration calculation |
| Phase 3 — Activity layer | Member activity updates, evidence review, filters, extra report sheets |
| Phase 4 — Production hardening | Offline sync, audit logs, spoofing checks, backups, deployment |

Full detail in `development_roadmap.md`.

## 9. Phase 2 Product Requirements

> Scope: This section extends the original Phase 1 PRD. All Phase 1 functional and non-functional
> requirements remain valid unless explicitly superseded in future phases.

Phase 2 expands the Attendance & Activity Tracking Application from a verified attendance system
into a basic attendance monitoring system. The objective is to provide administrators with better
visibility into member presence throughout an event while preserving the simple and reliable
attendance workflow established during Phase 1.

### 9.1 Goals (Phase 2)

1. Extend attendance beyond a single check-in event.
2. Record when a member leaves and returns to the event venue.
3. Calculate the total duration of attendance for each member.
4. Provide administrators with a more complete attendance timeline.

### 9.2 Functional Requirements (Phase 2)

#### Attendance Monitoring

- FR-18: The system shall allow members to check out of an active session.
- FR-19: The system shall calculate attendance duration using server-side timestamps.
- FR-20: The system shall record when a member leaves the approved attendance area.
- FR-21: The system shall record when a member re-enters the approved attendance area.
- FR-22: A member's attendance timeline shall preserve the chronological order of all attendance events.
- FR-23: Administrators shall be able to view the complete attendance timeline for every member participating in a session.

#### Session Status Extension

The simplified attendance lifecycle introduced in Phase 1 is extended to include additional
attendance events required for monitoring.

The detailed business rules, status transitions, and validation logic are documented separately
within the project rule documentation and implementation documents.

### 9.3 Reporting

Phase 2 reporting extends the Phase 1 attendance report by incorporating attendance duration and
additional attendance events where applicable.

The detailed report structure will be defined within the reporting documentation during
implementation.

### 9.4 Non-Goals (Phase 2)

The following capabilities remain outside the scope of Phase 2:

- Member activity submissions.
- Activity evidence management.
- Offline synchronization.
- Advanced anti-spoofing detection.
- Complete audit logging.
- AI-assisted features.

These remain scheduled for later phases as described in
`development_roadmap.md`.

### 9.5 Acceptance Criteria (Phase 2)

| ID | Scenario | Expected Result |
|---|---|---|
| F11 | Member checks out normally | Attendance duration is calculated correctly |
| F12 | Member leaves the venue | Leave event is recorded |
| F13 | Member returns to the venue | Return event is recorded chronologically |
| F14 | Administrator views attendance timeline | Complete sequence of attendance events is displayed |
| F15 | Attendance report generated | Report includes Phase 2 attendance information where applicable |
