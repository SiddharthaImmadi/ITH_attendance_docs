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

1. Standard Response Format
2. Standard Error Format
3. Authentication
4. Events
5. Attendance
6. Presence Monitoring
7. Emergency Tickets
8. Activities
9. Notifications
10. Volunteer Blocking
11. Reports
12. Activity History
13. Audit Logs
14. Error Code Reference
15. Status Enums
16. API Contract Change Process
17. API Changelog

# Activities
# Activities

## Overview

The Activities module enables administrators to plan, assign, monitor, and review volunteer work during an event.

Activities are independent work items that may optionally be associated with an attendance session.

Members only see activities assigned to them.

All activity operations follow the business rules defined throughout the project documentation.

---

## Activity Lifecycle

```
Draft
    ↓
Published
    ↓
Assigned
    ↓
In Progress
    ↓
Under Review
    ↓
Verified
```

Activities may also be:

- Cancelled
- Archived

Cancelled activities never become active again.

Archived activities are read-only.

---

## POST /activities

**Purpose:** Create a new activity.

**Auth required:** Yes (admin only)


### Validation Rules

- Title is required.
- Category is required.
- Priority is required.
- Session association is optional.
- Event template is optional.

---

### Response 201

```json
{
  "success": true,
  "message": "Activity created successfully.",
  "data": {
    "activity_id": "uuid",
    "status": "DRAFT"
  }
}
```

---

## GET /activities



## GET /activities/{activityId}

Returns complete activity details.

---

## PATCH /activities/{activityId}/publish

Publishes a draft activity.

Only published activities may be assigned.

---

## POST /activities/{activityId}/duplicate

Creates a new activity by copying the selected activity.

Assignments, evidence, progress, reviews, notifications, and history are **not** copied.

---

## POST /activities/{activityId}/cancel

Cancels an activity.

Only administrators may cancel activities.

Cancelled activities cannot be resumed.

---

## DELETE /activities/{activityId}

Deletes an activity.

Validation Rules:

- Only Draft activities without assignments may be deleted.
- Assigned activities cannot be deleted.
- Cancelled activities cannot be deleted.
- Archived activities cannot be deleted.

---

# Activity Assignment

## POST /activities/{activityId}/assign

**Purpose:** Assign one activity to one or more members.

**Auth required:** Yes (admin only)

### Request

```json
{
  "member_ids": [
    "member_uuid_1",
    "member_uuid_2",
    "member_uuid_3"
  ]
}
```

### Validation Rules

- Activity must be in the **Published** state.
- Only administrators may assign activities.
- Members cannot assign themselves.
- Assignment conflicts caused by overlapping schedules shall prevent assignment.
- Members added to a group after assignment shall not automatically receive existing assignments.

---

### Response 201

```json
{
  "success": true,
  "message": "Activity assigned successfully.",
  "data": {
    "activity_id": "uuid",
    "assigned_members": 3
  }
}
```

---

## GET /activities/{activityId}/assignments

**Purpose:** Retrieve all assignments for a specific activity.

**Auth required:** Yes (admin only)

### Response 200

```json
{
  "success": true,
  "message": "Assignments retrieved successfully.",
  "data": [
    {
      "assignment_id": "uuid",
      "member_id": "uuid",
      "member_name": "Alice",
      "status": "ASSIGNED",
      "assigned_at": "2026-09-01T09:00:00Z"
    }
  ]
}
```

---

## GET /assignments/me

### Response 200

```json
{
  "success": true,
  "message": "Assigned activities retrieved successfully.",
  "data": {
    "items": [
      {
        "assignment_id": "uuid",
        "activity_id": "uuid",
        "title": "Stage Setup",
        "priority": "HIGH",
        "status": "ASSIGNED",
        "due_time": "2026-09-01T09:30:00Z"
      }
    ],
    "pagination": {
      "page": 1,
      "page_size": 20,
      "total_records": 8,
      "total_pages": 1
    }
  }
}
```

---

## DELETE /assignments/{assignmentId}

**Purpose:** Remove an assignment from a member.

**Auth required:** Yes (admin only)

### Validation Rules

- Assignment may only be removed if the activity has not been started.
- Assignments in **In Progress**, **Under Review**, or **Verified** cannot be removed.

---

### Response 200

```json
{
  "success": true,
  "message": "Assignment removed successfully."
}
```

---

## POST /assignments/{assignmentId}/submit

**Purpose:** Submit the completed activity for review.

**Auth required:** Yes (member)

### Validation Rules

- Assignment must be **IN_PROGRESS**.
- At least one evidence item is required.
- Only the assigned member may submit the assignment.
- After submission, the assignment becomes read-only until the review is completed or changes are requested.

---

### Response 200

```json
{
  "success": true,
  "message": "Activity submitted successfully.",
  "data": {
    "assignment_id": "uuid",
    "status": "UNDER_REVIEW",
    "submitted_at": "2026-09-01T10:30:00Z"
  }
}
```

---

## GET /assignments/{assignmentId}

**Purpose:** Retrieve the complete assignment details.

**Auth required:** Yes

### Response 200

```json
{
  "success": true,
  "message": "Assignment retrieved successfully.",
  "data": {
    "assignment_id": "uuid",
    "activity": {
      "id": "uuid",
      "title": "Stage Setup",
      "priority": "HIGH"
    },
    "status": "IN_PROGRESS",
    "started_at": "2026-09-01T09:05:00Z",
    "submitted_at": null
  }
}
```

---

# Evidence Management

Evidence is always associated with an Assignment.

Members upload evidence directly to their assigned activity before submitting it for review.

Only live camera capture is supported. Gallery uploads and manual file uploads are not permitted.

Evidence remains editable until the activity is submitted for review.

---

## POST /assignments/{assignmentId}/photos

**Purpose:** Capture and upload one or more live photos for a progress update.

**Auth required:** Yes (member)

### Request

Multipart Form Data

| Field | Type | Required |
|---|---|---|
| photos | File[] | Yes |

### Validation Rules

- Only the assigned member may upload evidence.
- Photos must be captured using the device camera.
- Gallery uploads are not permitted.
- Maximum of 10 photos per assignment..
- Photos are automatically optimized before storage.
- Evidence cannot be uploaded after the activity is submitted for review.

---

### Response 201

```json
{
  "success": true,
  "message": "Photos uploaded successfully.",
  "data": {
    "uploaded": 4
  }
}
```

---

## POST /assignments/{assignmentId}/videos

**Purpose:** Capture and upload live videos for a progress update.

**Auth required:** Yes (member)

### Request

Multipart Form Data

| Field | Type | Required |
|---|---|---|
| videos | File[] | Yes |

### Validation Rules

- Only live camera recording is permitted.
- Gallery uploads are not permitted.
- Maximum of 2 videos per activity.
- Each video must not exceed 1 minute.
- Videos are automatically optimized before storage.
- Evidence cannot be uploaded after the activity is submitted for review.

---

### Response 201

```json
{
  "success": true,
  "message": "Videos uploaded successfully.",
  "data": {
    "uploaded": 1
  }
}
```

---

## GET /assignments/{assignmentId}/evidence

**Purpose:** Retrieve all evidence associated with a progress update.

**Auth required:** Yes

### Response 200

```json
{
  "success": true,
  "message": "Evidence retrieved successfully.",
  "data": {
    "photos": [
      {
        "id": "uuid",
        "url": "/media/photo1.jpg",
        "captured_at": "2026-09-01T09:15:22Z"
      }
    ],
    "videos": [
      {
        "id": "uuid",
        "url": "/media/video1.mp4",
        "duration_seconds": 42,
        "captured_at": "2026-09-01T09:18:40Z"
      }
    ]
  }
}
```

---

## DELETE /evidence/{evidenceId}

**Purpose:** Delete evidence before activity submission.

**Auth required:** Yes (member)

### Validation Rules

- Only the member who uploaded the evidence may delete it.
- Evidence may only be deleted before the activity is submitted for review.
- Members may replace deleted evidence with newly captured evidence.

---

### Response 200

```json
{
  "success": true,
  "message": "Evidence deleted successfully."
}
```

---

## Evidence Metadata

The system automatically records metadata for every evidence item.

Members cannot modify this information.

### Stored Metadata

- Evidence ID
- Assignment ID
- Captured Timestamp
- Device Information
- GPS Coordinates (when available)
- File Type
- File Size
- Media Duration (videos only)

---

## Business Rules

- Evidence belongs to an assignment.
- Evidence cannot exist without an assignment.
- Gallery uploads are never permitted.
- Manual file uploads are never permitted.
- Members may capture multiple photos and videos before submission.
- After submission for review, all evidence becomes read-only.
- Optimized media shall preserve sufficient quality for administrative review.

---

# Review Workflow

Completed activities enter the review process after submission.

Only administrators can review submitted activities.

Members cannot modify submitted activities unless the reviewer requests changes.

---

## Review Lifecycle

```
Under Review
      │
      ├──────────────► Verified
      │
      └──────────────► Needs Changes
                            │
                            ▼
                       In Progress
                            │
                            ▼
                      Under Review
```

Activities remain in this cycle until they are successfully verified.

---

## GET /reviews/pending



### Response 200

```json
{
  "success": true,
  "message": "Pending reviews retrieved successfully.",
  "data": {
    "items": [
      {
        "assignment_id": "uuid",
        "activity_title": "Stage Setup",
        "member_name": "Alice",
        "submitted_at": "2026-09-01T10:30:00Z",
        "priority": "HIGH"
      }
    ]
  }
}
```

---

## GET /reviews/{assignmentId}

**Purpose:** Retrieve a complete submission for review.

**Auth required:** Yes (admin only)

### Response 200

```json
{
  "success": true,
  "message": "Review details retrieved successfully.",
  "data": {
    "assignment_id": "uuid",
    "activity": {
      "title": "Stage Setup",
      "category": "Logistics"
    },
    "evidence": [],
    "submitted_at": "2026-09-01T10:30:00Z"
  }
}
```

---

## POST /reviews/{assignmentId}/verify

**Purpose:** Verify a completed activity.

**Auth required:** Yes (admin only)

### Request

```json
{
  "remarks": "Excellent work."
}
```

### Validation Rules

- Assignment must be in **Under Review**.
- Remarks are optional.
- Only administrators may verify activities.

---

### Response 200

```json
{
  "success": true,
  "message": "Activity verified successfully.",
  "data": {
    "status": "VERIFIED"
  }
}
```

---

## POST /reviews/{assignmentId}/needs-changes

**Purpose:** Request corrections before approving an activity.

**Auth required:** Yes (admin only)

### Request

```json
{
  "remarks": "Please upload a photo of the completed stage from the front entrance."
}
```

### Validation Rules

- Assignment must be in **Under Review**.
- Review remarks are mandatory.
- Only administrators may request changes.

---

### Response 200

```json
{
  "success": true,
  "message": "Review completed. Changes requested.",
  "data": {
    "status": "NEEDS_CHANGES"
  }
}
```

---

## GET /reviews/history

**Purpose:** Retrieve completed review history.

**Auth required:** Yes (admin only)

### Optional Filters

- reviewer
- member
- event
- session
- status
- category

---

### Response 200

```json
{
  "success": true,
  "message": "Review history retrieved successfully.",
  "data": {
    "items": [
      {
        "assignment_id": "uuid",
        "member_name": "Alice",
        "review_status": "VERIFIED",
        "reviewed_at": "2026-09-01T11:10:00Z"
      }
    ]
  }
}
```

---

## Business Rules

- Only administrators may review activities.
- Members cannot review their own submissions.
- Verified activities become read-only.
- Selecting **Needs Changes** requires review remarks.
- Members respond to **Needs Changes** by uploading additional evidence and resubmitting the assignment.
- Previously submitted evidence is preserved for audit purposes.
- Every review action is recorded in the audit log.

---

# Activity Templates

Activity Templates help administrators quickly create commonly used activities for recurring event types.

Templates define the basic structure of an activity but do not include assignments, evidence, reviews, notifications, or historical data.
---

## GET /activity-templates

**Purpose:** Retrieve all available activity templates.

**Auth required:** Yes (admin only)

### Optional Filters

- event_type
- category
- status

---

### Response 200

```json
{
  "success": true,
  "message": "Activity templates retrieved successfully.",
  "data": {
    "items": [
      {
        "template_id": "uuid",
        "name": "Workshop Template",
        "event_type": "Workshop",
        "activities": 8,
        "created_at": "2026-09-01T09:00:00Z"
      }
    ]
  }
}
```

---

## POST /activity-templates

**Purpose:** Create a new activity template.

**Auth required:** Yes (admin only)

### Request

```json
{
  "name": "Workshop Template",
  "event_type": "Workshop",
  "activities": [
    {
      "title": "Stage Setup",
      "category": "Logistics",
      "priority": "HIGH"
    },
    {
      "title": "Photography",
      "category": "Media",
      "priority": "MEDIUM"
    }
  ]
}
```

### Validation Rules

- Template name is required.
- Event type is required.
- At least one activity is required.
- Activity titles must be unique within the template.

---

### Response 201

```json
{
  "success": true,
  "message": "Activity template created successfully.",
  "data": {
    "template_id": "uuid"
  }
}
```

---

## GET /activity-templates/{templateId}

**Purpose:** Retrieve complete template details.

**Auth required:** Yes (admin only)

### Response 200

```json
{
  "success": true,
  "message": "Activity template retrieved successfully.",
  "data": {
    "template_id": "uuid",
    "name": "Workshop Template",
    "event_type": "Workshop",
    "activities": [
      {
        "title": "Stage Setup",
        "category": "Logistics",
        "priority": "HIGH"
      }
    ]
  }
}
```

---

## PATCH /activity-templates/{templateId}

**Purpose:** Update an existing activity template.

**Auth required:** Yes (admin only)

### Validation Rules

- Templates may only be edited by administrators.
- Updating a template does not modify activities already created from it.

---

### Response 200

```json
{
  "success": true,
  "message": "Activity template updated successfully."
}
```

---

## DELETE /activity-templates/{templateId}

**Purpose:** Delete an activity template.

**Auth required:** Yes (admin only)

### Validation Rules

- Templates currently being used by active events cannot be deleted.
- Deleted templates do not affect activities that were previously created from them.

---

### Response 200

```json
{
  "success": true,
  "message": "Activity template deleted successfully."
}
```

---

## POST /activity-templates/{templateId}/apply


### Validation Rules

- Event ID is required.
- Session association is optional.
- Activities created from a template are created as **Draft** activities.
- Administrators may modify the generated draft activities before publishing.

