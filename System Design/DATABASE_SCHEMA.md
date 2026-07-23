
# Database Schema

**Project:** ITH Attendance Management System

**Document Version:** 2.0

**Status:** Approved

---

# Purpose

This document defines the logical data model for the ITH Attendance Management System.

It identifies the core entities, their responsibilities, relationships, and business ownership.

This document is technology-independent and describes the logical schema rather than any specific database implementation.

---

# Objectives

The database schema shall:

- Store all attendance-related information.
- Preserve historical records.
- Support reporting and auditing.
- Maintain data integrity.
- Support future project phases without redesigning existing entities.

---

# Entity Overview

| Entity | Purpose |
|---------|---------|
| User | Registered system users |
| Role | User authorization roles |
| Event | Volunteer events |
| Venue | Event venue information |
| Venue Boundary | GeoJSON boundary definition |
| Attendance | Volunteer attendance sessions |
| Presence Event | Significant attendance events |
| Leave Request | Volunteer leave requests |
| Leave Reason | Predefined leave reasons |
| Restriction | Volunteer participation restrictions |
| Audit Log | Historical system actions |

---

# Entity Descriptions

## User

Represents an authenticated system user.

### Responsibilities

- Identity
- Authentication
- Role assignment
- Volunteer profile

---

## Role

Defines system permissions.

Examples:

- Volunteer
- Administrator

A user may have one or more assigned roles according to organizational policy.

---

## Event

Represents an organized volunteering event.

An event defines:

- Name
- Schedule
- Venue
- Attendance window

---

## Venue

Represents the physical location associated with an event.

A venue contains one approved geographical boundary.

---

## Venue Boundary

Defines the approved attendance area.

Characteristics:

- GeoJSON polygon
- Version controlled
- Associated with one venue

The venue boundary is used during presence monitoring.

---

## Attendance

Represents one volunteer's attendance session for one event.

Stores:

- Check-in information
- Check-out information
- Current attendance state
- Attendance timeline reference

One attendance record shall represent one volunteer participating in one event.

---

## Presence Event

Represents significant attendance events.

Examples include:

- CHECK_IN
- LEFT
- RETURNED
- CHECK_OUT

Continuous location history shall not be stored.

Only significant events are recorded.

---

## Leave Request

Represents a volunteer's request to leave the venue.

Stores:

- Leave type
- Leave reason
- Additional explanation
- Current approval status
- Administrator decision

---

## Leave Reason

Represents predefined reasons available during leave request submission.

Reasons are managed administratively.

---

## Restriction

Represents a temporary volunteering restriction.

Stores:

- Restriction reason
- Start date
- End date (if applicable)
- Current status
- Administrative remarks

---

## Audit Log

Stores significant business actions.

Examples:

- Check-in
- Check-out
- Leave submission
- Leave approval
- Restriction applied
- Restriction removed

Audit records shall be immutable.

---

# Entity Relationships

```
User
 │
 ├──────────────┐
 │              │
 ▼              ▼

Role        Attendance
                 │
                 │
                 ▼
             Presence Event

Attendance
      │
      ▼
Leave Request
      │
      ▼
Leave Reason

User
 │
 ▼
Restriction

Event
 │
 ▼
Venue
 │
 ▼
Venue Boundary

Attendance
 │
 ▼
Event
```

---

# Ownership Rules

| Entity | Business Owner |
|----------|----------------|
| User | Authentication |
| Role | Authentication |
| Event | Administration |
| Venue | Administration |
| Venue Boundary | Administration |
| Attendance | Attendance Service |
| Presence Event | Presence Monitoring |
| Leave Request | Leave Management |
| Leave Reason | Administration |
| Restriction | Administration |
| Audit Log | Audit Service |

---

# Historical Data

The system shall preserve historical records for:

- Attendance
- Presence events
- Leave requests
- Administrative decisions
- Restrictions
- Audit logs

Historical records shall not be deleted as part of normal system operations.

---

# Data Integrity Principles

The schema shall ensure:

- Each attendance session belongs to one volunteer and one event.
- Presence events belong to exactly one attendance session.
- Leave requests belong to exactly one attendance session.
- Restrictions belong to one volunteer.
- Venue boundaries belong to one venue.

---

# Extensibility

Future phases may introduce additional entities without modifying existing business concepts.

Examples include:

- Activity
- Activity Assignment
- Activity Completion
- Performance Metrics

Existing relationships should remain stable wherever possible.

---

# Related Documents

- SYSTEM_ARCHITECTURE.md
- STATE_MACHINE.md
- DATA_FLOW.md
- VALIDATION_RULES.md