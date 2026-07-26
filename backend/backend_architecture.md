# backend_architecture.md — Backend Implementation Details

> How the FastAPI backend is structured internally: service layer, middleware, error handling, request/response flow, and validation patterns.

## 1. Layered Architecture

```
HTTP Request
    ↓
[Middleware: Auth, CORS, Logging]
    ↓
[Route Handler: Parse input, call service]
    ↓
[Service Layer: Business logic, validation, DB queries]
    ↓
[ORM Layer: SQLAlchemy models, database interactions]
    ↓
[Database: PostgreSQL]
```

## 2. Route Handlers (Routers)

Live in `app/api/` — one router per domain:
- `auth.py` — login, get current user
- `sessions.py` — create, list, detail, close
- `attendance.py` — check-in, history
- `reports.py` — Excel export

**Handler responsibility:**
- Parse request (Pydantic schema validation)
- Call appropriate service
- Return response (or let exception handling format errors)
- Never do business logic

**Pattern:**
```python
@router.post("/sessions", response_model=SessionRead)
async def create_session(req: SessionCreate, current_user: User = Depends(get_current_user)):
    if current_user.role != "admin":
        raise HTTPException(status_code=403, detail="Only admins can create sessions")
    return await sessions_service.create(req, current_user.id)
```

## 3. Service Layer

Live in `app/services/` — domain-specific business logic:
- `auth_service.py` — login, token management
- `sessions_service.py` — session CRUD, status logic
- `attendance_service.py` — check-in validation, distance calc
- `reports_service.py` — Excel generation
- `geo_service.py` — Haversine distance, location logic

**Service responsibility:**
- Validate business rules
- Call database/ORM
- Call other services if needed
- Raise `HTTPException` for business errors (not try/except in handlers)

**Pattern:**
```python
async def check_in(session_id: UUID, member_id: UUID, lat: float, lng: float, photo_file):
    session = await db.get_session(session_id)
    if not session or session.status != "open":
        raise HTTPException(422, "SESSION_NOT_OPEN")
    
    # Distance check
    distance = geo_service.haversine(lat, lng, session.venue_lat, session.venue_lng)
    if distance > session.radius_meters:
        raise HTTPException(422, "OUTSIDE_RADIUS")
    
    # Duplicate check
    existing = await db.get_attendance_record(session_id, member_id)
    if existing:
        raise HTTPException(409, "DUPLICATE_CHECK_IN")
    
    # Save record
    record = await db.create_attendance_record(...)
    return record
```

## 4. Error Handling

**Global exception handler** in `app/main.py`:
- Catches all exceptions
- Formats as standard error envelope (see API_contract.md)
- Logs to console/file

**Application-level exceptions:**
```python
class BusinessRuleError(Exception):
    def __init__(self, code: str, message: str, status_code: int = 422):
        self.code = code
        self.message = message
        self.status_code = status_code
```

**Pattern in services:**
```python
if not session.is_open():
    raise BusinessRuleError("SESSION_NOT_OPEN", "This session is not currently open.", 422)
```

## 5. Middleware

Order matters (top = first):

1. **Logging middleware** — log all requests/responses
2. **CORS middleware** — allow localhost:5173 (frontend)
3. **Auth middleware** — extract and validate JWT (applied per route via `Depends`)
4. **Error handling middleware** — catch exceptions and format

## 6. Database Session & Transactions

FastAPI + SQLAlchemy async pattern:

```python
# app/core/database.py
async def get_db():
    async with AsyncSession(engine) as session:
        try:
            yield session
        finally:
            await session.close()

# In services: dependency injection
async def create_session(req: SessionCreate, db: AsyncSession = Depends(get_db)):
    ...
    await db.commit()  # explicit, not auto-commit
```

Transactions are **per-request** — one DB session per HTTP request, auto-rollback on error.

## 7. Validation Strategy

**Three-tier validation:**

1. **Schema validation** — Pydantic (in handlers)
2. **Business rule validation** — Services (distance, time windows, duplicates)
3. **Database constraints** — Unique constraints, foreign keys (as backup)

**Never trust client input.** Server re-validates everything.

## 8. Server Time, Not Client Time

**Rule: All timestamps are server time.**

```python
from datetime import datetime, timezone

check_in_time = datetime.now(timezone.utc)  # server time, never client time
```

Store in DB as `TIMESTAMPTZ` (timezone-aware).

## 9. Dependency Injection Pattern

Use FastAPI's `Depends()` for:
- Database session
- Current authenticated user
- Configuration values

```python
async def get_current_user(token: str = Depends(oauth2_scheme)) -> User:
    payload = jwt.decode(token, SECRET_KEY)
    user_id = payload.get("sub")
    user = await db.get_user(user_id)
    if not user:
        raise HTTPException(401, "UNAUTHORIZED")
    return user

# In any handler:
@router.get("/me")
async def get_me(current_user: User = Depends(get_current_user)):
    return current_user
```