---

### Response 201

```json
{
  "success": true,
  "message": "Activity template applied successfully.",
  "data": {
    "activities_created": 8
  }
}
```

---

## Business Rules

- Each event type should typically maintain one standard template.
- Administrators may update the template over time as organizational needs evolve.
- Templates define the initial structure of activities.
- Assignments are never stored in templates.
- Evidence is never stored in templates.
- Reviews are never stored in templates.
- Templates are intended to reduce repetitive activity creation for recurring event types.

---

# Activity Reports

Activity reports extend the attendance reports introduced in earlier phases by providing detailed information about activity planning, execution, evidence, and review.

Reports are generated on demand and may be exported for auditing and record keeping.

---

## GET /reports/activities.xlsx

**Purpose:** Export activities for one or more events.

**Auth required:** Yes (admin only)

### Optional Query Parameters

| Parameter | Description |
|---|---|
| event_id | Filter by event |
| category | Filter by activity category |
| priority | Filter by priority |
| status | Filter by activity status |
| assigned_member | Filter by member |
| review_status | Filter by review result |

---

### Response 200

Returns an Excel workbook.

### Workbook Sheets

- Activity Summary
- Assignment Summary
- Evidence Summary
- Review Summary
- Activity Statistics

---

## GET /reports/activities/{activityId}.xlsx

**Purpose:** Export a complete report for a single activity.

**Auth required:** Yes (admin only)

### Response 200

Returns an Excel workbook containing:

- Activity Details
- Assignment Details
- Evidence
- Review Information
- Activity Audit Summary

---

## GET /reports/activity-statistics



### Response 200

```json
{
  "success": true,
  "message": "Activity statistics retrieved successfully.",
  "data": {
    "total_activities": 120,
    "assigned": 118,
    "in_progress": 27,
    "under_review": 8,
    "verified": 74,
    "needs_changes": 9,
    "cancelled": 2
  }
}
```

---

## Business Rules

- Reports are generated on demand.
- Only administrators may export reports.
- Archived activities remain available for reporting.
- Reports include verified, needs changes, cancelled, and archived activities.
- Report contents always reflect the latest approved data.

---

# Activity Error Codes

| Code | HTTP Status | Used In | Meaning |
|---|---|---|---|
| ACTIVITY_NOT_FOUND | 404 | Activities | Activity does not exist |
| ACTIVITY_NOT_PUBLISHED | 422 | Assignment | Activity must be published before assignment |
| ACTIVITY_ALREADY_STARTED | 409 | Start Activity | Activity has already been started |
| ACTIVITY_ALREADY_SUBMITTED | 409 | Submit Activity | Activity has already been submitted |
| ASSIGNMENT_CONFLICT | 409 | Assignment | Member has a conflicting assignment |
| ASSIGNMENT_NOT_FOUND | 404 | Assignments | Assignment not found |
| ACTIVITY_NOT_ASSIGNED | 403 | Execution | Activity is not assigned to this member |
| EVIDENCE_REQUIRED | 422 | Submit Activity | At least one evidence item is required |
| PHOTO_LIMIT_EXCEEDED | 422 | Evidence | Maximum photo limit exceeded |
| VIDEO_LIMIT_EXCEEDED | 422 | Evidence | Maximum video limit exceeded |
| VIDEO_DURATION_EXCEEDED | 422 | Evidence | Video exceeds the allowed duration |
| LIVE_CAPTURE_REQUIRED | 422 | Evidence | Gallery uploads and manual file uploads are not permitted |
| REVIEW_ALREADY_COMPLETED | 409 | Review | Review has already been completed |
| REVIEW_REMARK_REQUIRED | 422 | Review | Remarks are required when requesting changes |
| TEMPLATE_NOT_FOUND | 404 | Templates | Activity template not found |
| TEMPLATE_IN_USE | 409 | Templates | Template cannot be deleted while in use |

---

# Activity Status Enums

## Activity Status

- DRAFT
- PUBLISHED
- CANCELLED
- ARCHIVED

---

## Assignment Status

- ASSIGNED
- IN_PROGRESS
- UNDER_REVIEW
- VERIFIED
- NEEDS_CHANGES

---

## Priority

- LOW
- MEDIUM
- HIGH
- CRITICAL

---

## Evidence Type

- PHOTO
- VIDEO

---

## Review Decision

- VERIFIED
- NEEDS_CHANGES

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

Phase 3 
# Activity Management

## Administrator

POST   /activities

GET    /activities

GET    /activities/{activityId}

PATCH  /activities/{activityId}

POST   /activities/{activityId}/publish

POST   /activities/{activityId}/cancel

POST   /activities/{activityId}/archive

---

# Activity Assignments

## Administrator

POST   /activities/{activityId}/assignments

GET    /activities/{activityId}/assignments

PATCH  /assignments/{assignmentId}

POST   /assignments/{assignmentId}/cancel

---

## Member

GET    /members/me/assignments

GET    /assignments/{assignmentId}

---

# Activity Progress

## Member

POST   /assignments/{assignmentId}/start

POST   /assignments/{assignmentId}/progress

GET    /assignments/{assignmentId}/progress

POST   /assignments/{assignmentId}/submit

---

# Activity Evidence

## Member

POST   /progress/{progressId}/evidence

GET    /progress/{progressId}/evidence

DELETE /progress/{progressId}/evidence/{evidenceId}

---

# Activity Review

## Administrator

GET    /reviews/pending

GET    /reviews/{reviewId}

POST   /reviews/{reviewId}/verify

POST   /reviews/{reviewId}/needs-changes

---

## Member

GET    /assignments/{assignmentId}/review

---

# Activity Templates

## Administrator

POST   /activity-templates

GET    /activity-templates

GET    /activity-templates/{templateId}

PATCH  /activity-templates/{templateId}

POST   /activity-templates/{templateId}/apply

POST   /activity-templates/{templateId}/archive

---

# Activity Reports

## Administrator

POST   /activity-reports

GET    /activity-reports/{reportId}

GET    /activity-reports/summary

---

# Activity History

## Member

GET    /members/me/activity-history

GET    /assignments/{assignmentId}/history

---

## Administrator

GET    /activities/{activityId}/history


# Create Activity

Creates a new activity in **Draft** status. Activities remain editable until they are published.

## Endpoint

`POST /activities`

## Authentication

**Required**

Administrator access only.

## Request Body

```json
{
  "title": "Campus Clean-Up Drive",
  "description": "Monthly campus cleaning initiative.",
  "category": "Community Service",
  "location": "Main Campus",
  "startTime": "2026-09-15T09:00:00Z",
  "endTime": "2026-09-15T12:00:00Z",
  "maxParticipants": 40,
  "requiresEvidence": true,
  "minimumPhotoCount": 2,
  "maximumPhotoCount": 10,
  "minimumVideoCount": 0,
  "maximumVideoCount": 2,
  "maximumVideoDurationSeconds": 60,
  "instructions": [
    "Wear your volunteer ID.",
    "Report to the event coordinator before starting."
  ]
}
```

## Request Fields

| Field | Type | Required | Description |
|--------|------|----------|-------------|
| title | String | Yes | Activity title. |
| description | String | Yes | Detailed description of the activity. |
| category | String | Yes | Activity category. |
| location | String | Yes | Activity location or venue. |
| startTime | ISO-8601 Datetime | Yes | Planned activity start date and time. |
| endTime | ISO-8601 Datetime | Yes | Planned activity end date and time. |
| maxParticipants | Integer | Yes | Maximum number of members that may be assigned. |
| requiresEvidence | Boolean | Yes | Indicates whether evidence submission is mandatory. |
| minimumPhotoCount | Integer | No | Minimum number of required photographs. |
| maximumPhotoCount | Integer | No | Maximum number of allowed photographs. Maximum: 10. |
| minimumVideoCount | Integer | No | Minimum number of required videos. |
| maximumVideoCount | Integer | No | Maximum number of allowed videos. Maximum: 2. |
| maximumVideoDurationSeconds | Integer | No | Maximum duration of each uploaded video in seconds. Maximum: 60. |
| instructions | Array<String> | No | Instructions displayed to assigned members. |

## Validation Rules

- Title cannot be empty.
- Description cannot be empty.
- Start time must be earlier than end time.
- Maximum participants must be greater than zero.
- Maximum photo count cannot exceed **10**.
- Maximum video count cannot exceed **2**.
- Maximum video duration cannot exceed **60 seconds**.
- Minimum values cannot exceed their corresponding maximum values.
- Activities are always created in the **DRAFT** state.

## Successful Response

**HTTP 201 Created**

```json
{
  "activityId": "a8b2d0c4-76d9-4db7-a58f-3d2dbf6b96c8",
  "status": "DRAFT",
  "message": "Activity created successfully."
}
```

## Business Rules

- Newly created activities always begin in the **DRAFT** state.
- Only administrators may create activities.
- Draft activities are visible only to administrators.
- Draft activities may be edited.
- Draft activities cannot be assigned to members.
- Draft activities cannot receive progress updates.
- Draft activities must be published before they become available for assignment.
- Activity creation is automatically recorded in the Audit Log.

## Possible Errors

| HTTP Status | Description |
|-------------|-------------|
| 400 | Invalid request body. |
| 401 | Authentication required. |
| 403 | Administrator permission required. |
| 409 | Activity schedule conflicts with an existing activity. |
| 422 | Validation failed. |
| 500 | Internal server error. |

# List Activities

Returns a paginated list of activities available to the authenticated administrator.

## Endpoint

`GET /activities`

## Authentication

**Required**

Administrator access only.

## Query Parameters

| Parameter | Type | Required | Description |
|----------|------|----------|-------------|
| page | Integer | No | Page number. Default: `1`. |
| pageSize | Integer | No | Number of records per page. Default: `20`. |
| status | String | No | Filter by activity status (`DRAFT`, `ACTIVE`, `CANCELLED`, `ARCHIVED`). |
| category | String | No | Filter by activity category. |
| search | String | No | Search by activity title or description. |
| startDate | ISO-8601 Datetime | No | Return activities starting on or after this date. |
| endDate | ISO-8601 Datetime | No | Return activities ending on or before this date. |
| sortBy | String | No | Sort field. Default: `startTime`. |
| sortOrder | String | No | `asc` or `desc`. Default: `asc`. |

## Successful Response

**HTTP 200 OK**

```json
{
  "page": 1,
  "pageSize": 20,
  "totalRecords": 42,
  "totalPages": 3,
  "activities": [
    {
      "activityId": "a8b2d0c4-76d9-4db7-a58f-3d2dbf6b96c8",
      "title": "Campus Clean-Up Drive",
      "category": "Community Service",
      "location": "Main Campus",
      "startTime": "2026-09-15T09:00:00Z",
      "endTime": "2026-09-15T12:00:00Z",
      "status": "ACTIVE",
      "assignedMembers": 28,
      "maxParticipants": 40
    }
  ]
}
```

## Business Rules

- Only administrators may retrieve the complete activity list.
- Activities are returned using pagination.
- Multiple filters may be combined.
- Search performs a partial match on supported fields.
- Results may be sorted using supported sort fields.
- Archived activities remain searchable.
- Cancelled activities remain visible for historical purposes.

## Possible Errors

| HTTP Status | Description |
|-------------|-------------|
| 401 | Authentication required. |
| 403 | Administrator permission required. |
| 422 | Invalid query parameters. |
| 500 | Internal server error. |

# Get Activity Details

Returns the complete details of a single activity.

## Endpoint

`GET /activities/{activityId}`

## Authentication

**Required**

Administrator access only.

## Path Parameters

| Parameter | Type | Description |
|----------|------|-------------|
| activityId | UUID | Unique Activity identifier. |

## Successful Response

**HTTP 200 OK**

```json
{
  "activityId": "a8b2d0c4-76d9-4db7-a58f-3d2dbf6b96c8",
  "title": "Campus Clean-Up Drive",
  "description": "Monthly campus cleaning initiative.",
  "category": "Community Service",
  "location": "Main Campus",
  "startTime": "2026-09-15T09:00:00Z",
  "endTime": "2026-09-15T12:00:00Z",
  "status": "ACTIVE",
  "maxParticipants": 40,
  "assignedMembers": 28,
  "requiresEvidence": true,
  "minimumPhotoCount": 2,
  "maximumPhotoCount": 10,
  "minimumVideoCount": 0,
  "maximumVideoCount": 2,
  "maximumVideoDurationSeconds": 60,
  "instructions": [
    "Wear your volunteer ID.",
    "Report to the event coordinator before starting."
  ],
  "createdAt": "2026-08-01T10:15:30Z",
  "updatedAt": "2026-08-03T14:42:18Z",
  "createdBy": {
    "userId": "2e4e2b60-88b0-4b07-b8b8-1d8d6d76d2c9",
    "name": "John Doe"
  }
}
```

## Business Rules

- Only administrators may retrieve complete activity details.
- The response contains the latest activity configuration.
- Draft, Active, Cancelled and Archived activities may all be retrieved.
- Assigned member count reflects the current assignment state.
- Historical activities remain accessible for reporting and auditing purposes.

## Possible Errors

| HTTP Status | Description |
|-------------|-------------|
| 401 | Authentication required. |
| 403 | Administrator permission required. |
| 404 | Activity not found. |
| 500 | Internal server error. |

# Update Activity

Updates an existing activity.

Only activities in the **DRAFT** state may be updated.

## Endpoint

`PATCH /activities/{activityId}`

## Authentication

**Required**

Administrator access only.

## Path Parameters

| Parameter | Type | Description |
|----------|------|-------------|
| activityId | UUID | Unique Activity identifier. |

## Request Body

```json
{
  "title": "Campus Clean-Up Drive 2026",
  "description": "Updated activity description.",
  "category": "Community Service",
  "location": "Main Campus",
  "startTime": "2026-09-15T09:30:00Z",
  "endTime": "2026-09-15T12:30:00Z",
  "maxParticipants": 50,
  "requiresEvidence": true,
  "minimumPhotoCount": 2,
  "maximumPhotoCount": 10,
  "minimumVideoCount": 0,
  "maximumVideoCount": 2,
  "maximumVideoDurationSeconds": 60,
  "instructions": [
    "Arrive 15 minutes before the activity starts.",
    "Wear your volunteer ID."
  ]
}
```

## Request Fields

