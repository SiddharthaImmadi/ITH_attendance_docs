
# Report API

**Project:** ITH Attendance Management System

**Document Version:** 2.0

**Status:** Approved

---

# Purpose

This document defines the APIs used to retrieve attendance-related reports and export operational data.

Reporting APIs provide read-only access to attendance information for volunteers and administrators.

These APIs shall not modify operational data.

---

# Business Rule References

- Business Rules/ATTENDANCE_POLICY.md
- Business Rules/LEAVE_POLICY.md
- Business Rules/VOLUNTEERING_POLICY.md

---

# Resource

```
Reports
```

---

# Attendance Report

## Purpose

Retrieve attendance records for one or more volunteers.

---

### Endpoint

```
GET /reports/attendance
```

---

### Query Parameters

Typical filters include:

- Event
- Volunteer
- Date Range
- Attendance Status

---

### Successful Response

Returns attendance information including:

- Volunteer details
- Event information
- Check-in time
- Check-out time
- Attendance duration
- Attendance status

---

### Validation Rules

The system shall verify:

- User is authenticated.
- User is authorized to access attendance reports.

---

### Possible Errors

- AUTH-001
- AUTHZ-001

---

### State Transition

None.

---

# Presence Report

## Purpose

Retrieve presence monitoring information for attendance sessions.

---

### Endpoint

```
GET /reports/presence
```

---

### Query Parameters

Typical filters include:

- Event
- Volunteer
- Date Range

---

### Successful Response

Returns:

- Presence timeline
- Venue exit events
- Return events
- Outside duration

---

### Validation Rules

The system shall verify:

- User is authenticated.
- User is authorized.

---

### Possible Errors

- AUTH-001
- AUTHZ-001

---

### State Transition

None.

---

# Leave Report

## Purpose

Retrieve leave request history.

---

### Endpoint

```
GET /reports/leaves
```

---

### Query Parameters

Typical filters include:

- Event
- Volunteer
- Leave Type
- Leave Status
- Date Range

---

### Successful Response

Returns:

- Leave type
- Leave reason
- Explanation
- Approval status
- Administrator decision
- Decision timestamp

---

### Validation Rules

The system shall verify:

- User is authenticated.
- User is authorized.

---

### Possible Errors

- AUTH-001
- AUTHZ-001

---

### State Transition

None.

---

# Restriction Report

## Purpose

Retrieve volunteer restriction history.

---

### Endpoint

```
GET /reports/restrictions
```

---

### Query Parameters

Typical filters include:

- Volunteer
- Restriction Status
- Date Range

---

### Successful Response

Returns:

- Restriction reason
- Restriction status
- Applied date
- Removed date
- Administrative remarks

---

### Validation Rules

The system shall verify:

- User is authenticated.
- User is authorized.

---

### Possible Errors

- AUTH-001
- AUTHZ-001

---

### State Transition

None.

---

# Attendance Summary

## Purpose

Retrieve summarized attendance statistics.

---

### Endpoint

```
GET /reports/summary
```

---

### Successful Response

Typical information includes:

- Total volunteers
- Total attendance
- Active attendance
- Completed attendance
- Approved leave requests
- Rejected leave requests
- Active restrictions

---

### Validation Rules

The system shall verify:

- User is authenticated.
- User is authorized.

---

### Possible Errors

- AUTH-001
- AUTHZ-001

---

### State Transition

None.

---

# Export Report

## Purpose

Export report data in a supported format.

---

### Endpoint

```
GET /reports/export
```

---

### Query Parameters

Typical parameters include:

- Report Type
- Export Format
- Date Range

---

### Successful Response

Returns the requested report in the selected export format.

Supported export formats are defined by organizational requirements.

---

### Validation Rules

The system shall verify:

- User is authenticated.
- User is authorized.
- Requested report exists.
- Requested export format is supported.

---

### Possible Errors

- AUTH-001
- AUTHZ-001
- VALID-002

---

### State Transition

None.

---

# Reporting Principles

Reporting APIs:

- Shall not modify operational data.
- Shall return historical information only.
- Shall reflect committed attendance records.
- Shall preserve chronological accuracy.

---

# Related Documents

- Business Rules/ATTENDANCE_POLICY.md
- Business Rules/LEAVE_POLICY.md
- System Design/DATABASE_SCHEMA.md
- System Design/VALIDATION_RULES.md
- System Design/ERROR_CODES.md