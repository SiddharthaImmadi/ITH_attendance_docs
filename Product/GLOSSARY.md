# Glossary

**Project:** ITH Attendance Management System

**Document Version:** 2.0

**Status:** Approved

---

# Purpose

This document defines the terminology used throughout the project.

Every contributor, reviewer, and AI coding agent should interpret these terms consistently.

Whenever a term defined here appears in another document, its meaning shall be the one described in this glossary.

---

# General Terms

## Attendance

The recorded participation of a volunteer during an event.

Attendance begins with a successful check-in and ends with a successful check-out.

---

## Attendance Session

The attendance lifecycle of a volunteer for a specific event.

---

## Event

An organized volunteering program during which attendance is monitored.

---

## Volunteer

A registered user participating in an event.

---

## Administrator

A user responsible for managing events, attendance, leave requests, reports and volunteer restrictions.

---

# Attendance Terms

## Check-in

The successful verification of a volunteer's attendance.

Requirements include:

- Authentication
- GPS validation
- Face verification

Check-in starts presence monitoring.

---

## Check-out

The action that ends an attendance session.

Check-out may occur:

- Manually
- Automatically after the configured grace period

---

## Presence Monitoring

Continuous monitoring of volunteer location after check-in.

Monitoring stops immediately after check-out.

---

## Presence Event

A significant attendance event.

Examples:

- CHECK_IN
- LEFT
- RETURNED
- CHECK_OUT

---

## Presence Timeline

A chronological history of all presence events generated during an attendance session.

---

# Venue Terms

## Venue

The approved geographical area in which volunteers are expected to remain during an event.

---

## GeoJSON

A standard geographic file format used to define venue boundaries.

Administrators upload GeoJSON files to define valid attendance areas.

---

## Polygon

The closed geographical boundary extracted from a GeoJSON file.

Presence monitoring determines whether volunteers remain inside this polygon.

---

# Leave Management

## Partial Leave

A temporary exit from the venue for official work.

Characteristics:

- Requires administrator approval.
- Volunteer is expected to return.

---

## Complete Leave

A permanent exit from the current event.

Characteristics:

- Requires administrator approval.
- Ends further participation in the current event.

---

## Leave Request

A request submitted by a volunteer after leaving the venue.

A request includes:

- Leave type
- Selected reason
- Additional explanation

---

## Leave Reason

The predefined reason selected by a volunteer when requesting leave.

---

# Administrative Terms

## Approval

Administrator decision accepting a leave request.

---

## Rejection

Administrator decision rejecting a leave request.

---

## Volunteer Restriction

A temporary restriction preventing a volunteer from joining volunteering opportunities.

Restrictions may be removed by an administrator.

---

# Reporting Terms

## Attendance Report

A report summarizing attendance information.

---

## Presence Report

A report containing presence events recorded during an attendance session.

---

## Leave History

Historical record of submitted leave requests.

---

## Restriction History

Historical record of volunteer restrictions.

---

# Development Terms

## Phase

A major development stage of the project.

---

## Milestone

A measurable implementation target within a phase.

---

## Acceptance Criteria

Conditions that must be satisfied before a feature is considered complete.

---

## Exit Criteria

Conditions that must be satisfied before a milestone may be closed.

---

## Definition of Done

The complete set of requirements that determine whether implementation is finished.

---

# AI Development Terms

## Documentation First

Documentation is approved before implementation begins.

---

## Single Source of Truth

Repo A contains the authoritative project documentation.

Repo B contains approved copies required during implementation.

---

## Backward Compatibility

Previously approved APIs and functionality shall continue working after introducing new features.

---

# Related Documents

- PRODUCT_VISION.md
- PRD.md
- PROJECT_SCOPE.md