| Field | Type | Required | Description |
|--------|------|----------|-------------|
| title | String | Yes | Activity title. |
| description | String | Yes | Detailed description of the activity. |
| category | String | Yes | Activity category. |
| location | String | Yes | Activity location or venue. |
| startTime | ISO-8601 Datetime | Yes | Planned activity start date and time. |
| endTime | ISO-8601 Datetime | Yes | Planned activity end date and time. |
| maxParticipants | Integer | Yes | Maximum number of members that may be assigned. |
| requiresEvidence | Boolean | Yes | Indicates whether evidence submission is mandatory. |
| minimumPhotoCount | Integer | No | Minimum number of required photographs. |
| maximumPhotoCount | Integer | No | Maximum number of allowed photographs. Maximum: 10. |
| minimumVideoCount | Integer | No | Minimum number of required videos. |
| maximumVideoCount | Integer | No | Maximum number of allowed videos. Maximum: 2. |
| maximumVideoDurationSeconds | Integer | No | Maximum duration of each uploaded video in seconds. Maximum: 60. |
| instructions | Array<String> | No | Instructions displayed to assigned members. |

## Validation Rules

- Activity must exist.
- Only activities in the **DRAFT** state may be updated.
- Title cannot be empty.
- Description cannot be empty.
- Start time must be earlier than end time.
- Maximum participants must be greater than zero.
- Maximum photo count cannot exceed **10**.
- Maximum video count cannot exceed **2**.
- Maximum video duration cannot exceed **60 seconds**.
- Minimum values cannot exceed their corresponding maximum values.

## Successful Response

**HTTP 200 OK**

```json
{
  "activityId": "a8b2d0c4-76d9-4db7-a58f-3d2dbf6b96c8",
  "status": "DRAFT",
  "message": "Activity updated successfully."
}
```

## Business Rules

- Only administrators may update activities.
- Only Draft activities may be updated.
- Published (Active), Cancelled, and Archived activities cannot be modified.
- Updating an activity does not change its status.
- Every successful update is automatically recorded in the Audit Log.

## Possible Errors

| HTTP Status | Description |
|-------------|-------------|
| 400 | Invalid request body. |
| 401 | Authentication required. |
| 403 | Administrator permission required. |
| 404 | Activity not found. |
| 409 | Activity cannot be updated in its current state. |
| 422 | Validation failed. |
| 500 | Internal server error. |

# Publish Activity

Publishes a Draft activity and makes it available for member assignments.

Only activities in the **DRAFT** state may be published.

## Endpoint

`POST /activities/{activityId}/publish`

## Authentication

**Required**

Administrator access only.

## Path Parameters

| Parameter | Type | Description |
|----------|------|-------------|
| activityId | UUID | Unique Activity identifier. |

## Successful Response

**HTTP 200 OK**

```json
{
  "activityId": "a8b2d0c4-76d9-4db7-a58f-3d2dbf6b96c8",
  "status": "ACTIVE",
  "message": "Activity published successfully."
}
```

## Business Rules

- Only administrators may publish activities.
- Only activities in the **DRAFT** state may be published.
- Publishing changes the activity status from **DRAFT** to **ACTIVE**.
- Published activities become visible to eligible members.
- Published activities become available for member assignment.
- Published activities can receive progress updates and evidence submissions after members are assigned.
- Publishing an activity is automatically recorded in the Audit Log.

## Possible Errors

| HTTP Status | Description |
|-------------|-------------|
| 401 | Authentication required. |
| 403 | Administrator permission required. |
| 404 | Activity not found. |
| 409 | Activity is not in the Draft state. |
| 500 | Internal server error. |

# Cancel Activity

Cancels an activity and prevents any further participation.

Only activities in the **DRAFT** or **ACTIVE** state may be cancelled.

## Endpoint

`POST /activities/{activityId}/cancel`

## Authentication

**Required**

Administrator access only.

## Path Parameters

| Parameter | Type | Description |
|----------|------|-------------|
| activityId | UUID | Unique Activity identifier. |

## Request Body

```json
{
  "reason": "Event postponed due to severe weather conditions."
}
```

## Request Fields

| Field | Type | Required | Description |
|--------|------|----------|-------------|
| reason | String | Yes | Reason for cancelling the activity. |

## Validation Rules

- Activity must exist.
- Only activities in the **DRAFT** or **ACTIVE** state may be cancelled.
- Cancellation reason cannot be empty.

## Successful Response

**HTTP 200 OK**

```json
{
  "activityId": "a8b2d0c4-76d9-4db7-a58f-3d2dbf6b96c8",
  "status": "CANCELLED",
  "message": "Activity cancelled successfully."
}
```

## Business Rules

- Only administrators may cancel activities.
- Cancelling changes the activity status to **CANCELLED**.
- Cancelled activities cannot receive new member assignments.
- Members with existing assignments can no longer submit progress or evidence for the cancelled activity.
- Cancelled activities remain available for reporting, history, and auditing purposes.
- Activity cancellation is automatically recorded in the Audit Log.

## Possible Errors

| HTTP Status | Description |
|-------------|-------------|
| 400 | Invalid request body. |
| 401 | Authentication required. |
| 403 | Administrator permission required. |
| 404 | Activity not found. |
| 409 | Activity cannot be cancelled in its current state. |
| 422 | Validation failed. |
| 500 | Internal server error. |

# Archive Activity

Archives an activity for long-term retention.

Archived activities become read-only and remain available for reporting, history, and auditing purposes.

Only activities in the **DRAFT**, **ACTIVE**, or **CANCELLED** state may be archived.

## Endpoint

`POST /activities/{activityId}/archive`

## Authentication

**Required**

Administrator access only.

## Path Parameters

| Parameter | Type | Description |
|----------|------|-------------|
| activityId | UUID | Unique Activity identifier. |

## Successful Response

**HTTP 200 OK**

```json
{
  "activityId": "a8b2d0c4-76d9-4db7-a58f-3d2dbf6b96c8",
  "status": "ARCHIVED",
  "message": "Activity archived successfully."
}
```

## Business Rules

- Only administrators may archive activities.
- Activities in the **DRAFT**, **ACTIVE**, and **CANCELLED** states may be archived.
- Archiving changes the activity status to **ARCHIVED**.
- Archived activities become read-only.
- Archived activities cannot be modified.
- Archived activities cannot receive new member assignments.
- Archived activities cannot receive progress updates.
- Archived activities cannot receive evidence submissions.
- Archived activities remain available in reports.
- Archived activities remain accessible through Activity History.
- Activity archival is automatically recorded in the Audit Log.

## Possible Errors

| HTTP Status | Description |
|-------------|-------------|
| 401 | Authentication required. |
| 403 | Administrator permission required. |
| 404 | Activity not found. |
| 409 | Activity cannot be archived in its current state. |
| 500 | Internal server error. |

# Assign Members to Activity

Assigns one or more members to an active activity.

Only **ACTIVE** activities may receive new assignments.

## Endpoint

`POST /activities/{activityId}/assignments`

## Authentication

**Required**

Administrator access only.

## Path Parameters

| Parameter | Type | Description |
|----------|------|-------------|
| activityId | UUID | Unique Activity identifier. |

## Request Body

```json
{
  "memberIds": [
    "3b9d4e8f-86d8-4b52-9d6d-0b1ef61d5d18",
    "62d0d4d7-6b3d-4a5b-b7c4-df47a7d2c80f"
  ],
  "dueDate": "2026-09-20T18:00:00Z",
  "notes": "Complete the assigned tasks before the due date."
}
```

## Request Fields

| Field | Type | Required | Description |
|--------|------|----------|-------------|
| memberIds | Array<UUID> | Yes | List of members to assign. |
| dueDate | ISO-8601 Datetime | No | Assignment completion deadline. |
| notes | String | No | Additional instructions for assigned members. |

## Validation Rules

- Activity must exist.
- Activity must be in the **ACTIVE** state.
- At least one member must be selected.
- All selected members must exist.
- Duplicate member assignments are not permitted.
- Total assignments cannot exceed the activity's maximum participant limit.
- Members who are temporarily restricted from volunteering cannot be assigned.
- Due date, if provided, must not be earlier than the assignment creation time.

## Successful Response

**HTTP 201 Created**

```json
{
  "activityId": "a8b2d0c4-76d9-4db7-a58f-3d2dbf6b96c8",
  "assignedCount": 2,
  "message": "Members assigned successfully."
}
```

## Business Rules

- Only administrators may assign members.
- Members may only be assigned to **ACTIVE** activities.
- A member cannot receive duplicate assignments for the same activity.
- Assignment notifications are automatically generated for newly assigned members.
- Every successful assignment is automatically recorded in the Audit Log.

## Possible Errors

| HTTP Status | Description |
|-------------|-------------|
| 400 | Invalid request body. |
| 401 | Authentication required. |
| 403 | Administrator permission required. |
| 404 | Activity or member not found. |
| 409 | Assignment conflict or maximum participant limit reached. |
| 422 | Validation failed. |
| 500 | Internal server error. |

# Get Activity Assignments

Returns all assignments associated with a specific activity.

## Endpoint

`GET /activities/{activityId}/assignments`

## Authentication

**Required**

Administrator access only.

## Path Parameters

| Parameter | Type | Description |
|----------|------|-------------|
| activityId | UUID | Unique Activity identifier. |

## Query Parameters

| Parameter | Type | Required | Description |
|----------|------|----------|-------------|
| page | Integer | No | Page number. Default: `1`. |
| pageSize | Integer | No | Number of records per page. Default: `20`. |
| status | String | No | Filter by assignment status. |
| search | String | No | Search by member name or email. |
| sortBy | String | No | Sort field. Default: `assignedAt`. |
| sortOrder | String | No | `asc` or `desc`. Default: `desc`. |

## Successful Response

**HTTP 200 OK**

```json
{
  "page": 1,
  "pageSize": 20,
  "totalRecords": 18,
  "totalPages": 1,
  "assignments": [
    {
      "assignmentId": "1f23d4c5-6a78-4e8a-b9c1-2d3456789012",
      "member": {
        "userId": "3b9d4e8f-86d8-4b52-9d6d-0b1ef61d5d18",
        "name": "John Doe",
        "email": "john@example.com"
      },
      "status": "ASSIGNED",
      "assignedAt": "2026-09-10T09:15:00Z",
      "dueDate": "2026-09-20T18:00:00Z"
    }
  ]
}
```

## Business Rules

- Only administrators may retrieve activity assignments.
- Assignments are returned using pagination.
- Multiple filters may be combined.
- Search performs a partial match on member name and email.
- Assignment history remains available even after an activity is cancelled or archived.

## Possible Errors

| HTTP Status | Description |
|-------------|-------------|
| 401 | Authentication required. |
| 403 | Administrator permission required. |
| 404 | Activity not found. |
| 422 | Invalid query parameters. |
| 500 | Internal server error. |

# Update Assignment

Updates an existing activity assignment.

This endpoint allows administrators to modify assignment details before the assignment is completed or cancelled.

## Endpoint

`PATCH /assignments/{assignmentId}`

## Authentication

**Required**

Administrator access only.

## Path Parameters

| Parameter | Type | Description |
|----------|------|-------------|
| assignmentId | UUID | Unique Assignment identifier. |

## Request Body

```json
{
  "dueDate": "2026-09-22T18:00:00Z",
  "notes": "Please coordinate with the team leader before starting."
}
```

## Request Fields

| Field | Type | Required | Description |
|--------|------|----------|-------------|
| dueDate | ISO-8601 Datetime | No | Updated assignment completion deadline. |
| notes | String | No | Updated instructions for the assigned member. |

## Validation Rules

- Assignment must exist.
- Completed assignments cannot be updated.
- Cancelled assignments cannot be updated.
- Due date, if provided, must not be earlier than the assignment creation time.

## Successful Response

**HTTP 200 OK**

```json
{
  "assignmentId": "1f23d4c5-6a78-4e8a-b9c1-2d3456789012",
  "message": "Assignment updated successfully."
}
```

## Business Rules

- Only administrators may update assignments.
- Updating an assignment does not change its current status.
- Assignment history is preserved after every update.
- Every successful update is automatically recorded in the Audit Log.
- Assigned members are notified when assignment details are updated.

## Possible Errors

| HTTP Status | Description |
|-------------|-------------|
| 400 | Invalid request body. |
| 401 | Authentication required. |
| 403 | Administrator permission required. |
| 404 | Assignment not found. |
| 409 | Assignment cannot be updated in its current state. |
| 422 | Validation failed. |
| 500 | Internal server error. |

# Cancel Assignment

Cancels an existing activity assignment.

Only assignments that have not been completed or cancelled may be cancelled.

## Endpoint

`POST /assignments/{assignmentId}/cancel`

## Authentication

**Required**

Administrator access only.

## Path Parameters

| Parameter | Type | Description |
|----------|------|-------------|
| assignmentId | UUID | Unique Assignment identifier. |

## Request Body

```json
{
  "reason": "Volunteer is unavailable for the scheduled activity."
}
```

## Request Fields

| Field | Type | Required | Description |
|--------|------|----------|-------------|
| reason | String | Yes | Reason for cancelling the assignment. |

## Validation Rules

- Assignment must exist.
- Cancellation reason cannot be empty.
- Completed assignments cannot be cancelled.
- Cancelled assignments cannot be cancelled again.

## Successful Response

**HTTP 200 OK**

```json
{
  "assignmentId": "1f23d4c5-6a78-4e8a-b9c1-2d3456789012",
  "status": "CANCELLED",
  "message": "Assignment cancelled successfully."
}
```

## Business Rules

- Only administrators may cancel assignments.
- Cancelling an assignment changes its status to **CANCELLED**.
- Cancelled assignments cannot receive progress updates.
- Cancelled assignments cannot receive evidence submissions.
- Cancelled assignments cannot be submitted for review.
- Assignment history remains available after cancellation.
- Assigned members are notified when an assignment is cancelled.
- Assignment cancellation is automatically recorded in the Audit Log.

## Possible Errors

| HTTP Status | Description |
|-------------|-------------|
| 400 | Invalid request body. |
| 401 | Authentication required. |
| 403 | Administrator permission required. |
| 404 | Assignment not found. |
| 409 | Assignment cannot be cancelled in its current state. |
| 422 | Validation failed. |
| 500 | Internal server error. |

# Get My Assignments

Returns a paginated list of activities assigned to the authenticated member.

This endpoint allows members to view both current and historical assignments.

## Endpoint

`GET /members/me/assignments`

## Authentication

**Required**

Member access only.

## Query Parameters

