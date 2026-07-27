# API_contract.md — Phase 1 Complete API Contract

> Single source of truth for what the frontend and backend agree on. Every endpoint, every
> request/response shape, every error code — all in one place. If FastAPI's auto-generated OpenAPI
> docs (`/docs`) ever disagree with this file, treat that as a bug — fix whichever one is wrong and
> note it in `changelog.md`.

**Base URL (local dev):** `http://localhost:8000`

**Authentication:** All authenticated endpoints require:
```
Authorization: Bearer <jwt>
```

---

## Table of Contents

1. Standard Response Format
2. Standard Error Format
3. Authentication
4. Events
5. Attendance
6. Presence Monitoring
7. Emergency Tickets
8. Notifications
9. Volunteer Blocking
10. Reports
11. Activity History
12. Audit Logs
13. Error Code Reference
14. Changelog
---

## Standard Response Format

All successful responses use this envelope:

```json
{
  "success": true,
  "message": "Operation completed successfully.",
  "data": {}
}
```

All error responses use:

```json
{
  "success": false,
  "message": "Operation failed.",
  "errors": [
    {
      "code": "ERROR_CODE",
      "details": "Additional information."
    }
  ]
}
```

## Standard Error Format

All error responses use this envelope:

```json
{
  "error": {
    "code": "ERROR_CODE_CONSTANT",
    "message": "Human readable explanation of what went wrong",
    "details": {}
  }
}
```

| HTTP Status | Used for |
|---|---|
| 400 | Bad input shape (validation error) |
| 401 | Missing/invalid/expired token |
| 403 | Authenticated but wrong role for this action |
| 404 | Resource not found or not visible to this user |
| 409 | Conflict (e.g., duplicate check-in, session already closed) |
| 422 | Business-rule rejection (e.g., outside event boundary, session not open, time window closed) |

---

## Authentication

### POST /auth/login

**Purpose:** Authenticate an admin or member user. Role is determined by the account on the
server, not by the request — the client never specifies which role it expects.

**Auth required:** No

**Request:**
```json
{
  "email": "admin@innotech-hub.com",
  "password": "AdminPass123!"
}
```

**Field notes:**
- `email` (string, required) — must match a registered account
- `password` (string, required) — sent over HTTPS only, never logged or exposed

**Response 200 — Login successful:**
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "bearer",
  "expires_in_minutes": 60,
  "user": {
    "id": "5f8d21e0-1234-4a5b-9c6d-abcdef123456",
    "full_name": "B. Sankar",
    "email": "admin@innotech-hub.com",
    "role": "admin"
  }
}
```

**Response 401 — Invalid credentials:**
```json
{
  "error": {
    "code": "INVALID_CREDENTIALS",
    "message": "Incorrect email or password."
  }
}
```

**Response 422 — Validation error (missing field):**
```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "email is required."
  }
}
```

---

### GET /me

**Purpose:** Return the profile of the currently authenticated user. Used right after login and
on app reload to restore session state.

**Auth required:** Yes (any role — admin or member)

**Response 200 — User profile (admin example):**
```json
{
  "id": "5f8d21e0-1234-4a5b-9c6d-abcdef123456",
  "full_name": "B. Sankar",
  "email": "admin@innotech-hub.com",
  "role": "admin"
}
```

**Response 200 — User profile (member example):**
```json
{
  "id": "8a1b2c3d-4444-4a5b-9c6d-abcdef654321",
  "full_name": "R. Meera",
  "email": "member1@innotech-hub.com",
  "role": "member"
}
```

**Response 401 — Missing or expired token:**
```json
{
  "error": {
    "code": "UNAUTHORIZED",
    "message": "Missing or expired token."
  }
}
```

---

## Events

### POST /sessions

**Purpose:** Business-rule rejection (e.g., outside event boundary, session not open, time window closed)
Session starts in `scheduled` status.

**Auth required:** Yes (admin only)

**Request:**
```json
{
  "title": "Workshop Attendance",
  "purpose": "Weekly InnoTech-Hub workshop",
  "date": "2026-08-01",
  "start_time": "2026-08-01T09:00:00Z",
  "end_time": "2026-08-01T12:00:00Z",
  "grace_period_minutes": 10,

  "boundary_type": "GEOJSON",

  "boundary": {
    "type": "Polygon",
    "coordinates": [
      [
        [77.5944, 12.9715],
        [77.5951, 12.9715],
        [77.5951, 12.9722],
        [77.5944, 12.9722],
        [77.5944, 12.9715]
      ]
    ]
  },

  "gps_update_interval": 15,
  "outside_grace_period": 10,
  "auto_timeout_minutes": 30
}
```

**Field notes:**
- `title` (string, required)
- `purpose` (string, optional)
- `date` (date `YYYY-MM-DD`, required)
- `start_time` (ISO 8601 datetime, required) — must be before `end_time`
- `end_time` (ISO 8601 datetime, required) — must be after `start_time`
- `grace_period_minutes` (integer, required) — minutes after `start_time` before a check-in is marked `late` instead of `present`
- `boundary_type (enum, required)

Defines how the attendance boundary is created.

Supported values:

• GEOJSON
• CAPTURED_POINTS

---

boundary (object, required)

Boundary definition.

If boundary_type = GEOJSON,
this must contain a valid GeoJSON Polygon.

If boundary_type = CAPTURED_POINTS,
this contains the captured GPS boundary points collected by the client.

---

gps_update_interval (integer, required)

Number of seconds between GPS updates while attendance monitoring is active.

Must be greater than 0.

---

outside_grace_period (integer, required)

Number of seconds a volunteer may remain outside the event boundary before being marked as LEFT.

Must be zero or greater.

---

auto_timeout_minutes (integer, required)

Number of minutes after the event ends before monitoring automatically terminates.

Must be greater than 0.

**Response 201 — Session created:**
```json
{
  "id": "b2c3d4e5-0001-4a5b-9c6d-abcdef111111",
  "title": "Workshop Attendance",
  "purpose": "Weekly InnoTech-Hub workshop",
  "date": "2026-08-01",
  "start_time": "2026-08-01T09:00:00Z",
  "end_time": "2026-08-01T12:00:00Z",
  "grace_period_minutes": 10,
      "boundary_type": "GEOJSON",

    "boundary": {
      "type": "Polygon",
      "coordinates": [
        [
          [77.5944, 12.9715],
          [77.5951, 12.9715],
          [77.5951, 12.9722],
          [77.5944, 12.9722],
          [77.5944, 12.9715]
        ]
      ]
    },

    "gps_update_interval": 15,
    "outside_grace_period": 10,
    "auto_timeout_minutes": 30,
  "status": "scheduled",
  "created_by": "5f8d21e0-1234-4a5b-9c6d-abcdef123456",
  "created_at": "2026-07-20T10:00:00Z"
}
```

**Response 400 — Invalid time sequence:**
```json
{
  "error": {
    "code": "INVALID_TIME_RANGE",
    "message": "end_time must be after start_time."
  }
}
```

**Response 403 — Non-admin attempted to create:**
```json
{
  "error": {
    "code": "FORBIDDEN",
    "message": "Only admins can create sessions."
  }
}
```

**Response 422 — Missing required field:**
```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "A valid event boundary must be provided."
  }
}
```
}
  "boundary_type": "GEOJSON",
  "boundary": {},
  "gps_update_interval": 15,
  "outside_grace_period": 10,
  "auto_timeout_minutes": 30
}

### Validation Rules

