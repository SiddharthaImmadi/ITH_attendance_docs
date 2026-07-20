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