| Parameter | Type | Required | Description |
|----------|------|----------|-------------|
| page | Integer | No | Page number. Default: `1`. |
| pageSize | Integer | No | Number of records per page. Default: `20`. |
| status | String | No | Filter by assignment status. |
| category | String | No | Filter by activity category. |
| search | String | No | Search by activity title. |
| startDate | ISO-8601 Datetime | No | Return assignments starting on or after this date. |
| endDate | ISO-8601 Datetime | No | Return assignments ending on or before this date. |
| sortBy | String | No | Sort field. Default: `startTime`. |
| sortOrder | String | No | `asc` or `desc`. Default: `asc`. |

## Successful Response

**HTTP 200 OK**

```json
{
  "page": 1,
  "pageSize": 20,
  "totalRecords": 8,
  "totalPages": 1,
  "assignments": [
    {
      "assignmentId": "1f23d4c5-6a78-4e8a-b9c1-2d3456789012",
      "activityId": "a8b2d0c4-76d9-4db7-a58f-3d2dbf6b96c8",
      "title": "Campus Clean-Up Drive",
      "category": "Community Service",
      "location": "Main Campus",
      "startTime": "2026-09-15T09:00:00Z",
      "endTime": "2026-09-15T12:00:00Z",
      "status": "ASSIGNED",
      "dueDate": "2026-09-20T18:00:00Z"
    }
  ]
}
```

## Business Rules

- Members may retrieve only their own assignments.
- Assignments are returned using pagination.
- Multiple filters may be combined.
- Search performs a partial match on activity title.
- Both active and historical assignments are returned.
- Assignment details reflect the latest activity information available to the member.

## Possible Errors

| HTTP Status | Description |
|-------------|-------------|
| 401 | Authentication required. |
| 403 | Member permission required. |
| 422 | Invalid query parameters. |
| 500 | Internal server error. |

# Get Assignment Details

Returns the complete details of a specific assignment for the authenticated member.

This endpoint allows members to view assignment information, progress, submission status, review status, and activity instructions.

## Endpoint

`GET /assignments/{assignmentId}`

## Authentication

**Required**

Member access only.

## Path Parameters

| Parameter | Type | Description |
|----------|------|-------------|
| assignmentId | UUID | Unique Assignment identifier. |

## Successful Response

**HTTP 200 OK**

```json
{
  "assignmentId": "1f23d4c5-6a78-4e8a-b9c1-2d3456789012",
  "status": "IN_PROGRESS",
  "assignedAt": "2026-09-10T09:15:00Z",
  "dueDate": "2026-09-20T18:00:00Z",
  "notes": "Complete the assigned tasks before the due date.",
  "activity": {
    "activityId": "a8b2d0c4-76d9-4db7-a58f-3d2dbf6b96c8",
    "title": "Campus Clean-Up Drive",
    "description": "Monthly campus cleaning initiative.",
    "category": "Community Service",
    "location": "Main Campus",
    "startTime": "2026-09-15T09:00:00Z",
    "endTime": "2026-09-15T12:00:00Z",
    "requiresEvidence": true,
    "minimumPhotoCount": 2,
    "maximumPhotoCount": 10,
    "minimumVideoCount": 0,
    "maximumVideoCount": 2,
    "maximumVideoDurationSeconds": 60,
    "instructions": [
      "Wear your volunteer ID.",
      "Report to the event coordinator before starting."
    ]
  },
  "progress": {
    "completionPercentage": 45,
    "lastUpdated": "2026-09-16T11:42:18Z"
  },
  "reviewStatus": null
}
```

## Business Rules

- Members may retrieve only their own assignments.
- Administrators may retrieve any assignment.
- Assignment details always reflect the latest activity configuration available to the assigned member.
- Progress information is included when available.
- Review status is returned after the assignment has been submitted for review.
- Assignment history is preserved throughout the assignment lifecycle.

## Possible Errors

| HTTP Status | Description |
|-------------|-------------|
| 401 | Authentication required. |
| 403 | Permission denied. |
| 404 | Assignment not found. |
| 500 | Internal server error. |

# Start Activity

Marks an assigned activity as started.

Only assignments in the **ASSIGNED** state may be started.

## Endpoint

`POST /assignments/{assignmentId}/start`

## Authentication

**Required**

Member access only.

## Path Parameters

| Parameter | Type | Description |
|----------|------|-------------|
| assignmentId | UUID | Unique Assignment identifier. |

## Successful Response

**HTTP 200 OK**

```json
{
  "assignmentId": "1f23d4c5-6a78-4e8a-b9c1-2d3456789012",
  "status": "IN_PROGRESS",
  "startedAt": "2026-09-15T09:05:32Z",
  "message": "Activity started successfully."
}
```

## Business Rules

- Members may start only their own assignments.
- Only assignments in the **ASSIGNED** state may be started.
- Starting an assignment changes its status from **ASSIGNED** to **IN_PROGRESS**.
- An assignment can only be started once.
- Progress updates and evidence uploads are permitted only after the assignment has started.
- The activity must be in the **ACTIVE** state.
- Activity start is automatically recorded in the Audit Log.

## Possible Errors

| HTTP Status | Description |
|-------------|-------------|
| 401 | Authentication required. |
| 403 | Permission denied. |
| 404 | Assignment not found. |
| 409 | Assignment cannot be started in its current state. |
| 500 | Internal server error. |

# Update Activity Progress

Updates the progress of an activity assignment.

Only assignments in the **IN_PROGRESS** state may receive progress updates.

## Endpoint

`POST /assignments/{assignmentId}/progress`

## Authentication

**Required**

Member access only.

## Path Parameters

| Parameter | Type | Description |
|----------|------|-------------|
| assignmentId | UUID | Unique Assignment identifier. |

## Request Body

```json
{
  "completionPercentage": 60,
  "statusNote": "Completed registration desk setup and volunteer briefing."
}
```

## Request Fields

| Field | Type | Required | Description |
|--------|------|----------|-------------|
| completionPercentage | Integer | Yes | Current completion percentage (0–100). |
| statusNote | String | No | Progress update provided by the assigned member. |

## Validation Rules

- Assignment must exist.
- Member must be assigned to the activity.
- Assignment must be in the **IN_PROGRESS** state.
- Completion percentage must be between **0** and **100**.
- Progress cannot be updated after the assignment has been submitted for review.
- Status note is optional but cannot exceed the maximum supported length.

## Successful Response

**HTTP 200 OK**

```json
{
  "assignmentId": "1f23d4c5-6a78-4e8a-b9c1-2d3456789012",
  "completionPercentage": 60,
  "lastUpdated": "2026-09-15T10:42:18Z",
  "message": "Progress updated successfully."
}
```

## Business Rules

- Members may update progress only for their own assignments.
- Progress updates do not change the assignment status.
- Multiple progress updates are permitted while the assignment remains **IN_PROGRESS**.
- The latest progress update replaces the previous completion percentage.
- Every progress update is automatically recorded in the Assignment History.
- Every successful progress update is automatically recorded in the Audit Log.

## Possible Errors

| HTTP Status | Description |
|-------------|-------------|
| 400 | Invalid request body. |
| 401 | Authentication required. |
| 403 | Permission denied. |
| 404 | Assignment not found. |
| 409 | Assignment cannot receive progress updates in its current state. |
| 422 | Validation failed. |
| 500 | Internal server error. |

# Get Activity Progress

Returns the latest progress information for an activity assignment.

## Endpoint

`GET /assignments/{assignmentId}/progress`

## Authentication

**Required**

Member access only.

## Path Parameters

| Parameter | Type | Description |
|----------|------|-------------|
| assignmentId | UUID | Unique Assignment identifier. |

## Successful Response

**HTTP 200 OK**

```json
{
  "assignmentId": "1f23d4c5-6a78-4e8a-b9c1-2d3456789012",
  "status": "IN_PROGRESS",
  "completionPercentage": 60,
  "statusNote": "Completed registration desk setup and volunteer briefing.",
  "lastUpdated": "2026-09-15T10:42:18Z",
  "updatedBy": {
    "userId": "3b9d4e8f-86d8-4b52-9d6d-0b1ef61d5d18",
    "name": "John Doe"
  }
}
```

## Business Rules

- Members may retrieve progress only for their own assignments.
- Administrators may retrieve progress for any assignment.
- The latest progress information is always returned.
- Progress history is maintained separately in the Assignment History.
- Progress remains accessible after submission for review.
- Progress retrieval does not modify assignment state.

## Possible Errors

| HTTP Status | Description |
|-------------|-------------|
| 401 | Authentication required. |
| 403 | Permission denied. |
| 404 | Assignment not found. |
| 500 | Internal server error. |

# Submit Activity

Submits an activity assignment for administrator review.

Only assignments in the **IN_PROGRESS** state may be submitted.

## Endpoint

`POST /assignments/{assignmentId}/submit`

## Authentication

**Required**

Member access only.

## Path Parameters

| Parameter | Type | Description |
|----------|------|-------------|
| assignmentId | UUID | Unique Assignment identifier. |

## Request Body

```json
{
  "submissionComment": "All assigned tasks have been completed successfully."
}
```

## Request Fields

| Field | Type | Required | Description |
|--------|------|----------|-------------|
| submissionComment | String | No | Optional comments provided by the member during submission. |

## Validation Rules

- Assignment must exist.
- Member must be assigned to the activity.
- Assignment must be in the **IN_PROGRESS** state.
- All mandatory evidence must be uploaded before submission.
- Submission comment is optional but cannot exceed the maximum supported length.
- An assignment cannot be submitted more than once unless it has been returned with **NEEDS_CHANGES**.

## Successful Response

**HTTP 200 OK**

```json
{
  "assignmentId": "1f23d4c5-6a78-4e8a-b9c1-2d3456789012",
  "status": "UNDER_REVIEW",
  "submittedAt": "2026-09-15T12:08:42Z",
  "message": "Assignment submitted successfully."
}
```

## Business Rules

- Members may submit only their own assignments.
- Submitting an assignment changes its status from **IN_PROGRESS** to **UNDER_REVIEW**.
- Progress updates are no longer permitted after submission.
- Evidence uploads are no longer permitted after submission.
- The submitted assignment becomes read-only for the member while under review.
- If the administrator requests changes, the assignment returns to the **NEEDS_CHANGES** state.
- If the administrator verifies the submission, the assignment moves to the **VERIFIED** state.
- Assignment submission automatically creates an item in the administrator review queue.
- Every successful submission is automatically recorded in the Assignment History.
- Every successful submission is automatically recorded in the Audit Log.

## Possible Errors

| HTTP Status | Description |
|-------------|-------------|
| 400 | Invalid request body. |
| 401 | Authentication required. |
| 403 | Permission denied. |
| 404 | Assignment not found. |
| 409 | Assignment cannot be submitted in its current state. |
| 422 | Validation failed. |
| 500 | Internal server error. |

# Upload Activity Evidence

Uploads evidence for an activity assignment.

Evidence may include photographs and videos that demonstrate completion of the assigned work.

Only assignments in the **IN_PROGRESS** or **NEEDS_CHANGES** state may receive new evidence.

## Endpoint

`POST /progress/{progressId}/evidence`

## Authentication

**Required**

Member access only.

## Request Body

**Multipart/Form-Data**

| Field | Type | Required | Description |
|--------|------|----------|-------------|
| files | File[] | Yes | One or more image or video files. |
| description | String | No | Optional description for the uploaded evidence. |

## Validation Rules

- Progress record must exist.
- Member must own the assignment.
- Assignment must be in the **IN_PROGRESS** or **NEEDS_CHANGES** state.
- Only supported image and video formats are accepted.
- Maximum of **10 photographs** per assignment.
- Maximum of **2 videos** per assignment.
- Each uploaded video must not exceed **60 seconds**.
- Uploads exceeding the configured limits are rejected.

## Successful Response

**HTTP 201 Created**

```json
{
  "uploadedFiles": [
    {
      "evidenceId": "8c5df86d-1b32-4c72-bb7c-3a82d7a5f1c9",
      "type": "PHOTO",
      "fileName": "cleanup_01.jpg"
    },
    {
      "evidenceId": "a51e5a84-0f17-4dc0-bcc0-18d50fceef8d",
      "type": "VIDEO",
      "fileName": "cleanup_video.mp4"
    }
  ],
  "message": "Evidence uploaded successfully."
}
```

## Business Rules

- Members may upload evidence only for their own assignments.
- Evidence uploads are permitted only while the assignment is **IN_PROGRESS** or **NEEDS_CHANGES**.
- Uploaded evidence becomes immediately available for administrator review.
- Uploaded evidence cannot be modified.
- Evidence may be deleted only before assignment submission.
- Every successful upload is automatically recorded in the Assignment History.
- Every successful upload is automatically recorded in the Audit Log.

## Possible Errors

| HTTP Status | Description |
|-------------|-------------|
| 400 | Invalid request body. |
| 401 | Authentication required. |
| 403 | Permission denied. |
| 404 | Progress record not found. |
| 409 | Assignment cannot accept evidence in its current state. |
| 413 | Uploaded file exceeds the permitted size. |
| 415 | Unsupported file format. |
| 422 | Validation failed. |
| 500 | Internal server error. |

# Get Activity Evidence

Returns all evidence uploaded for an activity assignment.

## Endpoint

`GET /progress/{progressId}/evidence`

## Authentication

**Required**

Member access only.

Administrators may retrieve evidence for any assignment.

## Path Parameters

| Parameter | Type | Description |
|----------|------|-------------|
| progressId | UUID | Unique Progress identifier. |

## Successful Response

**HTTP 200 OK**

```json
{
  "progressId": "5d87b81c-f5d0-40aa-8d77-49f8d0d4bdb3",
  "assignmentId": "1f23d4c5-6a78-4e8a-b9c1-2d3456789012",
  "evidence": [
    {
      "evidenceId": "8c5df86d-1b32-4c72-bb7c-3a82d7a5f1c9",
      "type": "PHOTO",
      "fileName": "cleanup_01.jpg",
      "fileUrl": "/media/evidence/cleanup_01.jpg",
      "description": "Volunteer team cleaning the auditorium.",
      "uploadedAt": "2026-09-15T10:25:18Z"
    },
    {
      "evidenceId": "a51e5a84-0f17-4dc0-bcc0-18d50fceef8d",
      "type": "VIDEO",
      "fileName": "cleanup_video.mp4",
      "fileUrl": "/media/evidence/cleanup_video.mp4",
      "description": "Final inspection of the completed work.",
      "uploadedAt": "2026-09-15T11:08:52Z"
    }
  ]
}
```

## Business Rules

