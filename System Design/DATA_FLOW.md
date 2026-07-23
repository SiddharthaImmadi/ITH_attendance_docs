
# Data Flow

**Project:** ITH Attendance Management System

**Document Version:** 2.0

**Status:** Approved

---

# Purpose

This document defines how information flows through the Attendance Management System.

It describes the movement of data between users, system components, and persistent storage while remaining independent of implementation technologies.

This document complements the State Machine by describing the flow of information rather than state transitions.

---

# Objectives

The data flow design aims to:

- Clearly define the origin of every significant piece of information.
- Show how data is validated and processed.
- Ensure traceability of attendance events.
- Support future scalability without changing business behavior.

---

# High-Level Data Flow

```
Volunteer

    │

    ▼

User Interface

    │

    ▼

Application Layer

    │

    ├───────────────┐
    ▼               ▼

Validation      Business Rules

    │               │
    └──────┬────────┘
           ▼

Business Services

(Authentication
Attendance
Presence Monitoring
Leave Management
Administration)

           │

           ▼

Data Persistence

           │

           ▼

Reports / History
```

---

# Check-in Flow

```
Volunteer

↓

Authentication

↓

Attendance Validation

↓

Location Validation

↓

Identity Verification

↓

Attendance Created

↓

Presence Monitoring Starts

↓

Attendance History Updated
```

---

# Presence Monitoring Flow

```
Presence Monitoring

↓

Receive Current Location

↓

Validate Against Venue Boundary

↓

Inside Venue?

      │

 ┌────┴────┐

 │         │

Yes       No

 │         │

 │   Start Outside Timer

 │         │

 │   Returned?

 │         │

 │     ┌───┴────┐

 │     │        │

 │    Yes      No

 │     │        │

 │     ▼        ▼

 │  Record    10 Seconds

 │  Return        │

 │                ▼

 └──────────► Leave Alert
```

---

# Leave Request Flow

```
Volunteer

↓

Leave Alert

↓

Select Leave Type

↓

Select Leave Reason

↓

Provide Explanation

↓

Submit Leave Request

↓

Pending Approval

↓

Administrator Review

↓

Approve / Reject

↓

Attendance History Updated
```

---

# Approval Flow

```
Pending Approval

↓

Administrator Review

↓

Decision

↓

Approve
Reject

↓

Attendance Updated

↓

Volunteer Notified

↓

Reports Updated
```

---

# Check-out Flow

```
Volunteer

↓

Manual Check-out

        OR

Automatic Check-out

↓

Attendance Closed

↓

Presence Monitoring Stops

↓

Attendance Timeline Finalized

↓

Reports Updated
```

---

# Reporting Flow

Reports obtain information from:

- Attendance records.
- Presence events.
- Leave requests.
- Administrative decisions.
- Volunteer restrictions.

Reports are read-only and shall never modify operational data.

---

# Validation Flow

Every user request follows the same validation sequence.

```
Incoming Request

↓

Authentication

↓

Authorization

↓

Business Rule Validation

↓

Input Validation

↓

Processing

↓

Persistence

↓

Response
```

---

# Audit Flow

Every significant business event shall generate an audit record.

Examples include:

- Successful Check-in
- Venue Exit
- Return to Venue
- Leave Request Submission
- Leave Approval
- Leave Rejection
- Restriction Applied
- Restriction Removed
- Check-out

Audit records support reporting and administrative review.

---

# Data Ownership

Each business service owns its operational data.

| Service | Primary Responsibility |
|----------|------------------------|
| Authentication | User identity and session validation |
| Attendance | Attendance lifecycle |
| Presence Monitoring | Presence events |
| Leave Management | Leave requests |
| Administration | Administrative decisions and restrictions |
| Reporting | Read-only reporting and exports |

Services shall not directly modify data owned by another service except through defined business operations.

---

# Error Handling

If validation fails at any stage:

- Processing stops immediately.
- No partial business operation shall be committed.
- An appropriate error response shall be returned.
- Existing attendance data shall remain unchanged.

---

# Related Documents

- SYSTEM_ARCHITECTURE.md
- STATE_MACHINE.md
- DATABASE_SCHEMA.md
- VALIDATION_RULES.md