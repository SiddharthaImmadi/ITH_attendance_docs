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
| 422 | Business-rule rejection (e.g., outside radius, session not open, time window closed) |

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

**Purpose:** Admin creates an attendance session by defining the venue, time window, and radius.
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
  "venue_lat": 12.9716,
  "venue_lng": 77.5946,
  "radius_meters": 50
}
```

**Field notes:**
- `title` (string, required)
- `purpose` (string, optional)
- `date` (date `YYYY-MM-DD`, required)
- `start_time` (ISO 8601 datetime, required) — must be before `end_time`
- `end_time` (ISO 8601 datetime, required) — must be after `start_time`
- `grace_period_minutes` (integer, required) — minutes after `start_time` before a check-in is marked `late` instead of `present`
- `venue_lat` (double, required) — decimal degrees
- `venue_lng` (double, required) — decimal degrees
- `radius_meters` (integer, required) — must be > 0

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
  "venue_lat": 12.9716,
  "venue_lng": 77.5946,
  "radius_meters": 50,
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
    "message": "radius_meters is required and must be greater than 0."
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
  "venue_lat": 12.9716,
  "venue_lng": 77.5946,
  "radius_meters": 50,
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

**Distance check (Haversine formula):**
- Compute distance between (lat, lng) and session's (venue_lat, venue_lng)
- Compare against `radius_meters`

**Decision table:**

| Condition | Result |
|---|---|
| `distance <= radius_meters` AND `gps_accuracy_meters` is acceptable (≤ 30m) | ✅ Inside venue — eligible for `present` or `late` status |
| `distance > radius_meters + 10m tolerance` AND `gps_accuracy_meters` is acceptable | ❌ Outside venue — rejected with code `OUTSIDE_RADIUS` |
| `gps_accuracy_meters > 30m` (poor accuracy) OR reading is stale | ⚠️ Uncertain — mark as `pending_verification`, flag for admin review (don't auto-reject) |

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

**Response 201 — Accepted (inside radius, on time):**
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

**Response 201 — Late (inside radius, but after grace period):**
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

**Response 422 — Outside radius (with acceptable accuracy):**
```json
{
  "error": {
    "code": "OUTSIDE_RADIUS",
    "message": "185m from venue, allowed radius is 50m."
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

Reports provide aggregated, read-only views of attendance, presence, emergency tickets, and volunteer participation.

No report endpoint modifies application data.

---

## GET /reports/dashboard

**Purpose:** Return live dashboard statistics.

**Auth required:** Yes (admin)

### Optional Query Parameters

- event_id

### Response

```json
{
  "success": true,
  "message": "Dashboard retrieved successfully.",
  "data": {
    "inside_volunteers": 42,
    "outside_volunteers": 3,
    "pending_tickets": 2,
    "approved_leaves": 4,
    "disconnected_volunteers": 1,
    "completed_volunteers": 35
    }
}
```

---

## GET /reports/events/{eventId}

Returns a complete report for a single event.

Includes:

- Event summary
- Attendance statistics
- Presence statistics
- Emergency ticket summary
- Block summary

---

## GET /reports/volunteers/{volunteerId}

Returns the volunteer's participation history.

Includes:

- Events attended
- Events completed
- Emergency tickets
- Block history
- Participation statistics

---

## GET /reports/volunteers/me

Returns the authenticated volunteer's own report.

---

## GET /reports/events/{eventId}/export

Query Parameter

```
format=csv
```

or

```
format=xlsx
```

Downloads the selected report.

---

## GET /reports/volunteers/{volunteerId}/export

Supports:

- csv
- xlsx

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
| `OUTSIDE_RADIUS` | 422 | `/attendance/check-in` | GPS distance exceeds allowed radius |
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

## Status Enums (must match `system_architecture.md §4` exactly)

**User role:**
- `admin`
- `member`

**Session status:**
- `scheduled` — created, not yet open
- `open` — members may check in
- `closed` — admin closed it or time window passed

**Attendance final_status:**
- `present` — checked in on time, inside radius
- `late` — checked in after grace period, inside radius
- `pending_verification` — GPS accuracy too low or other flag, needs admin review
- `rejected` — outside radius with acceptable accuracy, or failed validation

---

## Change Process

Any change to a path, field name, enum value, or status code is a **breaking contract change**.
Before merging:
1. Update this file on `main`.
2. Add an entry to `changelog.md`.
3. Notify the other developer (or leave a clear PR description) before either branch codes against
   the new shape.

Example changelog entry:
```
- Changed POST /attendance/check-in response to include `gps_accuracy_meters` field
- See API_contract.md § Attendance / Check-in → Response 201
```

# Activity History

Activity History records volunteer actions performed during an event.

This module is read-only.

---

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

Audit Logs record administrative actions performed within the system.

Audit records are immutable.

---

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

# Changelog

## Version 2.0.0

### Added

- Event boundary configuration using GeoJSON or captured GPS points.
- Continuous presence monitoring.
- GPS location update endpoints.
- Emergency ticket workflow.
- Volunteer blocking system.
- In-app notification APIs.
- Dashboard reporting endpoints.
- Volunteer activity history.
- Administrative audit log APIs.
- CSV and XLSX report export endpoints.
- Standard success response envelope.
- Extended error code reference.

### Changed

- Session terminology updated to Event throughout the documentation.
- Existing Phase 1 APIs remain backward compatible.
- API documentation expanded to support Phase 2 functionality.

### Compatibility

- No Phase 1 endpoint was removed.
- Existing request and response formats remain supported.
- Phase 2 functionality is additive and does not introduce breaking changes.