- Members may retrieve evidence only for their own assignments.
- Administrators may retrieve evidence for any assignment.
- Evidence is returned in chronological order of upload.
- Uploaded evidence remains available after assignment submission.
- Historical evidence remains accessible after review completion.
- Retrieving evidence does not modify assignment state.

## Possible Errors

| HTTP Status | Description |
|-------------|-------------|
| 401 | Authentication required. |
| 403 | Permission denied. |
| 404 | Progress record not found. |
| 500 | Internal server error. |

# Delete Activity Evidence

Deletes an uploaded evidence file before the assignment is submitted for review.

Evidence cannot be deleted after the assignment has been submitted.

## Endpoint

`DELETE /progress/{progressId}/evidence/{evidenceId}`

## Authentication

**Required**

Member access only.

Administrators may delete evidence only for moderation purposes.

## Path Parameters

| Parameter | Type | Description |
|----------|------|-------------|
| progressId | UUID | Unique Progress identifier. |
| evidenceId | UUID | Unique Evidence identifier. |

## Successful Response

**HTTP 200 OK**

```json
{
  "evidenceId": "8c5df86d-1b32-4c72-bb7c-3a82d7a5f1c9",
  "message": "Evidence deleted successfully."
}
```

## Business Rules

- Members may delete only their own evidence.
- Evidence may only be deleted while the assignment is in the **IN_PROGRESS** or **NEEDS_CHANGES** state.
- Evidence cannot be deleted after the assignment has been submitted for review.
- Deleting evidence updates the assignment's evidence count.
- If deleting evidence causes the assignment to fall below the minimum required evidence, the assignment cannot be submitted until the requirement is satisfied.
- Deleted evidence is permanently removed from member access.
- Evidence deletion is automatically recorded in the Assignment History.
- Evidence deletion is automatically recorded in the Audit Log.

## Possible Errors

| HTTP Status | Description |
|-------------|-------------|
| 401 | Authentication required. |
| 403 | Permission denied. |
| 404 | Progress record or evidence not found. |
| 409 | Evidence cannot be deleted in the current assignment state. |
| 500 | Internal server error. |

# Get Pending Reviews

Returns all activity submissions that are awaiting administrator review.

Only assignments in the **UNDER_REVIEW** state are returned.

## Endpoint

`GET /reviews/pending`

## Authentication

**Required**

Administrator access only.

## Query Parameters

| Parameter | Type | Required | Description |
|----------|------|----------|-------------|
| page | Integer | No | Page number. Default: `1`. |
| pageSize | Integer | No | Number of records per page. Default: `20`. |
| activityId | UUID | No | Filter by activity. |
| memberId | UUID | No | Filter by assigned member. |
| category | String | No | Filter by activity category. |
| submittedAfter | ISO-8601 Datetime | No | Return submissions created on or after this date. |
| submittedBefore | ISO-8601 Datetime | No | Return submissions created on or before this date. |
| search | String | No | Search by activity title or member name. |
| sortBy | String | No | Sort field. Default: `submittedAt`. |
| sortOrder | String | No | `asc` or `desc`. Default: `asc`. |

## Successful Response

**HTTP 200 OK**

```json
{
  "page": 1,
  "pageSize": 20,
  "totalRecords": 12,
  "totalPages": 1,
  "reviews": [
    {
      "reviewId": "9d6f2a34-64b7-4e91-8af2-f2dcbeac8d18",
      "assignmentId": "1f23d4c5-6a78-4e8a-b9c1-2d3456789012",
      "activityTitle": "Campus Clean-Up Drive",
      "member": {
        "userId": "3b9d4e8f-86d8-4b52-9d6d-0b1ef61d5d18",
        "name": "John Doe"
      },
      "submittedAt": "2026-09-15T12:08:42Z",
      "status": "UNDER_REVIEW"
    }
  ]
}
```

## Business Rules

- Only administrators may access pending reviews.
- Only assignments in the **UNDER_REVIEW** state are returned.
- Results are returned using pagination.
- Multiple filters may be combined.
- Search performs a partial match on supported fields.
- Reviews are returned in ascending order of submission time by default so the oldest pending reviews are processed first.

## Possible Errors

| HTTP Status | Description |
|-------------|-------------|
| 401 | Authentication required. |
| 403 | Administrator permission required. |
| 422 | Invalid query parameters. |
| 500 | Internal server error. |

# Get Review Details

Returns the complete details of a submitted activity for administrator review.

## Endpoint

`GET /reviews/{reviewId}`

## Authentication

**Required**

Administrator access only.

## Path Parameters

| Parameter | Type | Description |
|----------|------|-------------|
| reviewId | UUID | Unique Review identifier. |

## Successful Response

**HTTP 200 OK**

```json
{
  "reviewId": "9d6f2a34-64b7-4e91-8af2-f2dcbeac8d18",
  "assignmentId": "1f23d4c5-6a78-4e8a-b9c1-2d3456789012",
  "status": "UNDER_REVIEW",
  "submittedAt": "2026-09-15T12:08:42Z",
  "submissionComment": "All assigned tasks have been completed successfully.",
  "activity": {
    "activityId": "a8b2d0c4-76d9-4db7-a58f-3d2dbf6b96c8",
    "title": "Campus Clean-Up Drive",
    "category": "Community Service",
    "location": "Main Campus"
  },
  "member": {
    "userId": "3b9d4e8f-86d8-4b52-9d6d-0b1ef61d5d18",
    "name": "John Doe",
    "email": "john@example.com"
  },
  "progress": {
    "completionPercentage": 100,
    "lastUpdated": "2026-09-15T11:58:26Z"
  },
  "evidence": [
    {
      "evidenceId": "8c5df86d-1b32-4c72-bb7c-3a82d7a5f1c9",
      "type": "PHOTO",
      "fileName": "cleanup_01.jpg",
      "fileUrl": "/media/evidence/cleanup_01.jpg"
    },
    {
      "evidenceId": "a51e5a84-0f17-4dc0-bcc0-18d50fceef8d",
      "type": "VIDEO",
      "fileName": "cleanup_video.mp4",
      "fileUrl": "/media/evidence/cleanup_video.mp4"
    }
  ]
}
```

## Business Rules

- Only administrators may retrieve review details.
- Review details include the submitted activity, member information, progress summary, and uploaded evidence.
- Review details remain available after the review has been completed.
- Retrieving review details does not modify the assignment or review state.

## Possible Errors

| HTTP Status | Description |
|-------------|-------------|
| 401 | Authentication required. |
| 403 | Administrator permission required. |
| 404 | Review not found. |
| 500 | Internal server error. |

# Verify Activity Submission

Verifies a submitted activity and marks it as successfully completed.

Only assignments in the **UNDER_REVIEW** state may be verified.

## Endpoint

`POST /reviews/{reviewId}/verify`

## Authentication

**Required**

Administrator access only.

## Path Parameters

| Parameter | Type | Description |
|----------|------|-------------|
| reviewId | UUID | Unique Review identifier. |

## Request Body

```json
{
  "reviewComment": "Excellent work. Activity completed successfully."
}
```

## Request Fields

| Field | Type | Required | Description |
|--------|------|----------|-------------|
| reviewComment | String | No | Optional verification comments for the assigned member. |

## Validation Rules

- Review must exist.
- Review must be in the **UNDER_REVIEW** state.
- Review cannot be verified more than once.
- Review comment is optional but cannot exceed the maximum supported length.

## Successful Response

**HTTP 200 OK**

```json
{
  "reviewId": "9d6f2a34-64b7-4e91-8af2-f2dcbeac8d18",
  "assignmentId": "1f23d4c5-6a78-4e8a-b9c1-2d3456789012",
  "status": "VERIFIED",
  "reviewedAt": "2026-09-15T14:25:18Z",
  "message": "Activity verified successfully."
}
```

## Business Rules

- Only administrators may verify activity submissions.
- Verifying a submission changes the assignment status from **UNDER_REVIEW** to **VERIFIED**.
- Verified assignments become read-only.
- Verified assignments cannot receive additional progress updates.
- Verified assignments cannot receive additional evidence uploads.
- Verified assignments cannot be resubmitted.
- Assigned members are notified when their submission has been verified.
- Activity verification is automatically recorded in the Assignment History.
- Activity verification is automatically recorded in the Audit Log.

## Possible Errors

| HTTP Status | Description |
|-------------|-------------|
| 400 | Invalid request body. |
| 401 | Authentication required. |
| 403 | Administrator permission required. |
| 404 | Review not found. |
| 409 | Review cannot be verified in its current state. |
| 422 | Validation failed. |
| 500 | Internal server error. |

# Request Activity Changes

Requests changes to a submitted activity instead of approving it.

Only assignments in the **UNDER_REVIEW** state may be returned for revision.

## Endpoint

`POST /reviews/{reviewId}/needs-changes`

## Authentication

**Required**

Administrator access only.

## Path Parameters

| Parameter | Type | Description |
|----------|------|-------------|
| reviewId | UUID | Unique Review identifier. |

## Request Body

```json
{
  "reviewComment": "Please upload clearer photographs of the completed work and provide additional progress details."
}
```

## Request Fields

| Field | Type | Required | Description |
|--------|------|----------|-------------|
| reviewComment | String | Yes | Reason for requesting changes. |

## Validation Rules

- Review must exist.
- Review must be in the **UNDER_REVIEW** state.
- Review comment cannot be empty.
- Review comment cannot exceed the maximum supported length.
- A completed review cannot be returned for changes.

## Successful Response

**HTTP 200 OK**

```json
{
  "reviewId": "9d6f2a34-64b7-4e91-8af2-f2dcbeac8d18",
  "assignmentId": "1f23d4c5-6a78-4e8a-b9c1-2d3456789012",
  "status": "NEEDS_CHANGES",
  "reviewedAt": "2026-09-15T14:32:47Z",
  "message": "Changes requested successfully."
}
```

## Business Rules

- Only administrators may request changes.
- Requesting changes changes the assignment status from **UNDER_REVIEW** to **NEEDS_CHANGES**.
- A review comment is mandatory.
- Members may update progress after changes are requested.
- Members may upload additional evidence after changes are requested.
- Members may edit the assignment and submit it again after addressing the requested changes.
- Previous review comments remain part of the assignment history.
- Assigned members are notified when changes are requested.
- Requesting changes is automatically recorded in the Assignment History.
- Requesting changes is automatically recorded in the Audit Log.

## Possible Errors

| HTTP Status | Description |
|-------------|-------------|
| 400 | Invalid request body. |
| 401 | Authentication required. |
| 403 | Administrator permission required. |
| 404 | Review not found. |
| 409 | Review cannot be returned for changes in its current state. |
| 422 | Validation failed. |
| 500 | Internal server error. |

# Create Activity Template

Creates a reusable activity template that can be used to quickly generate future activities.

Templates are not activities themselves and cannot be assigned to members until they are applied to create a new activity.

## Endpoint

`POST /activity-templates`

## Authentication

**Required**

Administrator access only.

## Request Body

```json
{
  "name": "Campus Clean-Up Template",
  "description": "Reusable template for monthly campus clean-up activities.",
  "category": "Community Service",
  "location": "Main Campus",
  "estimatedDurationMinutes": 180,
  "maxParticipants": 40,
  "requiresEvidence": true,
  "minimumPhotoCount": 2,
  "maximumPhotoCount": 10,
  "minimumVideoCount": 0,
  "maximumVideoCount": 2,
  "maximumVideoDurationSeconds": 60,
  "instructions": [
    "Wear your volunteer ID.",
    "Report to the event coordinator before starting."
  ]
}
```

## Request Fields

| Field | Type | Required | Description |
|--------|------|----------|-------------|
| name | String | Yes | Template name. |
| description | String | Yes | Template description. |
| category | String | Yes | Activity category. |
| location | String | Yes | Default activity location. |
| estimatedDurationMinutes | Integer | Yes | Estimated activity duration in minutes. |
| maxParticipants | Integer | Yes | Default maximum participants. |
| requiresEvidence | Boolean | Yes | Indicates whether evidence is required. |
| minimumPhotoCount | Integer | No | Minimum required photographs. |
| maximumPhotoCount | Integer | No | Maximum allowed photographs. Maximum: 10. |
| minimumVideoCount | Integer | No | Minimum required videos. |
| maximumVideoCount | Integer | No | Maximum allowed videos. Maximum: 2. |
| maximumVideoDurationSeconds | Integer | No | Maximum duration per uploaded video. Maximum: 60 seconds. |
| instructions | Array<String> | No | Default member instructions. |

## Validation Rules

- Template name cannot be empty.
- Description cannot be empty.
- Estimated duration must be greater than zero.
- Maximum participants must be greater than zero.
- Maximum photo count cannot exceed **10**.
- Maximum video count cannot exceed **2**.
- Maximum video duration cannot exceed **60 seconds**.
- Minimum values cannot exceed their corresponding maximum values.

## Successful Response

**HTTP 201 Created**

```json
{
  "templateId": "f47e8d3a-2b72-4c94-9f3b-8d7a2b74a8d2",
  "message": "Activity template created successfully."
}
```

## Business Rules

- Only administrators may create activity templates.
- Templates serve as reusable blueprints for future activities.
- Creating a template does not create an activity.
- Templates may be updated or archived later.
- Template creation is automatically recorded in the Audit Log.

## Possible Errors

| HTTP Status | Description |
|-------------|-------------|
| 400 | Invalid request body. |
| 401 | Authentication required. |
| 403 | Administrator permission required. |
| 409 | Template name already exists. |
| 422 | Validation failed. |
| 500 | Internal server error. |

# List Activity Templates

Returns a paginated list of all activity templates.

Templates can be filtered, searched, and sorted to help administrators quickly locate reusable activity configurations.

## Endpoint

`GET /activity-templates`

## Authentication

**Required**

Administrator access only.

## Query Parameters

| Parameter | Type | Required | Description |
|----------|------|----------|-------------|
| page | Integer | No | Page number. Default: `1`. |
| pageSize | Integer | No | Number of records per page. Default: `20`. |
| category | String | No | Filter by activity category. |
| search | String | No | Search by template name or description. |
| archived | Boolean | No | Include archived templates. Default: `false`. |
| sortBy | String | No | Sort field. Default: `name`. |
| sortOrder | String | No | `asc` or `desc`. Default: `asc`. |

## Successful Response

**HTTP 200 OK**