- boundary_type is required.
- GEOJSON requires a valid GeoJSON Polygon.
- CAPTURED_POINTS requires at least three GPS coordinates.
- Boundary cannot be modified after the session becomes ACTIVE.
- gps_update_interval must be greater than 0.
- outside_grace_period must be greater than or equal to 0.
- auto_timeout_minutes must be greater than 0.

---

### GET /sessions

**Purpose:** List sessions. Admin sees every session they created (any status). Member sees only
sessions currently in `open` status that they are eligible for.

**Auth required:** Yes (admin or member)

**Response 200 — Admin list (all statuses, all their sessions):**
```json
[
  {
    "id": "b2c3d4e5-0001-4a5b-9c6d-abcdef111111",
    "title": "Workshop Attendance",
    "status": "open",
    "date": "2026-08-01",
    "start_time": "2026-08-01T09:00:00Z",
    "end_time": "2026-08-01T12:00:00Z"
  },
  {
    "id": "b2c3d4e5-0002-4a5b-9c6d-abcdef222222",
    "title": "Volunteer Orientation",
    "status": "scheduled",
    "date": "2026-08-05",
    "start_time": "2026-08-05T09:00:00Z",
    "end_time": "2026-08-05T11:00:00Z"
  }
]
```

**Response 200 — Member list (open sessions only):**
```json
[
  {
    "id": "b2c3d4e5-0001-4a5b-9c6d-abcdef111111",
    "title": "Workshop Attendance",
    "status": "open",
    "start_time": "2026-08-01T09:00:00Z",
    "end_time": "2026-08-01T12:00:00Z"
  }
]
```

**Response 401 — Missing or expired token:**
```json
{
  "error": {
    "code": "UNAUTHORIZED",
    "message": "Missing or expired token."
  }
}
```

---

### GET /sessions/{id}

**Purpose:** Return full session configuration plus every attendance record (check-in) submitted
so far. Used for the admin's live monitoring / session review screen.

**Auth required:** Yes (admin only — must own the session)

**Path parameters:**
- `id` (uuid, required)

**Response 200 — Session detail with check-ins:**
```json
{
  "id": "b2c3d4e5-0001-4a5b-9c6d-abcdef111111",
  "title": "Workshop Attendance",
  "purpose": "Weekly InnoTech-Hub workshop",
  "status": "open",
  "date": "2026-08-01",
  "start_time": "2026-08-01T09:00:00Z",
  "end_time": "2026-08-01T12:00:00Z",
  "grace_period_minutes": 10,
    "boundary_type": "GEOJSON",
      "boundary": {
        "type": "Polygon",
        "coordinates": [
          [
            [77.5944, 12.9715],
            [77.5951, 12.9715],
            [77.5951, 12.9722],
            [77.5944, 12.9722],
            [77.5944, 12.9715]
          ]
        ]
      },
    "gps_update_interval": 15,
    "outside_grace_period": 10,
    "auto_timeout_minutes": 30,
  "attendance_records": [
    {
      "id": "c3d4e5f6-0001-4a5b-9c6d-abcdef333333",
      "member_id": "8a1b2c3d-4444-4a5b-9c6d-abcdef654321",
      "member_name": "R. Meera",
      "check_in_time": "2026-08-01T08:55:00Z",
      "distance_meters": 42,
      "gps_accuracy_meters": 12,
      "final_status": "present"
    }
  ]
}
```

**Response 404 — Session not found or not owned by this admin:**
```json
{
  "error": {
    "code": "NOT_FOUND",
    "message": "Session not found or not owned by this admin."
  }
}
```

---

### PATCH /sessions/{id}/close

**Purpose:** Manually close an open session, blocking any further check-ins. Corresponds to
Working Book section 4.4, "Close and Finalize Session".

**Auth required:** Yes (admin only — must own the session)

**Path parameters:**
- `id` (uuid, required)

**Response 200 — Session closed:**
```json
{
  "id": "b2c3d4e5-0001-4a5b-9c6d-abcdef111111",
  "status": "closed"
}
```

**Response 409 — Session was already closed:**
```json
{
  "error": {
    "code": "ALREADY_CLOSED",
    "message": "Session is already closed."
  }
}
```

**Response 404 — Session not found or not owned by this admin:**
```json
{
  "error": {
    "code": "NOT_FOUND",
    "message": "Session not found or not owned by this admin."
  }
}
```

---

## Attendance / Check-in

### POST /attendance/check-in

**Purpose:** Member submits their live GPS location AND a live-captured photo to check in to an
open session.

**Auth required:** Yes (member only)

**Content type:** `multipart/form-data`

**Request fields:**

| Field | Type | Required | Notes |
|---|---|---|---|
| `session_id` | uuid | yes | Which session to check in to |
| `lat` | double | yes | From `navigator.geolocation`, never manually entered |
| `lng` | double | yes | Latitude and longitude as decimal degrees |
| `gps_accuracy_meters` | double | yes | Accuracy reported by the device's location API |
| `photo` | file (binary) | yes | JPEG or PNG, max 5MB, **must be from live camera capture** (not gallery upload) |

**Server-side validation logic:**

**Boundary Validation**

The submitted GPS location is validated against the configured event boundary.

Validation depends on the session's boundary_type.

• GEOJSON
  Validate the submitted coordinates against the stored GeoJSON Polygon.

• CAPTURED_POINTS
  The backend first generates and stores a GeoJSON Polygon from the captured boundary points.
  Attendance validation is then performed against that generated polygon.

The client does not need to know which boundary type is used.
It only submits the current GPS location.

**Decision table:**
| Condition                                                         | Result                             |
| ----------------------------------------------------------------- | ---------------------------------- |
| Location is inside the configured boundary AND GPS accuracy ≤ 30m | ✅ Eligible for `present` or `late` |
| Location is outside the configured boundary                       | ❌ Reject with `OUTSIDE_BOUNDARY`   |
| GPS accuracy > 30m or location cannot be reliably verified        | ⚠️ `pending_verification`          |


**Time window check:**
- Must be during the session's open window
- If `check_in_time` is within `grace_period_minutes` after `start_time`, mark as `late`
- If `check_in_time` is after the `grace_period_minutes` window, reject with `SESSION_NOT_OPEN`

**Duplicate check:**
- Cannot have two check-ins from the same member in the same session
- Database unique constraint on `(session_id, member_id)`

**Photo evidence rules:**
- Must be a valid JPEG or PNG
- Max 5MB
- Cannot be a duplicate of a previously submitted photo (hash check to prevent token reuse)
- Blurred or dark photos are NOT auto-rejected — flagged as `pending_verification` for admin review

**Response 201 — Accepted (inside event boundary, on time):**
```json
{
  "id": "c3d4e5f6-0002-4a5b-9c6d-abcdef444444",
  "session_id": "b2c3d4e5-0001-4a5b-9c6d-abcdef111111",
  "check_in_time": "2026-08-01T08:55:00Z",
  "distance_meters": 42,
  "gps_accuracy_meters": 12,
  "final_status": "present"
}
```

**Response 201 — Accepted but pending review (poor GPS accuracy):**
```json
{
  "id": "c3d4e5f6-0003-4a5b-9c6d-abcdef555555",
  "session_id": "b2c3d4e5-0001-4a5b-9c6d-abcdef111111",
  "check_in_time": "2026-08-01T09:03:00Z",
  "distance_meters": 55,
  "gps_accuracy_meters": 95,
  "final_status": "pending_verification",
  "flag_reason": "GPS_ACCURACY_TOO_LOW"
}
```

