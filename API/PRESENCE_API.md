# Presence API

**Project:** ITH Attendance Management System

**Document Version:** 2.0

**Status:** Approved

---

# Purpose

This document defines the APIs responsible for presence monitoring, venue boundary validation, and leave management during an active attendance session.

Presence monitoring begins after a successful check-in and ends when attendance is completed.

---

# Business Rule References

- Business Rules/ATTENDANCE_POLICY.md
- Business Rules/LEAVE_POLICY.md
- Business Rules/APPROVAL_WORKFLOW.md
- System Design/STATE_MACHINE.md

---

# Resource

```
Presence
```

---

# Update Location

## Purpose

Receive the volunteer's current location during an active attendance session.

The system validates whether the volunteer remains inside the approved venue boundary.

---

### Endpoint

```
POST /presence/location
```

---

### Request Body

Typical information includes:

- Attendance identifier
- Current latitude
- Current longitude
- Timestamp

---

### Successful Response

Returns the current presence status.

Possible outcomes include:

- Inside venue
- Outside venue
- Pending leave action

---

### Validation Rules

The system shall verify:

- User is authenticated.
- Attendance is active.
- Presence monitoring is active.
- Location information is valid.

---

### Possible Errors

- AUTH-001
- ATT-001
- PRE-001
- VALID-003
- STATE-001

---

### State Transition

```
INSIDE_VENUE

↓

OUTSIDE_VENUE
```

---

# Submit Leave Request

## Purpose

Submit a leave request after remaining outside the approved venue boundary.

---

### Endpoint

```
POST /presence/leave-request
```

---

### Request Body

Typical information includes:

- Attendance identifier
- Leave type
- Leave reason
- Additional explanation

---

### Successful Response

The leave request is submitted successfully.

The request enters the Pending Approval workflow.

---

### Validation Rules

The system shall verify:

- Attendance is active.
- Volunteer is eligible to submit a leave request.
- Leave type is valid.
- Leave reason is valid.

---

### Possible Errors

- LEAVE-001
- LEAVE-002
- LEAVE-004
- LEAVE-005
- STATE-001

---

### State Transition

```
OUTSIDE_VENUE

↓

PENDING_APPROVAL
```

---

# Leave Request Status

## Purpose

Retrieve the current status of a submitted leave request.

---

### Endpoint

```
GET /presence/leave-request/{requestId}
```

---

### Successful Response

Returns:

- Leave type
- Leave reason
- Current approval status
- Administrator remarks, where available
- Submission timestamp
- Decision timestamp, where applicable

---

### Validation Rules

The system shall verify:

- User is authenticated.
- Leave request exists.
- User is authorized to access the request.

---

### Possible Errors

- AUTH-001
- AUTHZ-001
- LEAVE-001

---

### State Transition

None.

---

# Presence Timeline

## Purpose

Retrieve the significant presence events recorded during an attendance session.

---

### Endpoint

```
GET /presence/timeline/{attendanceId}
```

---

### Successful Response

Returns the chronological sequence of recorded presence events.

Examples include:

- CHECK_IN
- LEFT
- RETURNED
- CHECK_OUT

---

### Validation Rules

The system shall verify:

- User is authenticated.
- Attendance session exists.
- User is authorized to access the requested attendance session.

---

### Possible Errors

- AUTH-001
- AUTHZ-001
- ATT-001

---

### State Transition

None.

---

# Current Presence Status

## Purpose

Retrieve the volunteer's current presence status for the active attendance session.

---

### Endpoint

```
GET /presence/status
```

---

### Successful Response

Returns the current attendance and presence information.

Typical information includes:

- Current attendance state
- Presence state
- Current venue status
- Active leave request status, if any

---

### Validation Rules

The system shall verify:

- User is authenticated.
- Active attendance exists.

---

### Possible Errors

- AUTH-001
- ATT-001

---

### State Transition

None.

---

# Related Documents

- Business Rules/ATTENDANCE_POLICY.md
- Business Rules/LEAVE_POLICY.md
- Business Rules/APPROVAL_WORKFLOW.md
- System Design/STATE_MACHINE.md
- System Design/VALIDATION_RULES.md
- System Design/ERROR_CODES.md