## 10. Async/Await Pattern

All I/O is async:
- Database queries: `await db.get_session(id)`
- File operations: `await asyncio.to_thread(file_write, ...)`
- External APIs: use `aiohttp`, never `requests`

Never block the event loop.

## 11. File Storage (Photo Evidence)

- Store in `media/YYYY/MM/DD/` folder on disk (or cloud bucket later)
- Store **reference path** in database (e.g., `media/2026/08/01/uuid.jpg`)
- Return reference path in API response, never the binary

```python
photo_path = f"media/{now.year}/{now.month:02d}/{now.day:02d}/{record_id}.jpg"
with open(photo_path, "wb") as f:
    f.write(photo_file.file.read())
record.photo_reference = photo_path
```

## 12. Logging

Use Python's `logging` module:
```python
import logging
logger = logging.getLogger(__name__)

logger.info(f"User {user_id} checked in to session {session_id}")
logger.error(f"Distance check failed: {distance}m > {radius}m", exc_info=True)
```

Log key business events (login, check-in, session close), not every function call.


## 13. Phase 2 Architecture Extensions

Phase 2 expands the backend architecture to support attendance lifecycle management, presence monitoring, administrator approvals, and enhanced reporting while preserving the existing layered architecture.

---

## 13.1 Updated Request Flow

```
HTTP Request

↓

Middleware

↓

Route Handler

↓

Service Layer

↓

Business Validation

↓

ORM

↓

Database

↓

Response
```

Business validation may involve multiple services before a response is returned.

---

## 13.2 Expanded Router Structure

Phase 2 extends the API layer.

```
app/api/

├── auth.py

├── sessions.py

├── attendance.py

├── monitoring.py

└── reports.py
```

Responsibilities:

attendance.py

- check in
- active attendance
- check out
- attendance summary
- early check-out workflow
- attendance timeline

monitoring.py

- live attendance
- monitoring statistics
- pending approvals

reports.py

- attendance exports
- enhanced reports

---

## 13.3 Expanded Service Layer

```
app/services/

├── auth_service.py

├── sessions_service.py

├── attendance_service.py

├── presence_service.py

├── monitoring_service.py

├── reports_service.py

└── geo_service.py
```

Responsibilities remain clearly separated.

AttendanceService

- attendance lifecycle

PresenceService

- presence monitoring

MonitoringService

- administrator dashboard data

ReportsService

- reporting only

GeoService

- location calculations

---

## 13.4 Service Interaction

Recommended architecture:

```
AttendanceService

↓

PresenceService

↓

GeoService
```

Administrator workflow:

```
MonitoringService

↓

AttendanceService

↓

ReportsService
```

Each service should expose clear public methods.

---

## 13.5 Attendance Lifecycle

Attendance lifecycle is managed by the backend.

```
Check In

↓

Active Attendance

↓

(Optional)

Early Check-out Request

↓

Administrator Decision

↓

Approved

↓

Check Out

↓

Attendance Summary
```

Only valid state transitions are permitted.

---

## 13.6 Presence Monitoring

Presence events are handled independently from attendance records.

Responsibilities:

- evaluate member location;
- generate presence events;
- maintain chronological timelines;
- determine current presence state.

Presence monitoring should remain independent from reporting.

---

## 13.7 Administrator Approval Workflow

Approval requests follow this architecture.

```
Member

↓

AttendanceService

↓

Database

↓

Administrator

↓

MonitoringService

↓

AttendanceService

↓

Database
```

Attendance completion occurs only after approval.

---

## 13.8 Error Handling

Phase 2 continues using centralized exception handling.

Additional business errors include:

- INVALID_ATTENDANCE_STATE
- APPROVAL_PENDING
- APPROVAL_REJECTED
- ACTIVE_ATTENDANCE_NOT_FOUND
- INVALID_PRESENCE_EVENT

All errors continue using the standard API response format.

---

## 13.9 Transaction Boundaries

Single database transactions should be used for operations that update multiple records.

Examples:

- attendance completion;
- approval processing;
- presence event creation.

Rollback the transaction if any operation fails.

---

## 13.10 Performance Considerations

Phase 2 introduces additional read-heavy endpoints.

Recommendations:

- optimize attendance queries;
- minimize joins;
- eager load related entities when appropriate;
- avoid N+1 query patterns;
- index frequently filtered columns.

---

## 13.11 Logging

Additional business events should be logged.

Examples:

- attendance completed;
- early check-out requested;
- request approved;
- request rejected;
- presence event recorded.

Sensitive information must never be logged.

---

## 13.12 Architecture Principles

Phase 2 preserves the existing backend architecture.

Core principles remain:

- thin route handlers;
- business logic inside services;
- dependency injection;
- asynchronous I/O;
- centralized error handling;
- backend as the source of truth;
- transactional consistency;
- independently testable services.