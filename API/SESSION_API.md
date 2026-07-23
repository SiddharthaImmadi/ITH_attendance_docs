
# Session API

**Project:** ITH Attendance Management System

**Document Version:** 2.0

**Status:** Approved

---

# Purpose

This document defines the Session APIs used to manage volunteering event sessions.

An event session represents the time period during which volunteers may participate in attendance activities.

---

# Business Rule References

- Product/PRD.md
- Business Rules/ATTENDANCE_POLICY.md
- Business Rules/RULEBOOK.md

---

# Session Overview

An event session controls when attendance operations are permitted.

Attendance-related operations such as check-in and check-out are only valid while an event session is active.

---

# Resource

```
Session
```

---

# Get Active Sessions

## Purpose

Retrieve all currently active event sessions available to the authenticated user.

---

### Endpoint

```
GET /sessions
```

---

### Successful Response

Returns a list of active event sessions.

Each session includes information such as:

- Session identifier
- Event name
- Venue
- Start time
- End time
- Current session status

---

### Validation Rules

The system shall verify:

- User is authenticated.

---

### Possible Errors

- AUTH-001
- AUTH-002

---

### State Transition

None.

---

# Get Session Details

## Purpose

Retrieve detailed information about a specific event session.

---

### Endpoint

```
GET /sessions/{sessionId}
```

---

### Successful Response

Returns detailed information for the requested session.

Information may include:

- Event details
- Venue information
- Attendance availability
- Session schedule

---

### Validation Rules

The system shall verify:

- User is authenticated.
- Requested session exists.

---

### Possible Errors

- AUTH-001
- AUTH-002
- VALID-002

---

### State Transition

None.

---

# Create Session

## Purpose

Create a new volunteering event session.

---

### Endpoint

```
POST /sessions
```

---

### Request Body

Typical information includes:

- Event name
- Session date
- Start time
- End time
- Venue
- Venue boundary

---

### Successful Response

A new event session is created and becomes available according to its configured schedule.

---

### Validation Rules

The system shall verify:

- User has administrator privileges.
- Required information is provided.
- Venue information is valid.
- Session schedule is valid.

---

### Possible Errors

- AUTH-001
- AUTHZ-002
- VALID-001
- VALID-002

---

### State Transition

None.

---

# Update Session

## Purpose

Update an existing event session.

---

### Endpoint

```
PUT /sessions/{sessionId}
```

---

### Validation Rules

The system shall verify:

- User has administrator privileges.
- Session exists.
- Updated information is valid.

---

### Possible Errors

- AUTH-001
- AUTHZ-002
- VALID-001
- VALID-002

---

### State Transition

None.

---

# Close Session

## Purpose

Close an active event session.

Closing a session prevents new attendance check-ins.

Existing attendance sessions continue according to the Attendance Policy.

---

### Endpoint

```
POST /sessions/{sessionId}/close
```

---

### Validation Rules

The system shall verify:

- User has administrator privileges.
- Session exists.
- Session is currently active.

---

### Possible Errors

- AUTH-001
- AUTHZ-002
- VALID-002

---

### State Transition

No direct attendance state transition occurs.

Future check-in requests for the session shall be rejected.

---

# Related Documents

- Business Rules/ATTENDANCE_POLICY.md
- System Design/VALIDATION_RULES.md
- System Design/ERROR_CODES.md
- System Design/SECURITY_MODEL.md