**Response 201 — Late (inside event boundary, but after grace period):**
```json
{
  "id": "c3d4e5f6-0004-4a5b-9c6d-abcdef666666",
  "session_id": "b2c3d4e5-0001-4a5b-9c6d-abcdef111111",
  "check_in_time": "2026-08-01T09:15:00Z",
  "distance_meters": 38,
  "gps_accuracy_meters": 10,
  "final_status": "late"
}
```

**Response 422 — Outside event boundary:**
```json
{
  "error": {
    "code": "OUTSIDE_BOUNDARY",
    "message": "Submitted location is outside the configured event boundary."
  }
}
```

**Response 422 — Session not open:**
```json
{
  "error": {
    "code": "SESSION_NOT_OPEN",
    "message": "This session is not currently open for check-in."
  }
}
```

**Response 409 — Duplicate check-in:**
```json
{
  "error": {
    "code": "DUPLICATE_CHECK_IN",
    "message": "You have already checked in to this session."
  }
}
```

**Response 400 — Invalid photo type:**
```json
{
  "error": {
    "code": "INVALID_PHOTO_TYPE",
    "message": "Photo must be a JPEG or PNG image."
  }
}
```

**Response 400 — Photo file too large:**
```json
{
  "error": {
    "code": "PHOTO_TOO_LARGE",
    "message": "Photo exceeds the 5MB size limit."
  }
}
```

**Response 409 — Duplicate photo token:**
```json
{
  "error": {
    "code": "DUPLICATE_PHOTO_TOKEN",
    "message": "This photo has already been submitted for another check-in."
  }
}
```

---

### GET /attendance/history

**Purpose:** Return the logged-in member's own past check-ins across all sessions.

**Auth required:** Yes (member only)

**Response 200 — Attendance history:**
```json
[
  {
    "session_id": "b2c3d4e5-0001-4a5b-9c6d-abcdef111111",
    "session_title": "Workshop Attendance",
    "check_in_time": "2026-08-01T08:55:00Z",
    "final_status": "present"
  },
  {
    "session_id": "b2c3d4e5-0002-4a5b-9c6d-abcdef222222",
    "session_title": "Volunteer Orientation",
    "check_in_time": "2026-08-05T09:02:00Z",
    "final_status": "late"
  }
]
```

**Response 401 — Missing or expired token:**
```json
{
  "error": {
    "code": "UNAUTHORIZED",
    "message": "Missing or expired token."
  }
}
```

---

# Presence Monitoring

Presence Monitoring starts automatically after a successful check-in and ends when:

- The volunteer checks out.
- The event times out.
- The attendance session is completed.

The client submits periodic GPS updates while monitoring is active.

---

## POST /presence/location

**Purpose:** Submit the volunteer's current GPS location.

**Auth required:** Yes (member only)

### Request

```json
{
  "attendance_id": "uuid",
  "latitude": 17.3850,
  "longitude": 78.4867,
  "accuracy": 4.2,
  "provider": "gps",
  "captured_at": "2026-08-01T10:15:30Z"
}
```

### Field Notes

| Field | Description |
|--------|-------------|
| attendance_id | Attendance record identifier |
| latitude | Current latitude |
| longitude | Current longitude |
| accuracy | GPS accuracy in meters |
| provider | gps / network |
| captured_at | Device timestamp |

---

### Response 200

```json
{
  "success": true,
  "message": "Location received.",
  "data": {
    "status": "ACTIVE",
    "inside_boundary": true
  }
}
```

---

### Possible Status Values

- ACTIVE
- OUTSIDE
- RETURNED
- LEFT
- DISCONNECTED
- COMPLETED

---

### Business Rules

- GPS updates occur every 15 seconds.
- Remaining outside for 10 seconds changes status to OUTSIDE.
- Returning inside changes status to RETURNED.
- Approved emergency exits may transition to LEFT.
- GPS loss marks the session as DISCONNECTED.
- Monitoring automatically stops after the configured timeout once the event ends.

---

## GET /presence/{attendanceId}

Returns the current monitoring session.

---

## GET /presence

Optional filters:

- event_id
- volunteer_id
- status

--
---

# Emergency Tickets

Emergency Tickets allow volunteers to request permission to temporarily or permanently leave an active event.

Only one active emergency ticket may exist for an attendance record.

---

## POST /tickets

**Purpose:** Create an emergency leave request.

**Auth required:** Yes (member only)

### Request

```json
{
  "attendance_id": "uuid",
  "reason": "Medical Emergency",
  "description": "Need to visit the hospital."
}
```

### Validation Rules

- Attendance must be ACTIVE.
- One active ticket per attendance.
- Description maximum length: 100 characters.
- Attendance must belong to the authenticated member.

---

### Response 201

```json
{
  "success": true,
  "message": "Emergency ticket created.",
  "data": {
    "ticket_id": "uuid",
    "status": "PENDING"
  }
}
```

---

## GET /tickets

Returns emergency tickets.

### Optional Filters

- event_id
- volunteer_id
- status

---

## GET /tickets/{ticketId}

Returns complete ticket details.

---

## POST /tickets/{ticketId}/approve

**Auth required:** Admin only

Approves the emergency request.

Response:

```json
{
  "success": true,
  "message": "Emergency ticket approved."
}
```

---

## POST /tickets/{ticketId}/reject

**Auth required:** Admin only

Rejects the emergency request.

Response:

```json
{
  "success": true,
  "message": "Emergency ticket rejected."
}
```

---

## POST /tickets/{ticketId}/cancel

**Auth required:** Member only

Cancels a pending emergency request.

---

### Ticket Status

- PENDING
- APPROVED
- REJECTED
- CANCELLED
- USED

---

### Business Rules

- Only one active emergency ticket per attendance.
- Only PENDING tickets may be cancelled.
- APPROVED tickets become USED once the volunteer exits the venue.
- REJECTED tickets cannot be reused.
- All ticket actions are recorded in the activity history.

--
---

# Notifications

Notifications provide in-app alerts generated automatically by the system.

Users cannot create notifications manually.

---

## GET /notifications

**Purpose:** Return notifications for the authenticated user.

**Auth required:** Yes

### Optional Filters

- status
- type

---

### Response 200

```json
{
  "success": true,
  "message": "Notifications retrieved successfully.",
  "data": [
    {
      "id": "uuid",
      "type": "EVENT_REMINDER",
      "title": "Upcoming Event",
      "message": "Workshop begins in 30 minutes.",
      "status": "UNREAD",
      "created_at": "2026-08-01T08:30:00Z"
    }
  ]
}
```

---

## GET /notifications/{notificationId}

Returns a single notification.

---

## PATCH /notifications/{notificationId}/read

Marks a notification as READ.

### Response

```json
{
  "success": true,
  "message": "Notification marked as read."
}
```

---

## PATCH /notifications/read-all

Marks every unread notification for the authenticated user as READ.

### Response

```json
{
  "success": true,
  "message": "All notifications marked as read."
}
```

---

## Notification Types

- EVENT_REMINDER
- CHECK_IN_SUCCESS
- CHECK_OUT_SUCCESS
- EMERGENCY_APPROVED
- EMERGENCY_REJECTED
- VOLUNTEER_LEFT
- GPS_DISCONNECTED
- BLOCKED
- GENERAL

---

## Notification Status

- UNREAD
- READ
- EXPIRED

---

## Business Rules

