# Assumptions and Constraints

**Project:** ITH Attendance Management System

**Document Version:** 2.0

**Status:** Approved

---

# Purpose

This document records long-term assumptions, design decisions, and constraints that guide development.

These constraints prevent inconsistent implementations and reduce ambiguity for contributors and AI coding agents.

---

# Project Assumptions

## Documentation First

All implementation shall follow approved documentation.

Documentation is the primary source of truth.

---

## AI Assisted Development

Backend implementation is performed using Kiro.

Frontend implementation is performed using Claude Code.

---

## Repository Structure

Repo A

Purpose:

Authoritative documentation repository.

Repo B

Purpose:

Implementation repository.

Approved documentation is copied from Repo A into Repo B before implementation.

---

## Incremental Development

The project is developed incrementally through phases and milestones.

Large, unreviewed implementations are discouraged.

---

# Technical Constraints

## API Stability

Approved APIs shall remain backward compatible.

Existing endpoints shall not be modified unless explicitly approved.

New functionality should be introduced using new endpoints.

---

## GeoJSON

Venue boundaries shall be defined using uploaded GeoJSON files.

The backend shall validate volunteer location against the approved polygon.

---

## Continuous Monitoring

Presence monitoring begins immediately after successful check-in.

Monitoring continues while the application runs in the foreground or background.

Monitoring ends after check-out.

---

## Automatic Check-out

If a volunteer does not manually check out:

- Attendance remains active until the event end time plus the configured grace period.
- The attendance session closes automatically.
- Presence monitoring stops.
- Additional presence events are rejected.

---

# Business Constraints

## Partial Leave

Reserved for official work.

Requires administrator approval.

Volunteer is expected to return.

---

## Complete Leave

Represents leaving the event.

Requires administrator approval.

Ends participation in the current event.

---

## Volunteer Restrictions

Rejected leave requests may result in temporary volunteering restrictions.

Only administrators may remove restrictions.

Repeated violations may result in stronger administrative action.

---

# Documentation Constraints

Frozen documentation shall not be modified during implementation.

Suggested improvements shall be documented separately and reviewed before approval.

---

# Development Constraints

Implementation shall:

- Read documentation before coding.
- Explain implementation.
- Wait for approval.
- Implement only approved scope.
- Execute relevant tests.
- Stop after milestone completion.

---

# Future Constraints

Phase 2 shall not introduce functionality reserved for Phase 3.

The Activity Layer shall remain completely independent from Presence Monitoring.

---

# Related Documents

- PRD.md
- PROJECT_SCOPE.md
- DEVELOPMENT_ROADMAP.md