
# Attendance API

**Project:** ITH Attendance Management System

**Document Version:** 2.0

**Status:** Approved

---

# Purpose

This document defines the Attendance APIs responsible for managing the attendance lifecycle of volunteers.

Attendance begins with a successful check-in and ends with a successful check-out.

---

# Business Rule References

- Business Rules/ATTENDANCE_POLICY.md
- System Design/STATE_MACHINE.md

---

# Resource

```
Attendance
```

---

# Check-in

## Purpose

Create a new attendance session for the authenticated volunteer.

---

### Endpoint

```
POST /attendance/check-in
```

---

### Request Body

Typical information includes:

- Session identifier
- Current location
- Face verification data

---

### Successful Response

The volunteer is successfully checked in.

Attendance becomes active and presence monitoring begins.

---

### Validation Rules

The system shall verify:

- User is authenticated.
- Volunteer is eligible.
- Event session is active.
- No active attendance exists.
- Volunteer is inside the approved venue.
- Identity verification succeeds.

---

### Possible Errors

- AUTH-001
- ATT-002
- ATT-005
- VEN-001
- VALID-001
- STATE-001

---

### State Transition

```
NOT_CHECKED_IN

↓

INSIDE_VENUE
```

---

# Check-out

## Purpose

End the active attendance session.

---

### Endpoint

```
POST /attendance/check-out
```

---

### Successful Response

Attendance is completed.

Presence monitoring stops.

Attendance history is finalized.

---

### Validation Rules

The system shall verify:

- Attendance is active.
- Attendance has not already ended.

---

### Possible Errors

- ATT-001
- ATT-003
- STATE-001

---

### State Transition

```
INSIDE_VENUE

↓

CHECKED_OUT
```

---

# Attendance History

## Purpose

Retrieve attendance history for the authenticated volunteer.

---

### Endpoint

```
GET /attendance/history
```

---

### Successful Response

Returns completed and active attendance sessions for the authenticated volunteer.

---

### Validation Rules

- User is authenticated.

---

### Possible Errors

- AUTH-001
- AUTH-002

---

### State Transition

None.

---

# Attendance Details

## Purpose

Retrieve a specific attendance session.

---

### Endpoint

```
GET /attendance/{attendanceId}
```

---

### Successful Response

Returns detailed attendance information including:

- Attendance timeline
- Presence events
- Leave history
- Administrative decisions

---

### Validation Rules

The system shall verify:

- User is authenticated.
- Attendance exists.
- User is authorized to access the requested attendance.

---

### Possible Errors

- AUTH-001
- AUTHZ-001
- ATT-001

---

### State Transition

None.

---

# Related Documents

- Business Rules/ATTENDANCE_POLICY.md
- System Design/STATE_MACHINE.md
- System Design/VALIDATION_RULES.md
- System Design/ERROR_CODES.md