- Notifications are generated by other modules.
- Users cannot edit notification content.
- Notifications automatically expire according to the application's retention policy.
- Expired notifications are read-only and are not returned by default.

--
---

# Volunteer Blocking

Volunteer Blocking temporarily prevents a volunteer from participating in future events after violating attendance policies.

Only administrators can remove an active block.

---



## GET /blocks/me

**Purpose:** Return the authenticated volunteer's current block status.

**Auth required:** Yes (member)

### Response 200

```json
{
  "success": true,
  "message": "Block status retrieved successfully.",
  "data": {
    "status": "ACTIVE",
    "blocked_events_remaining": 2,
    "reason": "Unauthorized venue exit",
    "blocked_since": "2026-08-15T10:20:00Z"
  }
}
```

---

## GET /blocks

**Purpose:** List blocked volunteers.

**Auth required:** Yes (admin)

### Optional Filters

- status
- volunteer_id

---

## GET /blocks/{blockId}

Returns complete block details.

---

## POST /blocks/{blockId}/remove

**Purpose:** Remove an active block.

**Auth required:** Yes (admin)

### Request

```json
{
  "reason": "Administrative review completed."
}
```

### Response

```json
{
  "success": true,
  "message": "Volunteer block removed successfully."
}
```

---

## Block Status

- ACTIVE
- COMPLETED
- REMOVED

---

## Business Rules

- A volunteer can have only one ACTIVE block at a time.
- Active blocks prevent event registration and check-in.
- New blocks are automatically created by attendance and presence policy violations.
- By default, a block lasts for three eligible club-organized events.
- After each eligible event, the remaining block count decreases automatically.
- When the remaining count reaches zero, the block status changes to COMPLETED.
- Administrators may remove a block at any time, changing its status to REMOVED.
- All block creation, completion, and removal actions are recorded in the audit history.

--
---

### Module Integration

| Module | Integration |
|----------|-------------|
| Attendance | Prevents check-in while an ACTIVE block exists |
| Presence Monitoring | May create a block after unauthorized venue exit |
| Emergency Tickets | Approved emergency exits do not create blocks |
| Notifications | Sends BLOCKED notification to the volunteer |
| Reports | Includes block statistics |
| Audit Logs | Records all administrative block actions |


# Reports & Analytics

## Overview

The Reports & Analytics module provides aggregated attendance, presence, and volunteer participation data for administrative review and operational decision-making.

Reports are read-only and do not modify application data.

All report endpoints require administrator authentication.

---

# Business Rules

- Reports are generated using the latest committed data.
- Report generation must not modify attendance records.
- Reports may be filtered using supported query parameters.
- Reports are returned in descending chronological order unless specified otherwise.
- Exported reports must contain the same data as the corresponding API response.
- Volunteers cannot access administrative reports.

---

# Supported Reports

| Report | Description |
|---------|-------------|
| Attendance Summary | Overall attendance statistics for an event |
| Presence Summary | Presence monitoring statistics |
| Volunteer Participation | Individual volunteer participation summary |
| Event Summary | Overall event statistics |

---

# GET /reports/attendance

## Purpose

Retrieve attendance statistics for one or more events.

---

## Authentication

Required

Administrator

---

## Query Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| event_id | UUID | No | Filter by event |
| from | DateTime | No | Start date |
| to | DateTime | No | End date |

---

## Success Response

HTTP 200 OK

```json
{
  "success": true,
  "message": "Attendance report generated successfully.",
  "data": {
    "total_registered": 120,
    "checked_in": 110,
    "checked_out": 104,
    "absent": 10,
    "pending_verification": 2
  }
}
```

---

# GET /reports/presence

## Purpose

Retrieve volunteer presence statistics.

---

## Authentication

Required

Administrator

---

## Query Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| event_id | UUID | Yes | Event identifier |

---

## Success Response

```json
{
  "success": true,
  "message": "Presence report generated successfully.",
  "data": {
    "volunteers_monitored": 110,
    "boundary_exits": 18,
    "successful_returns": 14,
    "emergency_requests": 3,
    "average_presence_percentage": 96.4
  }
}
```

---

# GET /reports/volunteers

## Purpose

Retrieve volunteer participation statistics.

---

## Authentication

Required

Administrator

---

## Query Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| event_id | UUID | No | Event identifier |
| user_id | UUID | No | Volunteer identifier |

---

## Success Response

```json
{
  "success": true,
  "message": "Volunteer report generated successfully.",
  "data": [
    {
      "user_id": "uuid",
      "name": "John Doe",
      "events_attended": 8,
      "check_in_count": 8,
      "check_out_count": 8,
      "boundary_exits": 2,
      "emergency_requests": 1,
      "completed_activities": 12,
      "block_status": "NONE"
    }
  ]
}
```

---

# GET /reports/events

## Purpose

Retrieve summary statistics for events.

---

## Authentication

Required

Administrator

---

## Success Response

```json
{
  "success": true,
  "message": "Event report generated successfully.",
  "data": [
    {
      "event_id": "uuid",
      "event_name": "Community Cleanup",
      "registered_volunteers": 120,
      "checked_in": 110,
      "checked_out": 104,
      "presence_rate": 96.4,
      "activities_completed": 58
    }
  ]
}
```

---

# Export Reports

## GET /reports/export

### Purpose

Export report data in a supported format.

---

## Authentication

Required

Administrator

---

## Query Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| report | Enum | Yes | Report type |
| format | Enum | Yes | Export format |

---

## Supported Report Types

- ATTENDANCE
- PRESENCE
- VOLUNTEERS
- EVENTS

---

## Supported Export Formats

- CSV
- XLSX

---

## Success Response

HTTP 200 OK

Returns the generated report file.

---

# Validation Rules

- Only administrators may access reports.
- Invalid event identifiers return EVENT_NOT_FOUND.
- Unsupported export formats return VALIDATION_ERROR.
- Empty reports return a successful response with zero records.

---

# Error Responses

| Error Code | HTTP | Description |
|------------|------|-------------|
| UNAUTHORIZED | 401 | Authentication required |
| FORBIDDEN | 403 | Administrator access required |
| EVENT_NOT_FOUND | 404 | Event not found |
| VALIDATION_ERROR | 422 | Invalid request parameters |

---

# Related Modules

- Events
- Attendance
- Presence Monitoring
- Emergency Tickets
- Volunteer Blocking
- Activity History
- Audit Logs

--
---

## Business Rules

- Reports are generated dynamically.
- Reports are read-only.
- Exports support CSV and XLSX only.
- Volunteers may only access their own report.


## Reports

### GET /reports/attendance.xlsx

**Purpose:** Generate an Excel workbook for a session, built on demand from PostgreSQL query
results — not a hand-maintained file. Admin only.

**Auth required:** Yes (admin only)

**Query parameters:**
- `session_id` (uuid, required)

**Response 200 — Excel file:**
- **Content-Type:** `application/vnd.openxmlformats-officedocument.spreadsheetml.sheet`
- **Body:** Binary `.xlsx` file

**Sheets included (Phase 1):**

**Sheet 1: Attendance Summary**
| Column | Example |
|---|---|
| Member ID | 8a1b2c3d-4444-4a5b-9c6d-abcdef654321 |
| Name | R. Meera |
| Session | Workshop Attendance |
| Date | 2026-08-01 |
| Check-in Time | 2026-08-01T08:55:00Z |
| Distance (m) | 42 |
| Final Status | present |

