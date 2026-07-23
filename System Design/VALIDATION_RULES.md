
# Validation Rules

**Project:** ITH Attendance Management System

**Document Version:** 2.0

**Status:** Approved

---

# Purpose

This document defines the validation rules that govern all system operations.

These rules ensure that only valid business operations are processed and that data integrity is maintained throughout the attendance lifecycle.

Validation rules apply regardless of the implementation technology.

---

# Validation Principles

Every system operation shall:

- Validate the request before processing.
- Reject invalid operations.
- Preserve existing data integrity.
- Return a clear validation result.
- Prevent partial completion of business operations.

---

# Validation Flow

Every operation shall follow this validation sequence.

1. Authentication Validation
2. Authorization Validation
3. Business Rule Validation
4. Input Validation
5. State Validation
6. Processing
7. Persistence

If any validation fails, processing shall stop immediately.

---

# Authentication Validation

Authentication validation confirms the identity of the requesting user.

The system shall verify:

- The user is authenticated.
- The session is valid.
- The authentication token is valid.

If authentication fails:

- The request shall be rejected.
- No business processing shall occur.

---

# Authorization Validation

Authorization validation confirms that the authenticated user has permission to perform the requested action.

Examples include:

- Volunteers may submit leave requests.
- Administrators may approve leave requests.
- Administrators may manage restrictions.
- Volunteers shall not perform administrative actions.

---

# Attendance Validation

Before attendance-related processing, the system shall verify:

- The volunteer is eligible.
- The event is active.
- The attendance session is valid.
- Duplicate attendance does not exist.

---

# Check-in Validation

A check-in request shall be accepted only when:

- The volunteer is authenticated.
- The volunteer is not restricted.
- The event session is open.
- No active attendance session exists.
- The volunteer is within the approved venue boundary.
- Identity verification succeeds.

Otherwise, the request shall be rejected.

---

# Presence Monitoring Validation

Presence monitoring shall verify:

- Attendance is active.
- Presence monitoring has started.
- Venue boundary information exists.
- The received location is valid.

If monitoring requirements are not satisfied, location updates shall not affect attendance.

---

# Venue Validation

Every received location shall be validated against the approved venue boundary.

Possible outcomes:

- Inside venue
- Outside venue

If outside:

- The outside timer begins.
- Returning before the threshold resets the timer.

---

# Leave Request Validation

A leave request shall be accepted only when:

- Attendance is active.
- The volunteer is in the Pending Approval workflow.
- A valid leave type is selected.
- A valid leave reason is selected.
- Required explanation is provided.

Duplicate active leave requests shall not be permitted.

---

# Approval Validation

Before processing a decision, the system shall verify:

- The requester is an administrator.
- The leave request exists.
- The request is still pending.
- The request has not already been decided.

---

# Check-out Validation

Check-out shall be accepted only when:

- Attendance is active.
- The attendance session has not already ended.

Manual check-out and automatic check-out shall produce the same final attendance state.

---

# Restriction Validation

Before allowing a volunteer to participate in an event, the system shall verify whether an active volunteering restriction exists.

If a restriction is active:

- Event participation shall not begin.
- Attendance shall not be created.

---

# State Validation

Every operation shall verify that the requested state transition is valid according to the State Machine.

Invalid transitions shall be rejected.

Examples include:

- Check-out before check-in.
- Return after attendance has ended.
- Approving an already approved request.
- Rejecting a completed attendance session.

---

# Data Validation

The system shall validate:

- Required fields.
- Accepted value ranges.
- Enumeration values.
- Timestamp consistency.
- Geographic coordinate format.

Invalid data shall not be persisted.

---

# Historical Validation

Historical records shall be:

- Immutable.
- Chronologically consistent.
- Associated with the correct attendance session.

Historical records shall never be overwritten.

---

# Validation Failure

When validation fails:

- Processing stops immediately.
- No partial business operation shall be committed.
- Existing records remain unchanged.
- An appropriate error shall be generated.

---

# Related Documents

- STATE_MACHINE.md
- DATABASE_SCHEMA.md
- ERROR_CODES.md
- ATTENDANCE_POLICY.md