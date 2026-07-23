# Product Requirements Document (PRD)

**Project:** ITH Attendance Management System

**Version:** 2.0

**Status:** Approved

**Current Phase:** Phase 2 – Presence Monitoring

---

# Purpose

This document defines the functional and non-functional requirements of the ITH Attendance Management System.

It serves as the primary reference for all implementation activities and is the authoritative source for system behavior.

Any feature not described in this document must not be implemented without prior approval.

---

# Product Overview

The ITH Attendance Management System is a location-aware attendance platform designed for volunteer-based events.

The system verifies volunteer attendance using authentication, GPS validation, and facial verification. After successful check-in, it continuously monitors volunteer presence within the event venue and records significant attendance events until check-out.

The platform emphasizes accountability, transparency, and scalability while maintaining backward compatibility across development phases.

---

# Goals

The system shall:

- Securely authenticate volunteers and administrators.
- Support event-based attendance.
- Verify volunteer identity.
- Validate volunteer location.
- Monitor volunteer presence throughout an event.
- Record attendance events.
- Support administrator-controlled leave approvals.
- Generate attendance reports.
- Preserve historical attendance records.
- Remain extensible for future phases.

---

# Stakeholders

## Product Owner

Responsible for defining business requirements and approving changes.

---

## Administrator

Responsible for:

- Creating events
- Managing sessions
- Uploading GeoJSON venue boundaries
- Reviewing attendance
- Reviewing leave requests
- Managing volunteer restrictions
- Exporting reports

---

## Volunteer

Responsible for:

- Authentication
- Check-in
- Remaining within the venue
- Submitting leave requests
- Check-out

---

# Functional Requirements

## FR-1 Authentication

The system shall authenticate users before granting access.

---

## FR-2 Session Management

The system shall support event sessions that define attendance windows.

---

## FR-3 Check-in

The system shall:

- Verify authentication.
- Validate GPS location.
- Verify face detection.
- Prevent duplicate check-ins.
- Record attendance.

---

## FR-4 Presence Monitoring

After successful check-in:

- Continuous location monitoring shall begin.
- Monitoring shall continue in the background.
- Presence events shall be recorded.
- Monitoring shall stop after check-out.

---

## FR-5 Venue Monitoring

The system shall determine whether the volunteer remains inside the approved GeoJSON venue boundary.

If the volunteer exits the boundary for more than 10 seconds:

- Generate a leave event.
- Notify the volunteer.

---

## FR-6 Leave Management

The system shall support two leave types:

### Partial Leave

Characteristics:

- Official work only.
- Requires administrator approval.
- Volunteer may return and continue participation.

---

### Complete Leave

Characteristics:

- Volunteer leaves the event.
- Requires administrator approval.
- Ends further participation for the current event once processed.

---

## FR-7 Leave Reason Submission

The system shall require:

- Leave type selection.
- Reason selection from predefined options.
- Additional explanation using free text.

---

## FR-8 Approval Workflow

Administrators shall be able to:

- View leave requests.
- Approve requests.
- Reject requests.

Approval decisions shall be recorded.

---

## FR-9 Volunteer Restrictions

Rejected complete leave requests may result in:

- Temporary volunteering restriction.
- Restriction removal by administrator.

Repeated violations may result in stricter administrative action.

---

## FR-10 Check-out

The system shall support:

### Manual Check-out

Volunteer ends attendance.

---

### Automatic Check-out

If the volunteer does not manually check out:

- Attendance shall automatically close after the event end time plus the configured grace period.
- Presence monitoring shall stop.
- No further presence events shall be accepted.

---

## FR-11 Reports

The system shall generate reports including:

- Attendance summary
- Check-in details
- Check-out details
- Presence timeline
- Leave history
- Approval history
- Volunteer restriction history

---

# Non-Functional Requirements

## Performance

- GPS processing shall be efficient.
- Presence monitoring shall support concurrent volunteers.
- Reports shall be generated within acceptable response times.

---

## Reliability

The system shall preserve attendance data even if temporary network interruptions occur.

---

## Security

- JWT authentication.
- Role-based authorization.
- Secure password storage.
- Protected administrative actions.

---

## Scalability

Future phases shall extend functionality without redesigning Phase 1 features.

---

## Maintainability

Implementation shall follow the documented architecture.

---

## Compatibility

Existing approved APIs shall remain backward compatible.

New functionality shall be introduced through new endpoints.

---

# Out of Scope

The following are intentionally excluded from Phase 2:

- Activity Layer
- Volunteer scoring
- AI analytics
- Push notification optimization
- Multi-event attendance analysis
- Predictive insights

These features are reserved for later phases.

---

# Acceptance Criteria

The PRD is considered satisfied when:

- All documented functional requirements are implemented.
- Non-functional requirements are met.
- API contracts remain compatible.
- State transitions follow the documented workflow.
- Business rules are enforced.
- Required reports are generated.
- Backend and frontend satisfy milestone completion criteria.

---

# Related Documents

- PRODUCT_VISION.md
- PROJECT_SCOPE.md
- DEVELOPMENT_ROADMAP.md
- STATE_MACHINE.md
- API_CONTRACT.md
- BUSINESS_RULES.md