**Sheet 2: Photo Evidence**
| Column | Example |
|---|---|
| Member ID | 8a1b2c3d-4444-4a5b-9c6d-abcdef654321 |
| Session ID | b2c3d4e5-0001-4a5b-9c6d-abcdef111111 |
| Photo Reference | media/2026/08/01/c3d4e5f6-0002.jpg |
| Capture Time | 2026-08-01T08:55:00Z |

**Response 404 — Session not found:**
```json
{
  "error": {
    "code": "NOT_FOUND",
    "message": "Session not found."
  }
}
```

**Response 403 — Non-admin attempted export:**
```json
{
  "error": {
    "code": "FORBIDDEN",
    "message": "Only admins can export reports."
  }
}
```

---

## Error Code Reference

Every error code used across the entire API:

| Code | HTTP Status | Used in | Meaning |
|---|---|---|---|
| `INVALID_CREDENTIALS` | 401 | `/auth/login` | Email or password is incorrect |
| `UNAUTHORIZED` | 401 | All authenticated endpoints | Missing or expired JWT token |
| `FORBIDDEN` | 403 | `/sessions` (create), `/reports/attendance.xlsx` | Authenticated but wrong role |
| `VALIDATION_ERROR` | 422 | Any endpoint with malformed input | Required field missing or format invalid |
| `INVALID_TIME_RANGE` | 400 | `/sessions` (create) | `end_time` is not after `start_time` |
| `NOT_FOUND` | 404 | `/sessions/{id}`, `/reports/attendance.xlsx` | Resource not found or not owned |
| `ALREADY_CLOSED` | 409 | `/sessions/{id}/close` | Session is already in `closed` status |
| `SESSION_NOT_OPEN` | 422 | `/attendance/check-in` | Session is not currently open for check-in |
| OUTSIDE_BOUNDARY | 422 | /attendance/check-in | Submitted location is outside the configured event boundary |
| `DUPLICATE_CHECK_IN` | 409 | `/attendance/check-in` | Member already checked in to this session |
| `INVALID_PHOTO_TYPE` | 400 | `/attendance/check-in` | Photo is not a valid JPEG or PNG |
| `PHOTO_TOO_LARGE` | 400 | `/attendance/check-in` | Photo exceeds 5MB size limit |
| `DUPLICATE_PHOTO_TOKEN` | 409 | `/attendance/check-in` | Photo hash matches a previously submitted photo |
| `GPS_ACCURACY_TOO_LOW` | (201, flagged) | `/attendance/check-in` | GPS accuracy > 30m, marked `pending_verification` |

## Additional Phase 2 Error Codes

| Error Code | HTTP Status | Description |
|------------|-------------|-------------|
| EVENT_NOT_ACTIVE | 422 | The event has not started or is already closed. |
| EVENT_ALREADY_STARTED | 409 | The event has already been started. |
| EVENT_ALREADY_ENDED | 409 | The event has already ended. |
| INVALID_BOUNDARY | 422 | Invalid GeoJSON or captured boundary points. |
| OUTSIDE_BOUNDARY | 422 | Volunteer is outside the permitted event boundary. |
| GPS_SIGNAL_LOST | 422 | GPS signal is unavailable or unreliable. |
| LOCATION_UPDATE_REQUIRED | 422 | Required location update was not received. |
| ATTENDANCE_NOT_ACTIVE | 422 | Attendance record is not currently active. |
| DUPLICATE_LOCATION_UPDATE | 409 | Duplicate location update received. |
| EMERGENCY_TICKET_EXISTS | 409 | An active emergency ticket already exists. |
| EMERGENCY_TICKET_NOT_FOUND | 404 | Emergency ticket does not exist. |
| EMERGENCY_ALREADY_PROCESSED | 409 | Emergency ticket has already been approved, rejected, cancelled, or used. |
| VOLUNTEER_BLOCKED | 403 | Volunteer is currently blocked from participating. |
| BLOCK_NOT_FOUND | 404 | Block record does not exist. |
| NOTIFICATION_NOT_FOUND | 404 | Notification does not exist. |
| ACTIVITY_NOT_FOUND | 404 | Activity record does not exist. |
| AUDIT_RECORD_NOT_FOUND | 404 | Audit record does not exist. |
| REPORT_NOT_AVAILABLE | 404 | Requested report could not be generated. |
| INVALID_EXPORT_FORMAT | 400 | Unsupported export format. |

---

# Status Enums

These enum values are shared across the backend, frontend, database, and business rules.

Changing any enum value is considered a breaking API contract change.

---

## User Roles

| Value | Description |
|--------|-------------|
| `admin` | Organization administrator with permission to manage users, events, attendance, reports, and system configuration. |
| `member` | Volunteer or participant authorized to attend assigned events. |

---

## Event Status

| Value | Description |
|--------|-------------|
| `scheduled` | Event has been created but is not yet open for attendance. |
| `open` | Event is active and accepting attendance submissions. |
| `closed` | Event has ended or has been manually closed by an administrator. |

---

## Boundary Types

Defines how the event boundary was created.

| Value | Description |
|--------|-------------|
| `GEOJSON` | Administrator uploads a GeoJSON Polygon that represents the event boundary. |
| `CAPTURED_POINTS` | Administrator captures GPS points by walking around the venue. The backend converts these points into a GeoJSON Polygon and stores both the captured points and generated polygon. |

---

## Attendance Final Status

Represents the final outcome of a volunteer's attendance.

| Value | Description |
|--------|-------------|
| `present` | Volunteer successfully checked in from within the configured event boundary during the allowed attendance window. |
| `late` | Volunteer successfully checked in from within the configured event boundary after the grace period. |
| `pending_verification` | Attendance requires administrator review due to insufficient GPS accuracy or another verification issue. |
| `rejected` | Attendance submission was rejected because validation requirements were not satisfied. |

---

## Presence States

Represents the volunteer's real-time participation state during an active event.

| Value | Description |
|--------|-------------|
| `CHECK_IN` | Volunteer successfully started the attendance session. |
| `PRESENCE_OK` | Latest location update confirms the volunteer remains inside the configured event boundary. |
| `LEFT` | Volunteer remained outside the configured event boundary longer than the configured outside grace period. |
| `RETURNED` | Volunteer re-entered the configured event boundary after previously leaving. |
| `CHECK_OUT` | Volunteer completed or ended the attendance session. |

---

## Volunteer Block Status

Represents participation restrictions applied by administrators.

| Value | Description |
|--------|-------------|
| `ACTIVE` | Volunteer is currently blocked from joining eligible events. |
| `LIFTED` | Block has been removed by an administrator. |
| `EXPIRED` | Temporary block expired automatically according to policy. |

---

## Emergency Ticket Status

| Value | Description |
|--------|-------------|
| `OPEN` | Emergency ticket has been created and is awaiting administrator action. |
| `APPROVED` | Administrator approved the emergency request. |
| `REJECTED` | Administrator rejected the emergency request. |
| `CANCELLED` | Emergency request was cancelled before processing. |
| `RESOLVED` | Emergency workflow has been completed successfully. |

---

## Notification Status

| Value | Description |
|--------|-------------|
| `UNREAD` | Notification has not yet been viewed. |
| `READ` | Notification has been viewed by the recipient. |

---

## Notification Types

| Value | Description |
|--------|-------------|
| `CHECK_IN` | Attendance successfully started. |
| `LEFT_BOUNDARY` | Volunteer exited the configured event boundary. |
| `RETURNED` | Volunteer re-entered the configured event boundary. |
| `CHECK_OUT` | Attendance session completed. |
| `EMERGENCY` | Emergency-related notification. |
| `BLOCKED` | Volunteer participation restriction notification. |
| `SYSTEM` | General system notification. |

