# Error Codes

**Project:** ITH Attendance Management System

**Document Version:** 2.0

**Status:** Approved

---

# Purpose

This document defines the standard error codes used throughout the ITH Attendance Management System.

The purpose of these codes is to provide a consistent, predictable, and implementation-independent method of communicating errors across the system.

These codes are referenced by the API specification, backend implementation, frontend user interface, and quality assurance documentation.

---

# Error Code Principles

Every error shall:

- Represent one business or system failure.
- Have a unique identifier.
- Be reusable across the application.
- Be understandable by developers.
- Support troubleshooting and auditing.

---

# Error Categories

| Prefix | Category |
|----------|----------|
| AUTH | Authentication |
| AUTHZ | Authorization |
| ATT | Attendance |
| VEN | Venue Validation |
| PRE | Presence Monitoring |
| LEAVE | Leave Management |
| ADMIN | Administrative Operations |
| VALID | Input Validation |
| STATE | State Machine |
| SYS | Internal System |

---

# Authentication Errors

| Code | Description |
|------|-------------|
| AUTH-001 | User is not authenticated |
| AUTH-002 | Session has expired |
| AUTH-003 | Invalid authentication token |
| AUTH-004 | Authentication failed |

---

# Authorization Errors

| Code | Description |
|------|-------------|
| AUTHZ-001 | Permission denied |
| AUTHZ-002 | Administrative privileges required |
| AUTHZ-003 | Volunteer action not permitted |

---

# Attendance Errors

| Code | Description |
|------|-------------|
| ATT-001 | Attendance session not found |
| ATT-002 | Active attendance already exists |
| ATT-003 | Attendance session already completed |
| ATT-004 | Event is not accepting attendance |
| ATT-005 | Volunteer is restricted from participation |

---

# Venue Validation Errors

| Code | Description |
|------|-------------|
| VEN-001 | Volunteer is outside the approved venue |
| VEN-002 | Venue boundary unavailable |
| VEN-003 | Invalid venue boundary |
| VEN-004 | Location information unavailable |

---

# Presence Monitoring Errors

| Code | Description |
|------|-------------|
| PRE-001 | Presence monitoring has not started |
| PRE-002 | Presence monitoring already stopped |
| PRE-003 | Invalid presence event |
| PRE-004 | Location update rejected |

---

# Leave Management Errors

| Code | Description |
|------|-------------|
| LEAVE-001 | Leave request not found |
| LEAVE-002 | Active leave request already exists |
| LEAVE-003 | Leave request already reviewed |
| LEAVE-004 | Invalid leave type |
| LEAVE-005 | Invalid leave reason |
| LEAVE-006 | Leave request cannot be submitted in the current attendance state |

---

# Administrative Errors

| Code | Description |
|------|-------------|
| ADMIN-001 | Administrative review required |
| ADMIN-002 | Volunteer restriction already exists |
| ADMIN-003 | Restriction not found |
| ADMIN-004 | Administrative action not permitted |

---

# Validation Errors

| Code | Description |
|------|-------------|
| VALID-001 | Required field missing |
| VALID-002 | Invalid field value |
| VALID-003 | Invalid geographic coordinates |
| VALID-004 | Timestamp is invalid |
| VALID-005 | Unsupported enumeration value |

---

# State Machine Errors

| Code | Description |
|------|-------------|
| STATE-001 | Invalid state transition |
| STATE-002 | Operation not permitted in current state |
| STATE-003 | Attendance state conflict |
| STATE-004 | Leave request state conflict |

---

# System Errors

| Code | Description |
|------|-------------|
| SYS-001 | Unexpected system error |
| SYS-002 | Service temporarily unavailable |
| SYS-003 | Configuration error |
| SYS-004 | Data persistence failure |

---

# Error Response Principles

Every reported error should provide:

- Error code
- Human-readable message
- Context, where appropriate
- Timestamp
- Correlation or request identifier, where supported

The format of the response is defined in the API documentation.

---

# Error Handling Principles

The system shall:

- Stop processing when an unrecoverable error occurs.
- Preserve data integrity.
- Avoid partial completion of business operations.
- Record significant system failures for audit and diagnostics.

---

# Related Documents

- VALIDATION_RULES.md
- STATE_MACHINE.md
- API/API_OVERVIEW.md