```json
{
  "page": 1,
  "pageSize": 20,
  "totalRecords": 12,
  "totalPages": 1,
  "templates": [
    {
      "templateId": "f47e8d3a-2b72-4c94-9f3b-8d7a2b74a8d2",
      "name": "Campus Clean-Up Template",
      "category": "Community Service",
      "estimatedDurationMinutes": 180,
      "maxParticipants": 40,
      "requiresEvidence": true,
      "isArchived": false,
      "createdAt": "2026-09-01T10:15:30Z"
    }
  ]
}
```

## Business Rules

- Only administrators may retrieve activity templates.
- Templates are returned using pagination.
- Multiple filters may be combined.
- Search performs a partial match on the template name and description.
- Archived templates are excluded unless explicitly requested.
- Templates are reusable and remain independent of activities created from them.

## Possible Errors

| HTTP Status | Description |
|-------------|-------------|
| 401 | Authentication required. |
| 403 | Administrator permission required. |
| 422 | Invalid query parameters. |
| 500 | Internal server error. |

# Get Activity Template Details

Returns the complete details of a specific activity template.

## Endpoint

`GET /activity-templates/{templateId}`

## Authentication

**Required**

Administrator access only.

## Path Parameters

| Parameter | Type | Description |
|----------|------|-------------|
| templateId | UUID | Unique Activity Template identifier. |

## Successful Response

**HTTP 200 OK**

```json
{
  "templateId": "f47e8d3a-2b72-4c94-9f3b-8d7a2b74a8d2",
  "name": "Campus Clean-Up Template",
  "description": "Reusable template for monthly campus clean-up activities.",
  "category": "Community Service",
  "location": "Main Campus",
  "estimatedDurationMinutes": 180,
  "maxParticipants": 40,
  "requiresEvidence": true,
  "minimumPhotoCount": 2,
  "maximumPhotoCount": 10,
  "minimumVideoCount": 0,
  "maximumVideoCount": 2,
  "maximumVideoDurationSeconds": 60,
  "instructions": [
    "Wear your volunteer ID.",
    "Report to the event coordinator before starting."
  ],
  "isArchived": false,
  "createdAt": "2026-09-01T10:15:30Z",
  "updatedAt": "2026-09-05T14:20:12Z",
  "createdBy": {
    "userId": "2e4e2b60-88b0-4b07-b8b8-1d8d6d76d2c9",
    "name": "John Doe"
  }
}
```

## Business Rules

- Only administrators may retrieve activity template details.
- Templates are read-only through this endpoint.
- Templates remain available even after being archived.
- Activities created from a template remain independent of subsequent template changes.
- Retrieving template details does not modify the template.

## Possible Errors

| HTTP Status | Description |
|-------------|-------------|
| 401 | Authentication required. |
| 403 | Administrator permission required. |
| 404 | Activity template not found. |
| 500 | Internal server error. |

# Update Activity Template

Updates an existing activity template.

Updating a template affects only future activities created from the template. Existing activities remain unchanged.

## Endpoint

`PATCH /activity-templates/{templateId}`

## Authentication

**Required**

Administrator access only.

## Path Parameters

| Parameter | Type | Description |
|----------|------|-------------|
| templateId | UUID | Unique Activity Template identifier. |

## Request Body

```json
{
  "name": "Campus Clean-Up Template",
  "description": "Updated reusable template for monthly campus clean-up activities.",
  "category": "Community Service",
  "location": "Main Campus",
  "estimatedDurationMinutes": 210,
  "maxParticipants": 50,
  "requiresEvidence": true,
  "minimumPhotoCount": 2,
  "maximumPhotoCount": 10,
  "minimumVideoCount": 0,
  "maximumVideoCount": 2,
  "maximumVideoDurationSeconds": 60,
  "instructions": [
    "Arrive 15 minutes before the scheduled start time.",
    "Wear your volunteer ID."
  ]
}
```

## Request Fields

| Field | Type | Required | Description |
|--------|------|----------|-------------|
| name | String | Yes | Template name. |
| description | String | Yes | Template description. |
| category | String | Yes | Activity category. |
| location | String | Yes | Default activity location. |
| estimatedDurationMinutes | Integer | Yes | Estimated activity duration in minutes. |
| maxParticipants | Integer | Yes | Default maximum participants. |
| requiresEvidence | Boolean | Yes | Indicates whether evidence is required. |
| minimumPhotoCount | Integer | No | Minimum required photographs. |
| maximumPhotoCount | Integer | No | Maximum allowed photographs. Maximum: 10. |
| minimumVideoCount | Integer | No | Minimum required videos. |
| maximumVideoCount | Integer | No | Maximum allowed videos. Maximum: 2. |
| maximumVideoDurationSeconds | Integer | No | Maximum duration per uploaded video. Maximum: 60 seconds. |
| instructions | Array<String> | No | Default member instructions. |

## Validation Rules

- Activity template must exist.
- Archived templates cannot be updated.
- Template name cannot be empty.
- Description cannot be empty.
- Estimated duration must be greater than zero.
- Maximum participants must be greater than zero.
- Maximum photo count cannot exceed **10**.
- Maximum video count cannot exceed **2**.
- Maximum video duration cannot exceed **60 seconds**.
- Minimum values cannot exceed their corresponding maximum values.

## Successful Response

**HTTP 200 OK**

```json
{
  "templateId": "f47e8d3a-2b72-4c94-9f3b-8d7a2b74a8d2",
  "message": "Activity template updated successfully."
}
```

## Business Rules

- Only administrators may update activity templates.
- Archived templates cannot be modified.
- Updating a template does not affect activities previously created from that template.
- Updated template values are used only for future activities created from the template.
- Template updates are automatically recorded in the Audit Log.

## Possible Errors

| HTTP Status | Description |
|-------------|-------------|
| 400 | Invalid request body. |
| 401 | Authentication required. |
| 403 | Administrator permission required. |
| 404 | Activity template not found. |
| 409 | Activity template cannot be updated in its current state. |
| 422 | Validation failed. |
| 500 | Internal server error. |

# Apply Activity Template

Creates a new activity using an existing activity template.

Applying a template copies the template configuration into a new independent activity. Future changes to the template do not affect the created activity.

## Endpoint

`POST /activity-templates/{templateId}/apply`

## Authentication

**Required**

Administrator access only.

## Path Parameters

| Parameter | Type | Description |
|----------|------|-------------|
| templateId | UUID | Unique Activity Template identifier. |

## Request Body

```json
{
  "title": "Campus Clean-Up Drive - September 2026",
  "location": "North Campus",
  "startTime": "2026-09-15T09:00:00Z",
  "endTime": "2026-09-15T12:00:00Z",
  "maxParticipants": 45
}
```

## Request Fields

| Field | Type | Required | Description |
|--------|------|----------|-------------|
| title | String | Yes | Title for the new activity. |
| location | String | No | Overrides the template location. |
| startTime | ISO-8601 Datetime | Yes | Activity start date and time. |
| endTime | ISO-8601 Datetime | Yes | Activity end date and time. |
| maxParticipants | Integer | No | Overrides the template participant limit. |

## Validation Rules

- Activity template must exist.
- Archived templates cannot be applied.
- Title cannot be empty.
- Start time must be earlier than end time.
- Maximum participants must be greater than zero.

## Successful Response

**HTTP 201 Created**

```json
{
  "activityId": "a8b2d0c4-76d9-4db7-a58f-3d2dbf6b96c8",
  "templateId": "f47e8d3a-2b72-4c94-9f3b-8d7a2b74a8d2",
  "status": "DRAFT",
  "message": "Activity created from template successfully."
}
```

## Business Rules

- Only administrators may apply activity templates.
- Applying a template creates a new activity in the **DRAFT** state.
- The newly created activity is independent of the template.
- Subsequent changes to the template do not affect previously created activities.
- Administrators may modify the new activity before publishing.
- Applying a template is automatically recorded in the Audit Log.

## Possible Errors

| HTTP Status | Description |
|-------------|-------------|
| 400 | Invalid request body. |
| 401 | Authentication required. |
| 403 | Administrator permission required. |
| 404 | Activity template not found. |
| 409 | Activity template cannot be applied in its current state. |
| 422 | Validation failed. |
| 500 | Internal server error. |

# Archive Activity Template

Archives an activity template.

Archived templates remain available for historical reference but cannot be modified or used to create new activities.

## Endpoint

`POST /activity-templates/{templateId}/archive`

## Authentication

**Required**

Administrator access only.

## Path Parameters

| Parameter | Type | Description |
|----------|------|-------------|
| templateId | UUID | Unique Activity Template identifier. |

## Successful Response

**HTTP 200 OK**

```json
{
  "templateId": "f47e8d3a-2b72-4c94-9f3b-8d7a2b74a8d2",
  "isArchived": true,
  "message": "Activity template archived successfully."
}
```

## Business Rules

- Only administrators may archive activity templates.
- Archived templates become read-only.
- Archived templates cannot be updated.
- Archived templates cannot be applied to create new activities.
- Activities previously created from the archived template remain unaffected.
- Archived templates remain searchable and available for historical reference.
- Template archival is automatically recorded in the Audit Log.

## Possible Errors

| HTTP Status | Description |
|-------------|-------------|
| 401 | Authentication required. |
| 403 | Administrator permission required. |
| 404 | Activity template not found. |
| 409 | Activity template is already archived. |
| 500 | Internal server error. |

# Generate Activity Report

Generates a report for activities based on the specified filters.

Report generation may take time depending on the requested data. Reports are generated asynchronously.

## Endpoint

`POST /activity-reports`

## Authentication

**Required**

Administrator access only.

## Request Body

```json
{
  "reportType": "SUMMARY",
  "activityIds": [
    "a8b2d0c4-76d9-4db7-a58f-3d2dbf6b96c8"
  ],
  "category": "Community Service",
  "status": "ACTIVE",
  "startDate": "2026-09-01T00:00:00Z",
  "endDate": "2026-09-30T23:59:59Z",
  "format": "EXCEL"
}
```

## Request Fields

| Field | Type | Required | Description |
|--------|------|----------|-------------|
| reportType | String | Yes | Type of report to generate. |
| activityIds | Array<UUID> | No | Specific activities to include. |
| category | String | No | Filter by activity category. |
| status | String | No | Filter by activity status. |
| startDate | ISO-8601 Datetime | No | Report start date. |
| endDate | ISO-8601 Datetime | No | Report end date. |
| format | String | Yes | Export format (`EXCEL`, `CSV`, `PDF`). |

## Validation Rules

- Report type must be supported.
- Export format must be supported.
- Start date must not be later than end date.
- At least one filter should be provided to limit report size.

## Successful Response

**HTTP 202 Accepted**

```json
{
  "reportId": "6b7b3b5d-3a12-4d7b-8b7c-8cb1d03dcb43",
  "status": "GENERATING",
  "message": "Report generation started successfully."
}
```

## Business Rules

- Only administrators may generate reports.
- Report generation executes asynchronously.
- The generated report becomes available after processing completes.
- Generated reports remain available until removed by the backend retention policy.
- Report generation is automatically recorded in the Audit Log.

## Possible Errors

| HTTP Status | Description |
|-------------|-------------|
| 400 | Invalid request body. |
| 401 | Authentication required. |
| 403 | Administrator permission required. |
| 422 | Validation failed. |
| 500 | Internal server error. |

# Get Activity Report

Returns the status and download information for a previously generated activity report.

## Endpoint

`GET /activity-reports/{reportId}`

## Authentication

**Required**

Administrator access only.

## Path Parameters

| Parameter | Type | Description |
|----------|------|-------------|
| reportId | UUID | Unique Activity Report identifier. |

## Successful Response

**HTTP 200 OK**

```json
{
  "reportId": "6b7b3b5d-3a12-4d7b-8b7c-8cb1d03dcb43",
  "reportType": "SUMMARY",
  "status": "COMPLETED",
  "format": "EXCEL",
  "generatedAt": "2026-09-30T17:42:18Z",
  "generatedBy": {
    "userId": "2e4e2b60-88b0-4b07-b8b8-1d8d6d76d2c9",
    "name": "John Doe"
  },
  "expiresAt": "2026-10-07T17:42:18Z",
  "downloadUrl": "/reports/6b7b3b5d-3a12-4d7b-8b7c-8cb1d03dcb43.xlsx"
}
```

## Business Rules

- Only administrators may retrieve activity reports.
- Reports remain in the **GENERATING** state until processing is complete.
- A download URL is available only after the report status becomes **COMPLETED**.
- Reports that fail during generation return the **FAILED** status.
- Expired reports are removed according to the backend retention policy.
- Retrieving a report does not regenerate the report.
- Report access is automatically recorded in the Audit Log.

## Possible Errors

| HTTP Status | Description |
|-------------|-------------|
| 401 | Authentication required. |
| 403 | Administrator permission required. |
| 404 | Activity report not found. |
| 500 | Internal server error. |

# Get Activity Report Summary

Returns a summarized overview of activity statistics for the specified filters.

This endpoint provides a high-level dashboard view and is intended for analytics rather than detailed reporting.

## Endpoint

`GET /activity-reports/summary`

## Authentication

**Required**

Administrator access only.

## Query Parameters

| Parameter | Type | Required | Description |
|----------|------|----------|-------------|
| startDate | ISO-8601 Datetime | No | Summary start date. |
| endDate | ISO-8601 Datetime | No | Summary end date. |
| category | String | No | Filter by activity category. |
| status | String | No | Filter by activity status. |

## Successful Response

**HTTP 200 OK**

```json
{
  "totalActivities": 42,
  "draftActivities": 4,
  "activeActivities": 18,
  "cancelledActivities": 3,
  "archivedActivities": 17,

  "totalAssignments": 356,
  "completedAssignments": 298,
  "pendingAssignments": 34,
  "underReviewAssignments": 12,
  "needsChangesAssignments": 8,
  "cancelledAssignments": 4,

  "completionRate": 83.71,

  "totalEvidence": {
    "photos": 894,
    "videos": 96
  },

  "generatedAt": "2026-09-30T18:20:41Z"
}
```

## Business Rules

- Only administrators may retrieve activity summaries.
- Summary statistics are calculated using the supplied filters.
- If no filters are provided, the summary includes all accessible activities.
- Summary data is generated in real time.
- This endpoint returns aggregated statistics only.
- Individual activity or member details are not included.
- Summary retrieval is automatically recorded in the Audit Log.

## Possible Errors

| HTTP Status | Description |
|-------------|-------------|
| 401 | Authentication required. |
| 403 | Administrator permission required. |
| 422 | Invalid query parameters. |
| 500 | Internal server error. |

# Get My Activity History

