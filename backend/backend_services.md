# backend_services.md — Service Layer Architecture

> How business logic is organized into services: domain separation, dependency injection, error patterns, and testing strategy.

## 1. Service Layer Overview

Services live in `app/services/` — one file per domain:

```
app/services/
├── __init__.py
├── auth_service.py          # Login, token management
├── sessions_service.py       # Session CRUD, status logic
├── attendance_service.py     # Check-in validation, verification
├── reports_service.py        # Excel generation
├── geo_service.py            # Distance calculation
└── base_service.py           # Common patterns (optional)
```

**Service responsibility:**
- Encapsulate business logic
- Validate business rules
- Coordinate ORM/database calls
- Call other services if needed
- Raise descriptive exceptions

**Service does NOT:**
- Parse HTTP input (that's the route handler)
- Format HTTP responses (that's the route handler)
- Handle middleware concerns (that's middleware)

## 2. Service Pattern

```python
# app/services/sessions_service.py
from app.models import Session, AttendanceRecord
from app.schemas import SessionCreate
from sqlalchemy.ext.asyncio import AsyncSession
from fastapi import HTTPException

class SessionsService:
    def __init__(self, db: AsyncSession):
        self.db = db
    
    async def create(self, req: SessionCreate, admin_id: str) -> Session:
        """Create a new session."""
        # Validate business rules
        if req.end_time <= req.start_time:
            raise HTTPException(400, "INVALID_TIME_RANGE")
        
        # Create and persist
        session = Session(
            title=req.title,
            purpose=req.purpose,
            start_time=req.start_time,
            end_time=req.end_time,
            venue_lat=req.venue_lat,
            venue_lng=req.venue_lng,
            radius_meters=req.radius_meters,
            grace_period_minutes=req.grace_period_minutes,
            created_by=admin_id
        )
        self.db.add(session)
        await self.db.commit()
        await self.db.refresh(session)
        return session
    
    async def get_by_id(self, session_id: str, admin_id: str = None) -> Session:
        """Get a session. If admin_id provided, verify ownership."""
        session = await self.db.query(Session).filter(Session.id == session_id).first()
        if not session:
            raise HTTPException(404, "NOT_FOUND")
        if admin_id and session.created_by != admin_id:
            raise HTTPException(404, "NOT_FOUND")  # Don't leak ownership info
        return session
    
    async def close(self, session_id: str, admin_id: str) -> Session:
        """Close a session."""
        session = await self.get_by_id(session_id, admin_id)
        if session.status == "closed":
            raise HTTPException(409, "ALREADY_CLOSED")
        session.status = "closed"
        await self.db.commit()
        return session
```

## 3. Dependency Injection in Routes

Routes pass the service via dependency injection:

```python
# app/api/sessions.py
from fastapi import APIRouter, Depends
from app.core.database import get_db
from app.core.security import get_current_admin
from app.services.sessions_service import SessionsService

router = APIRouter(prefix="/sessions", tags=["sessions"])

@router.post("")
async def create_session(
    req: SessionCreate,
    current_user: User = Depends(get_current_admin),
    db: AsyncSession = Depends(get_db)
):
    service = SessionsService(db)
    return await service.create(req, current_user.id)

@router.get("/{id}")
async def get_session(
    id: str,
    current_user: User = Depends(get_current_admin),
    db: AsyncSession = Depends(get_db)
):
    service = SessionsService(db)
    return await service.get_by_id(id, current_user.id)
```

**Why DI?**
- Services are testable (pass a mock DB)
- No global state
- Clear dependencies

## 4. Service: auth_service.py

```python
# app/services/auth_service.py
from app.models import User
from app.core.security import hash_password, verify_password, create_access_token

class AuthService:
    def __init__(self, db: AsyncSession):
        self.db = db
    
    async def login(self, email: str, password: str) -> dict:
        """Authenticate user and return JWT + profile."""
        user = await self.db.query(User).filter(User.email == email).first()
        
        if not user or not verify_password(password, user.password_hash):
            raise HTTPException(401, "INVALID_CREDENTIALS")
        
        token = create_access_token(str(user.id), user.email)
        return {
            "access_token": token,
            "token_type": "bearer",
            "expires_in_minutes": 60,
            "user": {
                "id": user.id,
                "full_name": user.full_name,
                "email": user.email,
                "role": user.role
            }
        }
    
    async def get_user(self, user_id: str) -> User:
        """Get user by ID."""
        user = await self.db.query(User).filter(User.id == user_id).first()
        if not user:
            raise HTTPException(401, "UNAUTHORIZED")
        return user
```

## 5. Service: attendance_service.py (Complex)

```python
# app/services/attendance_service.py
from app.models import AttendanceRecord, Session
from app.services.geo_service import GeoService
from datetime import datetime, timezone

class AttendanceService:
    def __init__(self, db: AsyncSession):
        self.db = db
        self.geo = GeoService()
    
    async def check_in(
        self,
        session_id: str,
        member_id: str,
        lat: float,
        lng: float,
        gps_accuracy: float,
        photo_path: str
    ) -> AttendanceRecord:
        """
        Process a check-in submission.
        
        Validation steps:
        1. Session exists and is open
        2. Member not already checked in
        3. Within time window
        4. Within radius (or flag pending_verification if accuracy poor)
        """
        
        # 1. Fetch session
        session = await self.db.query(Session).filter(Session.id == session_id).first()
        if not session:
            raise HTTPException(404, "NOT_FOUND")
        if session.status != "open":
            raise HTTPException(422, "SESSION_NOT_OPEN")
        
        # 2. Check for duplicate
        existing = await self.db.query(AttendanceRecord).filter(
            AttendanceRecord.session_id == session_id,
            AttendanceRecord.member_id == member_id
        ).first()
        if existing:
            raise HTTPException(409, "DUPLICATE_CHECK_IN")
        
        # 3. Time window check
        now = datetime.now(timezone.utc)
        if now < session.start_time or now > session.end_time:
            raise HTTPException(422, "SESSION_NOT_OPEN")
        
        # 4. Distance & accuracy check
        distance = self.geo.haversine(
            lat, lng,
            session.venue_lat, session.venue_lng
        )
        
        # Determine final status
        if distance > session.radius_meters:
            # Outside radius
            if gps_accuracy <= 30:
                # Accuracy acceptable, reject
                raise HTTPException(422, "OUTSIDE_RADIUS")
            else:
                # Poor accuracy, flag for review
                final_status = "pending_verification"
                flag_reason = "GPS_ACCURACY_TOO_LOW"
        else:
            # Inside radius
            if now < (session.start_time + timedelta(minutes=session.grace_period_minutes)):
                final_status = "present"
            else:
                final_status = "late"
            flag_reason = None
        
        # 5. Create record
        record = AttendanceRecord(
            session_id=session_id,
            member_id=member_id,
            check_in_time=now,
            check_in_lat=lat,
            check_in_lng=lng,
            distance_meters=distance,
            gps_accuracy_meters=gps_accuracy,
            photo_reference=photo_path,
            final_status=final_status,
            rejection_reason=flag_reason
        )
        self.db.add(record)
        await self.db.commit()
        await self.db.refresh(record)
        return record
    
    async def get_member_history(self, member_id: str) -> list:
        """Get all check-ins for this member."""
        records = await self.db.query(AttendanceRecord).filter(
            AttendanceRecord.member_id == member_id
        ).order_by(AttendanceRecord.created_at.desc()).all()
        return records
```

## 6. Service: geo_service.py (Utility)

```python
# app/services/geo_service.py
import math

class GeoService:
    """Geographic calculations."""
    
    EARTH_RADIUS_METERS = 6371000  # Earth's radius in meters
    
    def haversine(self, lat1: float, lng1: float, lat2: float, lng2: float) -> float:
        """
        Calculate great-circle distance between two points (in meters).
        
        Formula: d = 2 * R * arcsin(sqrt(sin²(Δφ/2) + cos(φ1) * cos(φ2) * sin²(Δλ/2)))
        """
        φ1 = math.radians(lat1)
        φ2 = math.radians(lat2)
        Δφ = math.radians(lat2 - lat1)
        Δλ = math.radians(lng2 - lng1)
        
        a = (math.sin(Δφ / 2) ** 2 +
             math.cos(φ1) * math.cos(φ2) * math.sin(Δλ / 2) ** 2)
        c = 2 * math.atan2(math.sqrt(a), math.sqrt(1 - a))
        
        distance = self.EARTH_RADIUS_METERS * c
        return distance
```

## 7. Service: reports_service.py

```python
# app/services/reports_service.py
from openpyxl import Workbook
from app.models import Session, AttendanceRecord

class ReportsService:
    def __init__(self, db: AsyncSession):
        self.db = db
    
    async def generate_attendance_xlsx(self, session_id: str, admin_id: str):
        """Generate Excel workbook for a session."""
        session = await self.db.query(Session).filter(Session.id == session_id).first()
        if not session or session.created_by != admin_id:
            raise HTTPException(404, "NOT_FOUND")
        
        records = await self.db.query(AttendanceRecord).filter(
            AttendanceRecord.session_id == session_id
        ).all()
        
        wb = Workbook()
        
        # Sheet 1: Attendance Summary
        ws1 = wb.active
        ws1.title = "Attendance Summary"
        ws1.append(["Member ID", "Name", "Session", "Date", "Check-in Time", "Distance (m)", "Final Status"])
        for record in records:
            member = await self.db.query(User).filter(User.id == record.member_id).first()
            ws1.append([
                record.member_id,
                member.full_name,
                session.title,
                session.date,
                record.check_in_time,
                record.distance_meters,
                record.final_status
            ])
        
        # Sheet 2: Photo Evidence
        ws2 = wb.create_sheet("Photo Evidence")
        ws2.append(["Member ID", "Session ID", "Photo Reference", "Capture Time"])
        for record in records:
            ws2.append([
                record.member_id,
                record.session_id,
                record.photo_reference,
                record.check_in_time
            ])
        
        return wb  # Caller saves to file or streams to HTTP response
```

## 8. Exception Strategy

**Raise HTTPException, don't catch silently:**

```python
# ✗ Bad
try:
    session = await self.db.query(Session).filter(...).first()
except Exception:
    session = None  # Silent failure

# ✓ Good
session = await self.db.query(Session).filter(...).first()
if not session:
    raise HTTPException(404, "NOT_FOUND")
```

**Use descriptive error codes:**
```python
raise HTTPException(409, "DUPLICATE_CHECK_IN")
raise HTTPException(422, "OUTSIDE_RADIUS")
raise HTTPException(422, "SESSION_NOT_OPEN")
```

## 9. Transaction Management

Services handle commits/rollbacks:

```python
try:
    record = AttendanceRecord(...)
    self.db.add(record)
    await self.db.commit()  # Explicit commit
except Exception:
    await self.db.rollback()  # Rollback on error
    raise
```

## 10. Service Testing (Unit Tests)

Mock the database:

```python
# tests/test_attendance_service.py
import pytest
from unittest.mock import AsyncMock, MagicMock
from app.services.attendance_service import AttendanceService

@pytest.mark.asyncio
async def test_check_in_duplicate():
    """Verify duplicate check-in is rejected."""
    # Mock DB
    mock_db = AsyncMock()
    
    # Mock existing record (duplicate)
    mock_db.query().filter().first = AsyncMock(return_value=MagicMock())
    
    service = AttendanceService(mock_db)
    
    with pytest.raises(HTTPException) as exc_info:
        await service.check_in(session_id="123", member_id="456", ...)
    
    assert exc_info.value.detail == "DUPLICATE_CHECK_IN"
    assert exc_info.value.status_code == 409
```

## 11. Service Layer Summary

| Aspect | Rule |
|---|---|
| **Location** | `app/services/*.py` |
| **Responsibility** | Business logic, validation, DB queries |
| **Error handling** | Raise HTTPException with error code |
| **Testing** | Mock the DB, test service in isolation |
| **Dependencies** | Inject DB, other services via `__init__` |
| **No global state** | Every service instance is fresh (created per request) |


## 12. Phase 2 Service Layer Extensions

Phase 2 extends the service layer to support the complete attendance lifecycle, presence monitoring, administrator approvals, and enhanced reporting.

---

## 12.1 Updated Service Structure

```
app/services/

├── auth_service.py

├── sessions_service.py

├── attendance_service.py

├── presence_service.py

├── monitoring_service.py

├── activity_service.py

├── activity_assignment_service.py

├── activity_progress_service.py

├── activity_review_service.py

├── activity_template_service.py

├── reports_service.py

├── geo_service.py

└── notification_service.py (optional)

Each service should own a single business domain.

---

## 12.2 AttendanceService Responsibilities

AttendanceService now manages the complete attendance lifecycle.

Responsibilities include:

- check in
- active attendance retrieval
- attendance status
- attendance completion
- attendance duration calculation
- attendance summary generation
- attendance state validation

Example methods:

```python
check_in()

get_active_attendance()

complete_check_out()

generate_summary()

validate_transition()
```

AttendanceService should not manage administrator approvals directly.

---

## 12.3 PresenceService

Presence monitoring should be isolated into its own service.

Responsibilities:

- evaluate member location
- determine inside/outside status
- create presence events
- build attendance timeline
- prevent duplicate events

Example methods:

```python
record_presence()

create_event()

get_timeline()

get_current_status()
```

---

## 12.4 MonitoringService

MonitoringService provides administrator dashboards.

Responsibilities:

- active attendance
- attendance statistics
- pending approvals
- completed attendance
- live monitoring summaries

Example methods:

```python
get_live_statistics()

get_active_members()

get_pending_requests()

get_monitoring_summary()
```

MonitoringService should not modify attendance records.

---

## 12.5 ReportsService

ReportsService should generate reports only.

Responsibilities:

- attendance summary export
- attendance timeline export
- approval information
- duration calculations for reports

Report formatting belongs inside this service.

---

## 12.6 Service Communication

Services may collaborate but should avoid circular dependencies.

Recommended flow:

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

Services should communicate through clearly defined method calls.

---

## 12.7 Business Rule Ownership

Business rules belong in only one service.

Examples:

Attendance validation

→ AttendanceService

Distance calculation

→ GeoService

Presence evaluation

→ PresenceService

Monitoring statistics

→ MonitoringService

Report generation

→ ReportsService

Avoid duplicating business rules across services.

---

## 12.8 State Management

Attendance state transitions must be validated before changes are committed.

Example:

```
Checked In

↓

Active

↓

Pending Approval

↓

Approved

↓

Checked Out
```

Invalid transitions should raise appropriate exceptions.

---

## 12.9 Database Transactions

Operations affecting multiple tables should execute within a single transaction.

Examples:

- attendance completion;
- approval processing;
- timeline creation.

Rollback the transaction if any operation fails.

---

## 12.10 Service Error Handling

Services should raise descriptive business exceptions.

Examples:

```python
raise HTTPException(409, "INVALID_ATTENDANCE_STATE")

raise HTTPException(409, "APPROVAL_PENDING")

raise HTTPException(404, "ACTIVE_ATTENDANCE_NOT_FOUND")

raise HTTPException(422, "INVALID_PRESENCE_EVENT")
```

Avoid returning partially updated business objects.

---

## 12.11 Service Testing

Each service should be tested independently.

AttendanceService:

- lifecycle
- state transitions
- summary generation

PresenceService:

- presence detection
- timeline creation
- duplicate prevention

MonitoringService:

- statistics
- active attendance
- pending approvals

ReportsService:

- workbook generation
- attendance summaries
- export formatting

Mock database interactions where appropriate.

---

## 12.12 Phase 2 Service Principles

All Phase 2 services should follow these principles:

- Single Responsibility Principle.
- Business logic belongs only in services.
- Route handlers remain thin.
- Services communicate through clear interfaces.
- Database operations are transactional.
- Business rules are implemented once and reused.
- Services remain independently testable.

---

# 12.13 Phase 3 Service Layer Extensions

Phase 3 extends the service layer by introducing activity management, volunteer assignments, progress tracking, evidence management, activity review, and reusable activity templates.

Each service owns a single business domain and communicates with other services through well-defined interfaces.

---

## 12.14 ActivityService

ActivityService manages the lifecycle of activities.

Responsibilities:

- create activities
- update activities
- publish activities
- cancel activities
- archive activities

Example methods:

```python
create_activity()

update_activity()

publish_activity()

cancel_activity()

archive_activity()
```

ActivityService should not manage volunteer assignments or evidence.

---

## 12.15 ActivityAssignmentService

ActivityAssignmentService manages volunteer assignments.

Responsibilities:

- assign volunteers
- validate assignment conflicts
- retrieve assignments
- maintain assignment status

Example methods:

```python
assign_member()

remove_assignment()

get_assignments()

validate_assignment()
```

AssignmentService should not manage progress updates or reviews.

---

## 12.16 ActivityProgressService

ActivityProgressService manages activity execution.

Responsibilities:

- create progress updates
- upload evidence
- validate activity completion
- submit activities for review

Example methods:

```python
add_progress()

upload_photo()

upload_video()

submit_for_review()
```

ProgressService should not perform administrative reviews.

---

## 12.17 ActivityReviewService

ActivityReviewService manages administrator reviews.

Responsibilities:

- retrieve pending reviews
- verify completed activities
- request changes
- preserve review history

Example methods:

```python
get_pending_reviews()

verify_activity()

request_changes()

get_review_history()
```

ReviewService should never modify activity progress.

---

## 12.18 ActivityTemplateService

ActivityTemplateService manages reusable templates.

Responsibilities:

- create templates
- update templates
- retrieve templates
- generate activities

Example methods:

```python
create_template()

update_template()

apply_template()

get_templates()
```

Templates should only define reusable activity structures.

---

## 12.19 Activity Service Communication

Activity services communicate through clearly defined interfaces.

Recommended workflow:

```
ActivityService

↓

ActivityAssignmentService

↓

ActivityProgressService

↓

ActivityReviewService
```

Supporting services:

```
ActivityProgressService

↓

GeoService
```

Administrative workflow:

```
ActivityReviewService

↓

ReportsService
```

Circular dependencies should be avoided.

---

## 12.20 Activity Business Rule Ownership

Each business rule belongs to one service only.

Examples:

Activity lifecycle

→ ActivityService

Assignment validation

→ ActivityAssignmentService

Progress updates

→ ActivityProgressService

Evidence validation

→ ActivityProgressService

Review decisions

→ ActivityReviewService

Template generation

→ ActivityTemplateService

Report generation

→ ReportsService

Avoid implementing the same business rule in multiple services.

---

## 12.21 Activity State Management

Assignment state transitions must be validated before changes are committed.

Example:

```
ASSIGNED

↓

IN_PROGRESS

↓

UNDER_REVIEW

↓

VERIFIED
```

or

```
UNDER_REVIEW

↓

NEEDS_CHANGES

↓

IN_PROGRESS

↓

UNDER_REVIEW
```

Invalid transitions should raise appropriate exceptions.

---

## 12.22 Activity Transactions

Operations affecting multiple tables should execute within a single database transaction.

Examples:

- activity assignment;
- activity submission;
- evidence upload;
- review completion;
- template application.

Rollback the transaction if any operation fails.

---

## 12.23 Activity Service Testing

Each Activity service should be tested independently.

ActivityService:

- lifecycle
- publication
- cancellation

ActivityAssignmentService:

- assignment validation
- duplicate prevention
- conflict detection

ActivityProgressService:

- progress updates
- evidence upload
- submission validation

ActivityReviewService:

- verification
- needs changes
- review history

ActivityTemplateService:

- template creation
- template application
- generated activities

Mock database interactions where appropriate.

---

## 12.24 Phase 3 Service Principles

All Phase 3 services should follow these principles:

- Single Responsibility Principle.
- Activities remain independent of attendance records.
- Business logic belongs only in services.
- Route handlers remain thin.
- Services communicate through clear interfaces.
- Database operations are transactional.
- Review history is preserved.
- Progress updates are append-only.
- Templates generate independent activities.
- Services remain independently testable.