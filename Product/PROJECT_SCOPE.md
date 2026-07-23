# Project Scope

**Project:** ITH Attendance Management System

**Document Version:** 2.0

**Status:** Approved

**Current Phase:** Phase 2 – Presence Monitoring

---

# Purpose

This document defines the functional boundaries of the ITH Attendance Management System.

It clearly specifies:

- What the project includes.
- What the project intentionally excludes.
- What belongs to future development phases.

Any feature outside this scope shall not be implemented unless the documentation is updated and approved.

---

# Project Scope Statement

The ITH Attendance Management System is designed to manage volunteer attendance throughout the lifecycle of an event.

The system covers:

- User authentication
- Event attendance
- Presence monitoring
- Leave management
- Administrative approval workflows
- Attendance reporting

The system is **not** intended to manage volunteer activities, task assignments, or organizational operations beyond attendance during Phase 2.

---

# In Scope

## User Management

The system shall support:

- User authentication
- Role-based access
- Volunteer accounts
- Administrator accounts

---

## Event Management

The system shall support:

- Event creation
- Session management
- Event schedules
- Attendance windows

---

## Venue Management

The system shall support:

- GeoJSON venue uploads
- Venue boundary validation
- Polygon storage
- Venue assignment to events

---

## Attendance Management

The system shall support:

- Check-in
- Continuous presence monitoring
- Leave detection
- Return detection
- Manual check-out
- Automatic check-out

---

## Presence Monitoring

The system shall:

- Continuously monitor volunteer location after successful check-in.
- Detect when a volunteer exits the approved venue boundary.
- Generate presence events.
- Stop monitoring after check-out.

---

## Leave Management

The system shall support:

### Partial Leave

Characteristics:

- Intended for official work only.
- Requires administrator approval.
- Allows the volunteer to return and continue participation after approval.

---

### Complete Leave

Characteristics:

- Indicates the volunteer is leaving the event.
- Requires administrator approval.
- Ends participation for the current event once processed.

---

## Approval Workflow

Administrators shall be able to:

- Review leave requests.
- Approve requests.
- Reject requests.
- Reset volunteer restrictions.

---

## Reporting

The system shall generate reports containing:

- Attendance summary
- Check-in details
- Check-out details
- Presence timeline
- Leave history
- Approval history
- Volunteer restriction history

---

# Out of Scope

The following features are intentionally excluded from Phase 2.

## Activity Management

The system shall not:

- Track volunteer task completion.
- Assign activities.
- Record activity progress.
- Measure productivity.

These capabilities belong to Phase 3.

---

## Volunteer Performance Analytics

The system shall not:

- Calculate volunteer scores.
- Rank volunteers.
- Predict volunteer performance.
- Generate behavioral analytics.

---

## AI Features

The system shall not include:

- AI-based attendance analysis.
- Predictive monitoring.
- Smart recommendations.
- Automated decision making.

---

## Organization Management

The system shall not manage:

- Volunteer recruitment.
- Team formation.
- Committee management.
- Organizational hierarchy.

---

## Communication Features

The system shall not include:

- Chat systems.
- Messaging.
- Broadcast announcements.
- Notification campaigns.

Only operational alerts required for attendance monitoring are included.

---

# Future Scope

Future phases may introduce:

## Phase 3

- Activity Layer
- Activity tracking
- Task assignments
- Activity validation
- Activity reports

---

## Phase 4

- Production optimization
- Performance tuning
- Advanced security
- Deployment improvements
- Scalability enhancements

---

# Scope Control

Any feature request shall be evaluated against this document.

If the requested functionality is:

- Already within scope → proceed through the normal documentation process.
- Outside the approved scope → update documentation before implementation.

Implementation must never expand the project scope without approval.

---

# Acceptance Criteria

This document is considered satisfied when:

- Every implemented feature belongs to the approved project scope.
- Features reserved for future phases are not implemented prematurely.
- Developers and AI agents can determine whether a requested feature is in scope before implementation.

---

# Related Documents

- PRODUCT_VISION.md
- PRD.md
- DEVELOPMENT_ROADMAP.md
- ASSUMPTIONS_AND_CONSTRAINTS.md