Returns the authenticated member's complete activity history.

The activity history includes completed, verified, cancelled, and archived assignments along with their final outcomes.

## Endpoint

`GET /members/me/activity-history`

## Authentication

**Required**

Member access only.

## Query Parameters

| Parameter | Type | Required | Description |
|----------|------|----------|-------------|
| page | Integer | No | Page number. Default: `1`. |
| pageSize | Integer | No | Number of records per page. Default: `20`. |
| status | String | No | Filter by assignment status. |
| category | String | No | Filter by activity category. |
| startDate | ISO-8601 Datetime | No | Return activities completed on or after this date. |
| endDate | ISO-8601 Datetime | No | Return activities completed on or before this date. |
| search | String | No | Search by activity title. |
| sortBy | String | No | Sort field. Default: `completedAt`. |
| sortOrder | String | No | `asc` or `desc`. Default: `desc`. |

## Successful Response

**HTTP 200 OK**

```json
{
  "page": 1,
  "pageSize": 20,
  "totalRecords": 24,
  "totalPages": 2,
  "activities": [
    {
      "assignmentId": "1f23d4c5-6a78-4e8a-b9c1-2d3456789012",
      "activityId": "a8b2d0c4-76d9-4db7-a58f-3d2dbf6b96c8",
      "title": "Campus Clean-Up Drive",
      "category": "Community Service",
      "status": "VERIFIED",
      "completedAt": "2026-09-15T14:25:18Z",
      "reviewedAt": "2026-09-15T15:02:41Z"
    }
  ]
}
```

## Business Rules

- Members may retrieve only their own activity history.
- Activity history is returned using pagination.
- Multiple filters may be combined.
- Search performs a partial match on activity title.
- Historical records are immutable.
- Activity history remains available even if the original activity has been archived.
- Retrieving activity history does not modify any application state.

## Possible Errors

| HTTP Status | Description |
|-------------|-------------|
| 401 | Authentication required. |
| 403 | Member permission required. |
| 422 | Invalid query parameters. |
| 500 | Internal server error. |

# Get Activity History

Returns the complete history of an activity, including lifecycle events, assignments, progress updates, evidence submissions, review actions, and status changes.

This endpoint provides administrators with a chronological timeline of the activity from creation to its current state.

## Endpoint

`GET /activities/{activityId}/history`

## Authentication

**Required**

Administrator access only.

## Path Parameters

| Parameter | Type | Description |
|----------|------|-------------|
| activityId | UUID | Unique Activity identifier. |

## Query Parameters

| Parameter | Type | Required | Description |
|----------|------|----------|-------------|
| page | Integer | No | Page number. Default: `1`. |
| pageSize | Integer | No | Number of records per page. Default: `20`. |
| eventType | String | No | Filter by history event type. |
| startDate | ISO-8601 Datetime | No | Return events occurring on or after this date. |
| endDate | ISO-8601 Datetime | No | Return events occurring on or before this date. |
| sortOrder | String | No | `asc` or `desc`. Default: `asc`. |

## Successful Response

**HTTP 200 OK**

```json
{
  "activityId": "a8b2d0c4-76d9-4db7-a58f-3d2dbf6b96c8",
  "page": 1,
  "pageSize": 20,
  "totalRecords": 9,
  "totalPages": 1,
  "history": [
    {
      "eventId": "b3c57bc8-84b9-4db2-a8b6-9bc8b0dc9c1f",
      "eventType": "ACTIVITY_CREATED",
      "performedBy": {
        "userId": "2e4e2b60-88b0-4b07-b8b8-1d8d6d76d2c9",
        "name": "John Doe"
      },
      "timestamp": "2026-09-01T09:15:42Z",
      "details": "Activity created."
    },
    {
      "eventId": "9c6d65f0-7e51-49b2-b77d-4d2dc2bca1d1",
      "eventType": "ACTIVITY_PUBLISHED",
      "performedBy": {
        "userId": "2e4e2b60-88b0-4b07-b8b8-1d8d6d76d2c9",
        "name": "John Doe"
      },
      "timestamp": "2026-09-02T10:30:18Z",
      "details": "Activity published."
    }
  ]
}
```

## Business Rules

- Only administrators may retrieve activity history.
- Activity history is returned in chronological order by default.
- Pagination is applied to all history records.
- Multiple filters may be combined.
- Activity history is immutable.
- Historical records remain available after an activity has been cancelled or archived.
- Activity history includes lifecycle events, assignment events, progress updates, evidence events, review actions, and administrative actions related to the activity.
- Retrieving activity history does not modify any application state.

## Possible Errors

| HTTP Status | Description |
|-------------|-------------|
| 401 | Authentication required. |
| 403 | Administrator permission required. |
| 404 | Activity not found. |
| 422 | Invalid query parameters. |
| 500 | Internal server error. |

Phase 4 

# Get Audit Logs

Returns a paginated list of audit logs generated by the system.

Audit logs provide a read-only history of important system events for operational monitoring, troubleshooting, compliance, and security investigations.

## Endpoint

`GET /audit-logs`

## Authentication

**Required**

Administrator access only.

## Query Parameters

| Parameter | Type | Required | Description |
|----------|------|----------|-------------|
| page | Integer | No | Page number. Default: `1`. |
| pageSize | Integer | No | Number of records per page. Default: `20`. |
| category | String | No | Filter by audit category. |
| module | String | No | Filter by application module. |
| action | String | No | Filter by performed action. |
| outcome | String | No | Filter by audit outcome (`SUCCESS`, `FAILED`). |
| userId | UUID | No | Filter by user who performed the action. |
| resourceType | String | No | Filter by affected resource type. |
| resourceId | UUID | No | Filter by affected resource identifier. |
| startDate | ISO-8601 Datetime | No | Return records on or after this date. |
| endDate | ISO-8601 Datetime | No | Return records on or before this date. |
| search | String | No | Search supported audit fields. |
| sortBy | String | No | Sort field. Default: `timestamp`. |
| sortOrder | String | No | `asc` or `desc`. Default: `desc`. |

## Successful Response

**HTTP 200 OK**

```json
{
  "page": 1,
  "pageSize": 20,
  "totalRecords": 248,
  "totalPages": 13,
  "auditLogs": [
    {
      "auditLogId": "2c99f3d9-4cb4-44a7-8d35-bf3f61d7d5f0",
      "timestamp": "2026-09-30T16:42:18Z",
      "category": "Synchronization",
      "module": "Offline Queue",
      "action": "SYNC_COMPLETED",
      "outcome": "SUCCESS",
      "performedBy": {
        "userId": "2e4e2b60-88b0-4b07-b8b8-1d8d6d76d2c9",
        "name": "John Doe"
      },
      "resource": {
        "type": "Attendance Session",
        "id": "a8b2d0c4-76d9-4db7-a58f-3d2dbf6b96c8"
      }
    }
  ]
}
```

## Business Rules

- Only administrators may retrieve audit logs.
- Audit logs are returned using pagination.
- Multiple filters may be combined.
- Search performs a partial match on supported audit fields.
- Audit records are immutable.
- Audit logs are returned in descending order by timestamp by default.
- Historical audit records remain available even if the associated resource has been removed.
- Retrieving audit logs does not modify any application state.

## Possible Errors

| HTTP Status | Description |
|-------------|-------------|
| 401 | Authentication required. |
| 403 | Administrator permission required. |
| 422 | Invalid query parameters. |
| 500 | Internal server error. |

# Get Audit Log Details

Returns the complete details of a specific audit log entry.

Audit log details provide complete information about the recorded event, including the actor, affected resource, metadata, and any recorded changes.

## Endpoint

`GET /audit-logs/{auditLogId}`

## Authentication

**Required**

Administrator access only.

## Path Parameters

| Parameter | Type | Description |
|----------|------|-------------|
| auditLogId | UUID | Unique Audit Log identifier. |

## Successful Response

**HTTP 200 OK**

```json
{
  "auditLogId": "2c99f3d9-4cb4-44a7-8d35-bf3f61d7d5f0",
  "timestamp": "2026-09-30T16:42:18Z",
  "category": "Synchronization",
  "module": "Offline Queue",
  "action": "SYNC_COMPLETED",
  "outcome": "SUCCESS",
  "performedBy": {
    "userId": "2e4e2b60-88b0-4b07-b8b8-1d8d6d76d2c9",
    "name": "John Doe",
    "role": "ADMIN"
  },
  "resource": {
    "type": "Attendance Session",
    "id": "a8b2d0c4-76d9-4db7-a58f-3d2dbf6b96c8"
  },
  "changes": {
    "before": {},
    "after": {}
  },
  "metadata": {
    "syncDurationMs": 1842,
    "processedOperations": 8,
    "ipAddress": "192.168.1.100",
    "userAgent": "Chrome 139"
  }
}
```

## Business Rules

- Only administrators may retrieve audit log details.
- Audit log entries are immutable.
- Audit log details cannot be modified or deleted.
- The `changes` object is included only when the audited operation modified application data.
- The `metadata` object varies depending on the recorded event.
- Audit records remain available even if the related resource has been deleted.
- Retrieving audit log details does not modify any application state.

## Possible Errors

| HTTP Status | Description |
|-------------|-------------|
| 401 | Authentication required. |
| 403 | Administrator permission required. |
| 404 | Audit log not found. |
| 500 | Internal server error. |

# Get Security Events

Returns a paginated list of security events detected by the system.

Security events include authentication anomalies, authorization failures, suspicious activity, backup failures, synchronization failures, and other security-related incidents.

## Endpoint

`GET /security-events`

## Authentication

**Required**

Administrator access only.

## Query Parameters

| Parameter | Type | Required | Description |
|----------|------|----------|-------------|
| page | Integer | No | Page number. Default: `1`. |
| pageSize | Integer | No | Number of records per page. Default: `20`. |
| severity | String | No | Filter by severity (`LOW`, `MEDIUM`, `HIGH`, `CRITICAL`). |
| status | String | No | Filter by event status (`OPEN`, `UNDER_INVESTIGATION`, `RESOLVED`). |
| category | String | No | Filter by security category. |
| userId | UUID | No | Filter by affected user. |
| startDate | ISO-8601 Datetime | No | Return events on or after this date. |
| endDate | ISO-8601 Datetime | No | Return events on or before this date. |
| search | String | No | Search supported security event fields. |
| sortBy | String | No | Sort field. Default: `detectedAt`. |
| sortOrder | String | No | `asc` or `desc`. Default: `desc`. |

## Successful Response

**HTTP 200 OK**

```json
{
  "page": 1,
  "pageSize": 20,
  "totalRecords": 18,
  "totalPages": 1,
  "securityEvents": [
    {
      "securityEventId": "7f61d8f3-7d74-4f2d-9e7f-94d7b8d18f31",
      "category": "Authentication",
      "severity": "HIGH",
      "status": "OPEN",
      "title": "Multiple Failed Login Attempts",
      "detectedAt": "2026-09-30T18:12:24Z",
      "affectedUser": {
        "userId": "3b9d4e8f-86d8-4b52-9d6d-0b1ef61d5d18",
        "name": "John Doe"
      }
    }
  ]
}
```

## Business Rules

- Only administrators may retrieve security events.
- Security events are returned using pagination.
- Multiple filters may be combined.
- Search performs a partial match on supported fields.
- Security events are returned in descending order of detection time by default.
- Historical security events remain available after resolution.
- Retrieving security events does not modify any application state.

## Possible Errors

| HTTP Status | Description |
|-------------|-------------|
| 401 | Authentication required. |
| 403 | Administrator permission required. |
| 422 | Invalid query parameters. |
| 500 | Internal server error. |

# Get Security Event Details

Returns the complete details of a specific security event.

This endpoint provides administrators with detailed information about the detected security event, including its severity, status, affected resources, investigation history, and resolution details.

## Endpoint

`GET /security-events/{securityEventId}`

## Authentication

**Required**

Administrator access only.

## Path Parameters

| Parameter | Type | Description |
|----------|------|-------------|
| securityEventId | UUID | Unique Security Event identifier. |

## Successful Response

**HTTP 200 OK**

```json
{
  "securityEventId": "7f61d8f3-7d74-4f2d-9e7f-94d7b8d18f31",
  "category": "Authentication",
  "severity": "HIGH",
  "status": "OPEN",
  "title": "Multiple Failed Login Attempts",
  "description": "Five consecutive failed login attempts were detected within two minutes.",
  "detectedAt": "2026-09-30T18:12:24Z",
  "affectedUser": {
    "userId": "3b9d4e8f-86d8-4b52-9d6d-0b1ef61d5d18",
    "name": "John Doe",
    "email": "john@example.com"
  },
  "affectedResource": {
    "resourceType": "User Account",
    "resourceId": "3b9d4e8f-86d8-4b52-9d6d-0b1ef61d5d18"
  },
  "metadata": {
    "ipAddress": "192.168.1.120",
    "userAgent": "Chrome 139",
    "attemptCount": 5
  },
  "investigation": {
    "assignedTo": null,
    "startedAt": null
  },
  "resolution": null
}
```

## Business Rules

- Only administrators may retrieve security event details.
- Security events are immutable records.
- Investigation and resolution information are returned when available.
- Historical security events remain accessible after resolution.
- Retrieving security event details does not modify any application state.

## Possible Errors

| HTTP Status | Description |
|-------------|-------------|
| 401 | Authentication required. |
| 403 | Administrator permission required. |
| 404 | Security event not found. |
| 500 | Internal server error. |

# Resolve Security Event

Marks a security event as resolved after investigation has been completed.

Only security events in the **OPEN** or **UNDER_INVESTIGATION** state may be resolved.

## Endpoint

`POST /security-events/{securityEventId}/resolve`

## Authentication

**Required**

Administrator access only.

## Path Parameters

| Parameter | Type | Description |
|----------|------|-------------|
| securityEventId | UUID | Unique Security Event identifier. |

## Request Body

```json
{
  "resolution": "User identity verified. Account secured and password reset completed.",
  "resolutionCategory": "RESOLVED",
  "preventiveAction": "Enabled multi-factor authentication."
}
```

## Request Fields

| Field | Type | Required | Description |
|--------|------|----------|-------------|
| resolution | String | Yes | Summary of how the security event was resolved. |
| resolutionCategory | String | Yes | Resolution outcome. |
| preventiveAction | String | No | Preventive action taken to avoid recurrence. |

## Validation Rules

- Security event must exist.
- Only events in the **OPEN** or **UNDER_INVESTIGATION** state may be resolved.
- Resolution cannot be empty.
- Resolution category must be supported.
- A resolved event cannot be resolved again.