---

## Activity Status

| Value | Description |
|--------|-------------|
| `PENDING` | Activity has been assigned but not started. |
| `IN_PROGRESS` | Volunteer is actively performing the assigned activity. |
| `COMPLETED` | Activity completed successfully. |
| `REJECTED` | Activity submission was rejected during review. |

---

## Audit Action Types

| Value | Description |
|--------|-------------|
| `CREATE` | Resource created. |
| `UPDATE` | Resource updated. |
| `DELETE` | Resource deleted. |
| `LOGIN` | User logged into the system. |
| `LOGOUT` | User logged out of the system. |
| `CHECK_IN` | Attendance check-in recorded. |
| `CHECK_OUT` | Attendance check-out recorded. |
| `APPROVE` | Administrator approved a request. |
| `REJECT` | Administrator rejected a request. |

# API Contract Change Process

This API Contract is the authoritative interface specification shared between frontend, backend, QA, and documentation.

Any modification to this document must follow the contract management process defined below.

Breaking changes must never be introduced without explicit review and approval.

---

# Contract Stability Rules

The following items are considered part of the public API contract.

Changing any of these items is a breaking change.

- Endpoint path
- HTTP method
- Request field names
- Response field names
- Enum values
- Data types
- Required / optional fields
- Authentication requirements
- HTTP status codes
- Error codes
- Pagination structure
- Sorting/filter parameters
- Boundary model
- Validation behavior
- Business workflow that affects API behavior

---

# Non-Breaking Changes

The following changes are generally considered safe.

- Documentation improvements
- Additional examples
- Clarified descriptions
- New optional request fields
- New optional response fields
- Additional endpoints
- Performance improvements
- Internal implementation changes

These changes still require updating the changelog.

---

# Breaking Changes

The following changes require explicit approval before implementation.

## Endpoint Changes

Changing

- URL
- HTTP method
- Authentication requirement

Example

```
POST /attendance/check-in

↓

POST /attendance/start
```

Breaking.

---

## Request Changes

Changing

```
latitude

↓

lat
```

Breaking.

---

Removing

```
gps_accuracy_meters
```

Breaking.

---

Changing a required field into another required field.

Breaking.

---

Changing data type

```
integer

↓

string
```

Breaking.

---

## Response Changes

Removing any response field.

Breaking.

Changing response structure.

Breaking.

Changing enum values.

Breaking.

Changing status codes.

Breaking.

---

## Enum Changes

Changing

```
present

↓

checked_in
```

Breaking.

Changing

```
LEFT

↓

LEFT_VENUE
```

Breaking.

Adding new enum values requires frontend review.

---

## Error Code Changes

Changing

```
OUTSIDE_BOUNDARY

↓

OUTSIDE_GEOFENCE
```

Breaking.

Removing an error code.

Breaking.

Changing HTTP status.

Breaking.

---

## Boundary Model

The finalized boundary architecture is part of the API contract.

Supported boundary types are:

- GEOJSON
- CAPTURED_POINTS

No additional boundary type may be introduced without updating:

- API Contract
- Database Schema
- Backend Documentation
- Frontend Documentation
- Business Rules
- QA Documentation

---

# Documentation Synchronization

Whenever this API Contract changes, the following documents must be reviewed for consistency.

- Backend API Implementation
- Backend Architecture
- Database Schema
- Business Rules
- Frontend API Documentation
- Frontend Architecture
- Testing Documentation
- Changelog

---

# API Versioning

The project currently uses a single API version.

```
/api/v1
```

Breaking changes should be avoided whenever possible.

If a future change cannot remain backward compatible, a new API version should be introduced instead of modifying existing endpoints.

Example

```
/api/v2/attendance/check-in
```

---

# Backward Compatibility

Whenever possible,

- add new endpoints
- add optional fields
- preserve existing behavior

instead of modifying existing interfaces.

Existing clients should continue functioning without requiring immediate updates.

---

# Change Review Process

Every contract modification should follow this sequence.

```
Requirement

        ↓

Architecture Review

        ↓

Business Rule Validation

        ↓

API Contract Update

        ↓

Database Review

        ↓

Backend Review

        ↓

Frontend Review

        ↓

QA Review

        ↓

Implementation
```

Implementation must never begin before the API Contract has been updated.

---

# Pull Request Requirements

Every pull request affecting this contract must include

- Purpose of the change
- Affected endpoints
- Backward compatibility assessment
- Breaking change assessment
- Required frontend changes
- Required backend changes
- Required database changes
- Required QA updates
- Changelog reference

---

# Changelog Requirements

Every API Contract modification must be recorded in the project changelog.

Each entry should include

- Date
- Version
- Author
- Affected module
- Summary of changes
- Breaking change (Yes/No)

Example

```
Version: 2.0.0

Module:
Presence Monitoring

Summary:
Introduced boundary-based monitoring using stored GeoJSON.

Breaking Change:
No
```

---

# Implementation Rule

Backend and frontend implementations must follow this contract exactly.

If implementation requirements differ from this document,

implementation must stop until

- the discrepancy is discussed,
- the contract is updated,
- documentation is synchronized,
- approval is granted.

The API Contract always takes precedence over implementation assumptions.

# Activity History

## Overview

Activity History provides a chronological record of significant actions performed by or affecting a volunteer during an event.

The module is intended for transparency, auditing, and self-service review. It allows volunteers to review their own participation history while enabling administrators to investigate attendance, presence, emergency requests, activities, and administrative actions.

Activity History is read-only.

No endpoint in this module creates, updates, or deletes records directly.

History records are generated automatically by the system whenever supported actions occur.

---

# Business Rules

- Every activity history record is immutable after creation.
- Records are created only by successful system events.
- Records must always reference the originating module.
- Records must be ordered using server timestamps.
- Server time is always authoritative.
- Records must not be deleted through the public API.
- Administrators may archive records according to organizational retention policies.
- Volunteers may only access their own activity history.
- Administrators may access activity history for authorized users.

---

# History Categories

Each record belongs to one of the following categories.

| Category | Description |
|----------|-------------|
| ATTENDANCE | Check-in and check-out events |
| PRESENCE | Presence monitoring events |
| ACTIVITY | Assigned or completed volunteer activities |
| EMERGENCY | Emergency ticket lifecycle |
| BLOCK | Volunteer participation restrictions |
| NOTIFICATION | Important notifications delivered to the volunteer |
| ADMIN | Administrative actions affecting the volunteer |
| SYSTEM | Automatically generated system events |

---

# Activity Types

Every history record contains an activity type.

Supported values include:

- CHECK_IN
- CHECK_OUT
- PRESENCE_OK
- LEFT
- RETURNED
- ACTIVITY_ASSIGNED
- ACTIVITY_UPDATED
- ACTIVITY_COMPLETED
- EMERGENCY_CREATED
- EMERGENCY_APPROVED
- EMERGENCY_REJECTED
- EMERGENCY_CANCELLED
- BLOCK_APPLIED
- BLOCK_REMOVED
- NOTIFICATION_SENT
- PROFILE_UPDATED
- SESSION_JOINED
- SESSION_COMPLETED

Additional activity types may be introduced in future releases without affecting existing values.

---

# History Record Structure

Every activity history record contains the following information.

