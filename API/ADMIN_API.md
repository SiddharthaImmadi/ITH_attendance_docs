
# Administration API

**Project:** ITH Attendance Management System

**Document Version:** 2.0

**Status:** Approved

---

# Purpose

This document defines the administrative APIs used to manage the Attendance Management System.

These APIs are restricted to authorized administrators.

---

# Business Rule References

- Business Rules/RULEBOOK.md
- Business Rules/LEAVE_POLICY.md
- Business Rules/VOLUNTEERING_POLICY.md
- Business Rules/APPROVAL_WORKFLOW.md

---

# Resource

```
Administration
```

---

# Review Leave Request

## Purpose

Retrieve a leave request for administrative review.

---

### Endpoint

```
GET /admin/leave-requests/{requestId}
```

---

### Successful Response

Returns:

- Volunteer information
- Attendance information
- Leave details
- Supporting explanation
- Current request status

---

### Validation Rules

The system shall verify:

- User is authenticated.
- User has administrator privileges.
- Leave request exists.

---

### Possible Errors

- AUTH-001
- AUTHZ-002
- LEAVE-001

---

### State Transition

None.

---

# Approve Leave Request

## Purpose

Approve a pending leave request.

---

### Endpoint

```
POST /admin/leave-requests/{requestId}/approve
```

---

### Successful Response

The leave request is approved.

The attendance state changes according to the approved leave type.

---

### Validation Rules

The system shall verify:

- User has administrator privileges.
- Leave request exists.
- Request is pending approval.

---

### Possible Errors

- AUTH-001
- AUTHZ-002
- LEAVE-001
- LEAVE-003
- STATE-001

---

### State Transition

```
PENDING_APPROVAL

↓

PARTIAL_LEAVE

or

COMPLETE_LEAVE
```

---

# Reject Leave Request

## Purpose

Reject a pending leave request.

---

### Endpoint

```
POST /admin/leave-requests/{requestId}/reject
```

---

### Successful Response

The leave request is rejected.

Administrative actions may follow according to organizational policy.

---

### Validation Rules

The system shall verify:

- User has administrator privileges.
- Leave request exists.
- Request is pending approval.

---

### Possible Errors

- AUTH-001
- AUTHZ-002
- LEAVE-001
- LEAVE-003
- STATE-001

---

### State Transition

```
PENDING_APPROVAL

↓

RESTRICTED
```

---

# Manage Volunteer Restriction

## Purpose

Apply or remove a volunteer restriction.

---

### Endpoint

```
PUT /admin/restrictions/{volunteerId}
```

---

### Successful Response

The volunteer restriction is updated.

---

### Validation Rules

The system shall verify:

- User has administrator privileges.
- Volunteer exists.

---

### Possible Errors

- AUTH-001
- AUTHZ-002
- ADMIN-002
- ADMIN-003

---

### State Transition

None.

---

# Manage Leave Reasons

## Purpose

Create, update, or deactivate predefined leave reasons.

---

### Endpoint

```
/admin/leave-reasons
```

---

### Validation Rules

The system shall verify:

- User has administrator privileges.

---

### Possible Errors

- AUTH-001
- AUTHZ-002

---

### State Transition

None.

---

# Manage Venue Boundary

## Purpose

Create or update the GeoJSON boundary associated with an event venue.

---

### Endpoint

```
PUT /admin/venues/{venueId}/boundary
```

---

### Validation Rules

The system shall verify:

- User has administrator privileges.
- Venue exists.
- Boundary definition is valid.

---

### Possible Errors

- AUTH-001
- AUTHZ-002
- VEN-003
- VALID-002

---

### State Transition

None.

---

# Related Documents

- Business Rules/APPROVAL_WORKFLOW.md
- System Design/STATE_MACHINE.md
- System Design/VALIDATION_RULES.md
- System Design/SECURITY_MODEL.md
- System Design/ERROR_CODES.md