## Successful Response

**HTTP 200 OK**

```json
{
  "securityEventId": "7f61d8f3-7d74-4f2d-9e7f-94d7b8d18f31",
  "status": "RESOLVED",
  "resolvedAt": "2026-09-30T19:18:54Z",
  "message": "Security event resolved successfully."
}
```

## Business Rules

- Only administrators may resolve security events.
- Resolving an event changes its status to **RESOLVED**.
- Every resolved event must include a resolution summary.
- Resolution information becomes part of the permanent security event history.
- Resolved events remain searchable and available for auditing.
- Resolving a security event is automatically recorded in the Audit Log.

## Possible Errors

| HTTP Status | Description |
|-------------|-------------|
| 400 | Invalid request body. |
| 401 | Authentication required. |
| 403 | Administrator permission required. |
| 404 | Security event not found. |
| 409 | Security event cannot be resolved in its current state. |
| 422 | Validation failed. |
| 500 | Internal server error. |

# Create Backup

Creates a manual backup of the application data.

Manual backups may be initiated by administrators at any time. Scheduled backups are created automatically by the system.

## Endpoint

`POST /backups`

## Authentication

**Required**

Administrator access only.

## Request Body

```json
{
  "type": "MANUAL",
  "description": "Pre-release backup before Phase 4 deployment."
}
```

## Request Fields

| Field | Type | Required | Description |
|--------|------|----------|-------------|
| type | String | Yes | Backup type. Supported values: `MANUAL`. |
| description | String | No | Optional description for the backup. |

## Validation Rules

- Only manual backups may be requested through this endpoint.
- A new backup cannot be started while another backup is currently in progress.
- Description is optional but cannot exceed the maximum supported length.

## Successful Response

**HTTP 202 Accepted**

```json
{
  "backupId": "9d8b67a4-ec18-47df-a6b2-3d5b48dba2c8",
  "type": "MANUAL",
  "status": "IN_PROGRESS",
  "startedAt": "2026-09-30T20:15:42Z",
  "message": "Backup started successfully."
}
```

## Business Rules

- Only administrators may create manual backups.
- Scheduled backups are created automatically by the backend and do not use this endpoint.
- Only one backup operation may run at a time.
- Backup verification starts automatically after backup creation completes.
- Backup creation is automatically recorded in the Audit Log.
- Failed backups remain available for troubleshooting.

## Possible Errors

| HTTP Status | Description |
|-------------|-------------|
| 400 | Invalid request body. |
| 401 | Authentication required. |
| 403 | Administrator permission required. |
| 409 | Another backup operation is already in progress. |
| 422 | Validation failed. |
| 500 | Internal server error. |

# Get Backups

Returns a paginated list of all backups created by the system.

The list includes both manually created backups and automatically scheduled backups.

## Endpoint

`GET /backups`

## Authentication

**Required**

Administrator access only.

## Query Parameters

| Parameter | Type | Required | Description |
|----------|------|----------|-------------|
| page | Integer | No | Page number. Default: `1`. |
| pageSize | Integer | No | Number of records per page. Default: `20`. |
| type | String | No | Filter by backup type (`MANUAL`, `SCHEDULED`). |
| status | String | No | Filter by backup status (`IN_PROGRESS`, `COMPLETED`, `FAILED`, `VERIFIED`, `RESTORED`). |
| verified | Boolean | No | Filter by verification status. |
| startDate | ISO-8601 Datetime | No | Return backups created on or after this date. |
| endDate | ISO-8601 Datetime | No | Return backups created on or before this date. |
| search | String | No | Search by backup description. |
| sortBy | String | No | Sort field. Default: `createdAt`. |
| sortOrder | String | No | `asc` or `desc`. Default: `desc`. |

## Successful Response

**HTTP 200 OK**

```json
{
  "page": 1,
  "pageSize": 20,
  "totalRecords": 48,
  "totalPages": 3,
  "backups": [
    {
      "backupId": "9d8b67a4-ec18-47df-a6b2-3d5b48dba2c8",
      "type": "SCHEDULED",
      "status": "VERIFIED",
      "verified": true,
      "size": "185 MB",
      "createdAt": "2026-09-30T02:00:00Z",
      "completedAt": "2026-09-30T02:03:18Z",
      "description": "Nightly scheduled backup"
    },
    {
      "backupId": "63aab541-53cb-42c5-9cb8-c71dc4d7b1e4",
      "type": "MANUAL",
      "status": "COMPLETED",
      "verified": false,
      "size": "184 MB",
      "createdAt": "2026-09-30T20:15:42Z",
      "completedAt": "2026-09-30T20:18:06Z",
      "description": "Pre-release backup before Phase 4 deployment."
    }
  ]
}
```

## Business Rules

- Only administrators may retrieve backup information.
- Both manual and scheduled backups are returned.
- Results are returned using pagination.
- Multiple filters may be combined.
- Search performs a partial match on the backup description.
- Backups are returned in descending order of creation time by default.
- Historical backup records remain available even after newer backups are created.
- Retrieving backup information does not modify any application state.

## Possible Errors

| HTTP Status | Description |
|-------------|-------------|
| 401 | Authentication required. |
| 403 | Administrator permission required. |
| 422 | Invalid query parameters. |
| 500 | Internal server error. |

# Get Backup Details

Returns the complete details of a specific backup.

This endpoint provides administrators with detailed information about the backup, including its type, status, verification status, creation details, restore history, and metadata.

## Endpoint

`GET /backups/{backupId}`

## Authentication

**Required**

Administrator access only.

## Path Parameters

| Parameter | Type | Description |
|----------|------|-------------|
| backupId | UUID | Unique Backup identifier. |

## Successful Response

**HTTP 200 OK**

```json
{
  "backupId": "9d8b67a4-ec18-47df-a6b2-3d5b48dba2c8",
  "type": "SCHEDULED",
  "status": "VERIFIED",
  "verified": true,
  "description": "Nightly scheduled backup",
  "size": "185 MB",
  "createdAt": "2026-09-30T02:00:00Z",
  "completedAt": "2026-09-30T02:03:18Z",
  "verifiedAt": "2026-09-30T02:03:46Z",
  "createdBy": {
    "userId": null,
    "name": "System Scheduler"
  },
  "storage": {
    "location": "Local Storage",
    "checksum": "4cf53f2e7d5d94d8d43fcb52a82b9a7f"
  },
  "restoreHistory": {
    "restoreCount": 0,
    "lastRestoredAt": null
  }
}
```

## Business Rules

- Only administrators may retrieve backup details.
- Backup records are immutable.
- Verification information is available only after verification has completed.
- Restore history is maintained for every backup.
- Backup metadata remains available even after a backup has been restored.
- Retrieving backup details does not modify any application state.

## Possible Errors

| HTTP Status | Description |
|-------------|-------------|
| 401 | Authentication required. |
| 403 | Administrator permission required. |
| 404 | Backup not found. |
| 500 | Internal server error. |

# Restore Backup

Restores the application from a previously created backup.

Only successfully completed and verified backups may be restored.

## Endpoint

`POST /backups/{backupId}/restore`

## Authentication

**Required**

Administrator access only.

## Path Parameters

| Parameter | Type | Description |
|----------|------|-------------|
| backupId | UUID | Unique Backup identifier. |

## Request Body

```json
{
  "reason": "Recovering application after database corruption."
}
```

## Request Fields

| Field | Type | Required | Description |
|--------|------|----------|-------------|
| reason | String | Yes | Reason for initiating the restore operation. |

## Validation Rules

- Backup must exist.
- Only backups with **VERIFIED** status may be restored.
- Restore reason cannot be empty.
- Another backup or restore operation must not already be in progress.
- A backup cannot be restored if it failed verification.

## Successful Response

**HTTP 202 Accepted**

```json
{
  "backupId": "9d8b67a4-ec18-47df-a6b2-3d5b48dba2c8",
  "status": "RESTORING",
  "startedAt": "2026-09-30T21:18:42Z",
  "message": "Backup restoration started successfully."
}
```

## Business Rules

- Only administrators may restore backups.
- Only verified backups may be restored.
- Only one restore operation may run at a time.
- The application may become temporarily unavailable during restoration.
- Every restore operation is automatically recorded in the Audit Log.
- Every restore operation creates a corresponding Security Event.
- Restore history is permanently retained.

## Possible Errors

| HTTP Status | Description |
|-------------|-------------|
| 400 | Invalid request body. |
| 401 | Authentication required. |
| 403 | Administrator permission required. |
| 404 | Backup not found. |
| 409 | Backup cannot be restored in its current state or another restore is already in progress. |
| 422 | Validation failed. |
| 500 | Internal server error. |

# Verify Backup

Verifies the integrity and usability of a completed backup.

Verification confirms that the backup is valid and can be restored successfully. Both scheduled and manual backups are verified automatically after creation.

## Endpoint

`POST /backups/{backupId}/verify`

## Authentication

**Required**

Administrator access only.

## Path Parameters

| Parameter | Type | Description |
|----------|------|-------------|
| backupId | UUID | Unique Backup identifier. |

## Successful Response

**HTTP 202 Accepted**

```json
{
  "backupId": "9d8b67a4-ec18-47df-a6b2-3d5b48dba2c8",
  "status": "VERIFYING",
  "startedAt": "2026-09-30T02:03:20Z",
  "message": "Backup verification started successfully."
}
```

## Business Rules

- Only administrators may initiate backup verification manually.
- Automatic verification is performed after every successful backup, including both scheduled and manual backups.
- Only backups in the **COMPLETED** state may be verified.
- A backup cannot be verified while another verification is already in progress.
- Successfully verified backups transition to the **VERIFIED** state.
- Failed verification changes the backup status to **FAILED**.
- Verification confirms backup integrity and restore readiness.
- Every verification attempt is automatically recorded in the Audit Log.
- Failed verification automatically generates a Security Event.
- Verification history is permanently retained.

## Possible Errors

| HTTP Status | Description |
|-------------|-------------|
| 401 | Authentication required. |
| 403 | Administrator permission required. |
| 404 | Backup not found. |
| 409 | Backup cannot be verified in its current state or verification is already in progress. |
| 500 | Internal server error. |

# Synchronize Offline Data

Synchronizes all pending offline operations with the backend.

The backend processes the entire synchronization queue in a single request. Operations are processed in the order they were created to preserve data consistency.

## Endpoint

`POST /sync`

## Authentication

**Required**

Member access only.

## Request Body

```json
{
  "deviceId": "8e42f98a-9b18-4f2d-bd95-78b8a63a8d1a",
  "lastSyncAt": "2026-09-30T18:45:21Z",
  "operations": [
    {
      "operationId": "1f9d4d1d-76e8-4f76-a3d2-3d4ef8b6b8f1",
      "operationType": "CHECK_IN",
      "createdAt": "2026-09-30T18:46:12Z",
      "payload": {}
    },
    {
      "operationId": "e6b62c84-5d5e-45a9-bd56-c2f43a25d8e7",
      "operationType": "ACTIVITY_PROGRESS",
      "createdAt": "2026-09-30T18:48:41Z",
      "payload": {}
    }
  ]
}
```

## Request Fields

| Field | Type | Required | Description |
|--------|------|----------|-------------|
| deviceId | UUID | Yes | Unique device identifier. |
| lastSyncAt | ISO-8601 Datetime | No | Timestamp of the previous successful synchronization. |
| operations | Array | Yes | Queue of pending offline operations. |

## Validation Rules

- Device must be registered.
- User must be authenticated.
- Operations are processed in chronological order.
- Duplicate operations are ignored.
- Invalid operations do not stop processing of valid operations.
- The synchronization queue cannot be empty.

## Successful Response

**HTTP 200 OK**

```json
{
  "processed": 12,
  "successful": 11,
  "failed": 1,
  "lastSyncAt": "2026-09-30T19:02:41Z",
  "results": [
    {
      "operationId": "1f9d4d1d-76e8-4f76-a3d2-3d4ef8b6b8f1",
      "status": "SUCCESS"
    },
    {
      "operationId": "e6b62c84-5d5e-45a9-bd56-c2f43a25d8e7",
      "status": "FAILED",
      "reason": "Assignment already submitted."
    }
  ]
}
```

## Business Rules

- Members may synchronize only their own offline data.
- The entire synchronization queue is processed within a single synchronization session.
- Operations are processed in chronological order.
- Successfully synchronized operations are removed from the local queue.
- Failed operations remain in the local queue for automatic retry after the failure condition has been resolved.
- Duplicate operations are ignored to ensure idempotent synchronization.
- Synchronization activity is automatically recorded in the Audit Log.
- Synchronization failures automatically generate Security Events when applicable.

## Possible Errors

| HTTP Status | Description |
|-------------|-------------|
| 400 | Invalid request body. |
| 401 | Authentication required. |
| 403 | Permission denied. |
| 409 | Another synchronization is already in progress. |
| 422 | Validation failed. |
| 500 | Internal server error. |

# Get Synchronization Status

Returns the current synchronization status for the authenticated member.

This endpoint allows the client to determine whether synchronization is in progress, view the current queue status, identify failed operations, and display the latest successful synchronization time.

## Endpoint

`GET /sync/status`

## Authentication

**Required**

Member access only.

## Successful Response

**HTTP 200 OK**

```json
{
  "status": "ONLINE",
  "syncState": "IDLE",
  "lastSuccessfulSyncAt": "2026-09-30T19:02:41Z",
  "pendingOperations": 3,
  "failedOperations": 1,
  "isSynchronizationInProgress": false,
  "lastSynchronization": {
    "startedAt": "2026-09-30T19:01:58Z",
    "completedAt": "2026-09-30T19:02:41Z",
    "processedOperations": 12,
    "successfulOperations": 11,
    "failedOperations": 1
  }
}
```

## Business Rules

- Members may retrieve only their own synchronization status.
- Synchronization status reflects the current state of the local synchronization queue.
- Pending operations represent items waiting to be synchronized.
- Failed operations remain queued until a future synchronization attempt succeeds or the failure condition is resolved.
- The backend returns the result of the most recent synchronization attempt.
- Retrieving synchronization status does not trigger a synchronization.
- Retrieving synchronization status does not modify the synchronization queue.
- Synchronization status retrieval is automatically recorded in the Audit Log.

## Possible Errors

| HTTP Status | Description |
|-------------|-------------|
| 401 | Authentication required. |
| 403 | Permission denied. |
| 500 | Internal server error. |