| Field | Type | Required | Description |
|------|------|----------|-------------|
| id | UUID | Yes | History record identifier |
| user_id | UUID | Yes | Volunteer |
| event_id | UUID | No | Related event |
| attendance_id | UUID | No | Related attendance session |
| category | Enum | Yes | History category |
| activity_type | Enum | Yes | Activity performed |
| title | String | Yes | Short summary |
| description | String | Yes | Human-readable explanation |
| created_at | DateTime | Yes | Server timestamp |
| metadata | Object | No | Additional structured information |

---

# GET /activity/me

## Purpose

Return the authenticated volunteer's activity history.

---

## Authentication

Required

Member

---

## Request

No request body.

---

## Query Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| page | Integer | No | Page number |
| page_size | Integer | No | Number of records per page |
| category | Enum | No | Filter by history category |
| activity_type | Enum | No | Filter by activity type |
| event_id | UUID | No | Filter by event |
| from | DateTime | No | Start date |
| to | DateTime | No | End date |

---

## Success Response

HTTP 200 OK

```json
{
  "success": true,
  "message": "Activity history retrieved successfully.",
  "data": {
    "items": [
      {
        "id": "history_uuid",
        "category": "ATTENDANCE",
        "activity_type": "CHECK_IN",
        "title": "Attendance Started",
        "description": "Successfully checked in.",
        "created_at": "2026-08-01T09:02:15Z",
        "metadata": {
          "event_id": "event_uuid",
          "attendance_id": "attendance_uuid"
        }
      }
    ],
    "pagination": {
      "page": 1,
      "page_size": 20,
      "total_records": 52,
      "total_pages": 3
    }
  }
}
```

---

# GET /activity/{user_id}

## Purpose

Return activity history for a specific volunteer.

---

## Authentication

Administrator

---

## Path Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| user_id | UUID | Volunteer identifier |

---

## Query Parameters

Same as `/activity/me`.

---

## Success Response

Same response format as `/activity/me`.

---

# Filtering Rules

Filtering may be applied independently or combined.

Supported filters include:

- Category
- Activity Type
- Event
- Date Range

Filtering must not modify the chronological ordering.

---

# Sorting Rules

Default sorting

```
created_at DESC
```

Newest records appear first.

---

# Pagination

Default page size

```
20
```

Maximum page size

```
100
```

---

# Metadata Examples

Attendance

```json
{
    "attendance_id":"uuid",
    "event_id":"uuid"
}
```

Presence

```json
{
    "state":"LEFT",
    "gps_accuracy_meters":12
}
```

Emergency

```json
{
    "ticket_id":"uuid",
    "status":"APPROVED"
}
```

Volunteer Block

```json
{
    "block_id":"uuid",
    "reason":"Repeated unauthorized exits"
}
```

---

# Validation Rules

- Volunteers may only access their own history.
- Administrators may access authorized users.
- Unknown filters return validation errors.
- Invalid UUIDs return 404 or validation errors as appropriate.
- Empty history returns an empty list.

---

# Error Responses

| Error Code | HTTP | Description |
|------------|------|-------------|
| UNAUTHORIZED | 401 | Authentication required |
| FORBIDDEN | 403 | Access denied |
| USER_NOT_FOUND | 404 | Volunteer does not exist |
| VALIDATION_ERROR | 422 | Invalid filter values |

---

# Notes

- History records are append-only.
- History records cannot be modified through this API.
- Server timestamps are authoritative.
- The API returns records in descending chronological order.
- Activity History is intended for review and auditing, not operational updates.

---

# Related Modules

- Attendance
- Presence Monitoring
- Emergency Tickets
- Volunteer Blocking
- Notifications
- Reports
- Audit Logs

## GET /activity/me

**Purpose:** Return the authenticated volunteer's activity history.

**Auth required:** Yes (member)

### Response

```json
{
  "success": true,
  "message": "Activity history retrieved successfully.",
  "data": [
    {
      "id": "uuid",
      "activity_type": "CHECK_IN",
      "attendance_id": "uuid",
      "timestamp": "2026-08-01T09:00:00Z",
      "metadata": {}
    }
  ]
}
```

---

## GET /activity/attendance/{attendanceId}

Returns every activity associated with an attendance record.

---

## Activity Types

- CHECK_IN
- CHECK_OUT
- LEFT
- RETURNED
- PHOTO_CAPTURED
- EMERGENCY_REQUESTED
- EMERGENCY_APPROVED
- EMERGENCY_REJECTED

---

## Business Rules

- Activity records are created automatically.
- Activity history cannot be edited.
- Activity history cannot be deleted.
- Volunteers can only view their own history.
- Administrators may view any volunteer's history.


# Audit Logs

## Overview

The Audit Logs module records security-sensitive and administrative actions performed within the system.

Audit logs provide accountability, traceability, and historical records for actions that affect users, events, attendance, and system configuration.

Audit records are generated automatically by the system and are immutable.

Audit logs are read-only through the public API.

---

# Business Rules

- Every administrative action that modifies system state must generate an audit log.
- Audit logs cannot be modified or deleted through the API.
- Audit records are created using server timestamps.
- Audit logs are visible only to authorized administrators.
- Failed authorization attempts should be logged internally but are not exposed through this API.

---

# Logged Actions

The following actions generate audit records.

## Event Management

- Event Created
- Event Updated
- Event Cancelled
- Event Deleted

---

## Attendance

- Manual Check-in
- Manual Check-out
- Attendance Verification
- Attendance Correction

---

## Presence Monitoring

- Presence Monitoring Started
- Presence Monitoring Stopped
- Manual Presence Override

---

## Emergency Tickets

- Emergency Ticket Approved
- Emergency Ticket Rejected
- Emergency Ticket Closed

---

## Volunteer Management

- Volunteer Block Applied
- Volunteer Block Removed
- Volunteer Assignment Updated

---

## Administrative Actions

- User Created
- User Updated
- User Role Changed
- User Deactivated

---

# Audit Log Structure

| Field | Type | Required | Description |
|---------|------|----------|-------------|
| id | UUID | Yes | Audit log identifier |
| actor_id | UUID | Yes | Administrator performing the action |
| actor_role | Enum | Yes | Role of the acting user |
| action | String | Yes | Action performed |
| resource_type | String | Yes | Resource affected |
| resource_id | UUID | No | Identifier of the affected resource |
| description | String | Yes | Human-readable summary |
| created_at | DateTime | Yes | Server timestamp |
| metadata | Object | No | Additional contextual information |

---

# GET /audit-logs

## Purpose

Retrieve audit log records.

---

## Authentication

Required

Administrator

---

## Query Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| page | Integer | No | Page number |
| page_size | Integer | No | Records per page |
| actor_id | UUID | No | Filter by administrator |
| action | String | No | Filter by action |
| resource_type | String | No | Filter by affected resource |
| from | DateTime | No | Start date |
| to | DateTime | No | End date |

---

## Success Response

HTTP 200 OK

```json
{
  "success": true,
  "message": "Audit logs retrieved successfully.",
  "data": {
    "items": [
      {
        "id": "audit_uuid",
        "actor_id": "admin_uuid",
        "actor_role": "ADMIN",
        "action": "VOLUNTEER_BLOCK_APPLIED",
        "resource_type": "VOLUNTEER",
        "resource_id": "user_uuid",
        "description": "Volunteer participation restriction applied.",
        "created_at": "2026-08-01T11:20:45Z",
        "metadata": {
          "event_id": "event_uuid",
          "reason": "Repeated unauthorized boundary exits"
        }
      }
    ],
    "pagination": {
      "page": 1,
      "page_size": 20,
      "total_records": 185,
      "total_pages": 10
    }
  }
}
```

---

# Validation Rules

- Only administrators may access audit logs.
- Invalid filters return VALIDATION_ERROR.
- Unknown resources return an empty result set.
- Audit records are immutable.

---

# Error Responses

| Error Code | HTTP | Description |
|------------|------|-------------|
| UNAUTHORIZED | 401 | Authentication required |
| FORBIDDEN | 403 | Administrator access required |
| VALIDATION_ERROR | 422 | Invalid request parameters |

---

# Notes

- Audit logs are append-only.
- Audit records cannot be modified through the API.
- Server timestamps are authoritative.
- Audit logs are intended for compliance, investigation, and operational review.

---

# Related Modules

- Authentication
- Events
- Attendance
- Presence Monitoring
- Emergency Tickets
- Volunteer Blocking
- Activity History
- Reports & Analytics

## GET /audit

**Purpose:** Return administrative audit records.

**Auth required:** Yes (admin)

### Optional Filters

- admin_id
- action
- resource
- date_from
- date_to

---

## GET /audit/{auditId}

Returns a single audit entry.

---

## Audit Actions

- EVENT_CREATED
- EVENT_UPDATED
- EVENT_STARTED
- EVENT_ENDED
- EMERGENCY_APPROVED
- EMERGENCY_REJECTED
- BLOCK_REMOVED

---

## Response Example

```json
{
  "success": true,
  "message": "Audit record retrieved successfully.",
  "data": {
    "id": "uuid",
    "admin_id": "uuid",
    "action": "EVENT_CREATED",
    "resource": "EVENT",
    "resource_id": "uuid",
    "timestamp": "2026-08-01T08:15:00Z",
    "metadata": {}
  }
}
```

---

## Business Rules

- Audit records are generated automatically.
- Audit records cannot be modified.
- Audit records cannot be deleted.
- Only administrators may access audit logs.
--
```

# API Contract Change Process

## Overview

This API Contract is the authoritative specification for all frontend and backend implementations.

Any modification affecting API behavior must be reviewed, approved, and documented before implementation.

No implementation may intentionally deviate from this contract.

---

# Scope

This change process applies to:

- API endpoints
- HTTP methods
- Authentication requirements
- Request body schemas
- Response body schemas
- Query parameters
- Path parameters
- Error responses
- Status codes
- Enum values
- Business validation rules

---

# Change Classification

API changes are classified into two categories.

## Non-Breaking Changes

The following changes do not affect existing API consumers.

Examples:

- Adding a new endpoint
- Adding an optional request field
- Adding an optional response field
- Adding new documentation examples
- Clarifying descriptions
- Performance optimizations without changing API behavior

These changes must be documented in the API Changelog.

---

## Breaking Changes

Breaking changes modify existing API behavior and may require frontend or backend updates.

Examples include:

- Renaming endpoints
- Removing endpoints
- Changing HTTP methods
- Renaming request fields
- Removing request fields
- Renaming response fields
- Removing response fields
- Changing data types
- Changing enum values
- Modifying authentication requirements
- Changing validation behavior
- Changing HTTP status codes
- Removing supported functionality

Breaking changes require explicit approval before implementation.

---

# Change Approval Process

Every API modification must follow this process.

1. Identify the required change.
2. Review the impact on existing functionality.
3. Update the API Contract.
4. Review with frontend and backend developers.
5. Obtain approval.
6. Update dependent documentation.
7. Implement the approved change.
8. Record the change in the API Changelog.

Implementation must not begin before the API Contract has been updated and approved.

---

# Documentation Synchronization

Whenever the API Contract changes, the following documents must be reviewed for consistency.

- Backend Documentation
- Frontend Documentation
- Database Schema
- Business Rules
- Testing Documentation
- API Changelog

---

# Backward Compatibility

Whenever possible, existing endpoints should remain compatible with current clients.

Preferred approaches include:

- Adding new endpoints
- Adding optional fields
- Extending existing functionality without modifying existing behavior

Breaking changes should be avoided whenever practical.

---

# Versioning

API Contract versions must be recorded in the API Changelog.

Every approved modification must include:

- Version
- Release Date
- Modules Affected
- Summary of Changes
- Breaking Change Status

---

# Implementation Rule

Frontend and backend implementations must follow this API Contract.

If implementation requirements differ from this document, implementation must stop until:

- the discrepancy is identified,
- the API Contract is updated,
- approval is obtained, and
- the API Changelog is updated.

The approved API Contract is the single source of truth for all API behavior.
--
```


# API Changelog

## Overview

This changelog records all approved modifications to the API Contract.

Every change affecting endpoints, request or response schemas, authentication, business rules, validation behavior, error handling, or supported workflows must be documented before implementation.

Historical entries must remain immutable.

---

# Changelog Entry Format

Each changelog entry must include:

| Field | Description |
|--------|-------------|
| Version | API Contract version |
| Release Date | Date the contract revision was approved |
| Author | Individual or team responsible for the change |
| Modules Affected | List of modified modules |
| Summary | Description of the change |
| Breaking Change | Yes / No |

---

## Version 2.0.0

| Field | Value |
|--------|-------|
| Version | 2.0.0 |
| Release Date | YYYY-MM-DD |
| Author | Project Team |
| Breaking Change | No |

### Modules Affected

- Authentication
- Events
- Attendance
- Presence Monitoring
- Emergency Tickets
- Volunteer Blocking
- Notifications
- Activity History
- Reports & Analytics
- Audit Logs

### Summary

Initial Phase 2 API Contract.

Introduced:

- Boundary-based attendance validation using GeoJSON.
- Support for CAPTURED_POINTS boundary generation.
- Continuous presence monitoring.
- Presence state transitions.
- Emergency ticket workflow.
- Volunteer participation restriction workflow.
- Notification module.
- Activity history.
- Administrative reporting.
- Audit logging.
- Standardized response and error formats.

---

# Future Changes

Future API Contract revisions must be appended as new entries.

Existing entries must not be modified.

Example:

## Version X.Y.Z

| Field | Value |
|--------|-------|
| Version | X.Y.Z |
| Release Date | YYYY-MM-DD |
| Author | Project Team |
| Breaking Change | Yes / No |

### Modules Affected

- Module A
- Module B

### Summary

Describe the approved changes introduced in this version.

**Release Date**

2026-08-01

**Author**

Project Team

**Modules Affected**

- Events
- Attendance
- Presence Monitoring
- Emergency Tickets
- Notifications
- Volunteer Blocking
- Activity History
- Reports & Analytics
- Audit Logs

**Summary**

Initial Phase 2 API Contract.

Introduced:

- GeoJSON event boundaries
- Captured Points boundary generation
- Continuous presence monitoring
- Emergency ticket workflow
- Volunteer participation restrictions
- Activity history
- Notification module
- Reporting endpoints
- Audit logging

**Breaking Change**

No

---

# Future Versions

Subsequent releases should append new entries without modifying historical records.

Older changelog entries must remain immutable.

### Changed

- Session terminology updated to Event throughout the documentation.
- Existing Phase 1 APIs remain backward compatible.
- API documentation expanded to support Phase 2 functionality.

### Compatibility

- No Phase 1 endpoint was removed.
- Existing request and response formats remain supported.
- Phase 2 functionality is additive and does not introduce breaking changes.