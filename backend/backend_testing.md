# backend_testing.md — Testing Strategy & Setup

> How to test the backend: unit tests, integration tests, fixtures, and running the test suite.

## 1. Test Structure

```
backend/
├── tests/
│   ├── __init__.py
│   ├── conftest.py                    # Shared fixtures
│   ├── test_auth.py                   # Auth endpoint tests
│   ├── test_sessions.py               # Session endpoint tests
│   ├── test_attendance.py             # Check-in endpoint tests
│   ├── test_reports.py                # Report endpoint tests
│   ├── unit/
│   │   ├── test_geo_service.py        # Haversine distance tests
│   │   ├── test_auth_service.py       # Auth service tests
│   │   └── test_attendance_service.py # Check-in logic tests
│   └── integration/
│       └── test_end_to_end.py         # Full workflows
```

## 2. Testing Tools & Setup

```bash
# Install test dependencies
pip install pytest pytest-asyncio pytest-mock httpx

# Run all tests
pytest

# Run with coverage
pytest --cov=app tests/

# Run specific test file
pytest tests/test_auth.py

# Run in verbose mode
pytest -v

# Stop on first failure
pytest -x
```

## 3. Shared Fixtures (conftest.py)

```python
# tests/conftest.py
import pytest
from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession
from sqlalchemy.orm import sessionmaker
from app.core.database import Base
from app.core.security import hash_password
from app.models import User, Session
from datetime import datetime, timedelta, timezone

@pytest.fixture
async def test_db():
    """Create in-memory SQLite database for testing."""
    engine = create_async_engine("sqlite+aiosqlite:///:memory:")
    
    async with engine.begin() as conn:
        await conn.run_sync(Base.metadata.create_all)
    
    async_session_maker = sessionmaker(engine, class_=AsyncSession, expire_on_commit=False)
    
    async with async_session_maker() as session:
        yield session
    
    await engine.dispose()

@pytest.fixture
async def test_user(test_db):
    """Create a test admin user."""
    user = User(
        id="test-admin-id",
        full_name="Test Admin",
        email="admin@test.com",
        password_hash=hash_password("TestPass123!"),
        role="admin"
    )
    test_db.add(user)
    await test_db.commit()
    return user

@pytest.fixture
async def test_member(test_db):
    """Create a test member user."""
    user = User(
        id="test-member-id",
        full_name="Test Member",
        email="member@test.com",
        password_hash=hash_password("MemberPass123!"),
        role="member"
    )
    test_db.add(user)
    await test_db.commit()
    return user

@pytest.fixture
async def test_session(test_db, test_user):
    """Create a test session."""
    now = datetime.now(timezone.utc)
    session = Session(
        id="test-session-id",
        title="Test Session",
        purpose="For testing",
        date=now.date(),
        start_time=now,
        end_time=now + timedelta(hours=3),
        grace_period_minutes=10,
        venue_lat=12.9716,
        venue_lng=77.5946,
        radius_meters=50,
        status="open",
        created_by=test_user.id
    )
    test_db.add(session)
    await test_db.commit()
    return session

@pytest.fixture
def client(test_db):
    """FastAPI test client."""
    from fastapi.testclient import TestClient
    from app.main import app
    from app.core.database import get_db
    
    async def override_get_db():
        yield test_db
    
    app.dependency_overrides[get_db] = override_get_db
    
    with TestClient(app) as client:
        yield client
    
    app.dependency_overrides.clear()
```

## 4. Unit Tests (Example: Auth Service)

```python
# tests/unit/test_auth_service.py
import pytest
from fastapi import HTTPException
from app.services.auth_service import AuthService

@pytest.mark.asyncio
async def test_login_success(test_db, test_user):
    """Verify successful login returns JWT and user profile."""
    service = AuthService(test_db)
    
    result = await service.login("admin@test.com", "TestPass123!")
    
    assert result["access_token"]
    assert result["token_type"] == "bearer"
    assert result["user"]["email"] == "admin@test.com"
    assert result["user"]["role"] == "admin"

@pytest.mark.asyncio
async def test_login_invalid_email(test_db):
    """Verify login fails with non-existent email."""
    service = AuthService(test_db)
    
    with pytest.raises(HTTPException) as exc_info:
        await service.login("nonexistent@test.com", "password")
    
    assert exc_info.value.status_code == 401
    assert "INVALID_CREDENTIALS" in exc_info.value.detail

@pytest.mark.asyncio
async def test_login_invalid_password(test_db, test_user):
    """Verify login fails with wrong password."""
    service = AuthService(test_db)
    
    with pytest.raises(HTTPException) as exc_info:
        await service.login("admin@test.com", "WrongPassword123!")
    
    assert exc_info.value.status_code == 401
```

## 5. Integration Tests (Endpoint Tests)

```python
# tests/test_auth.py
import pytest
from fastapi.testclient import TestClient

def test_login_endpoint(client, test_user):
    """Test POST /auth/login endpoint."""
    response = client.post(
        "/auth/login",
        json={
            "email": "admin@test.com",
            "password": "TestPass123!"
        }
    )
    
    assert response.status_code == 200
    data = response.json()
    assert data["access_token"]
    assert data["user"]["role"] == "admin"

def test_get_me_endpoint(client, test_user):
    """Test GET /me endpoint."""
    # First login to get token
    login_response = client.post(
        "/auth/login",
        json={"email": "admin@test.com", "password": "TestPass123!"}
    )
    token = login_response.json()["access_token"]
    
    # Call /me with token
    response = client.get(
        "/me",
        headers={"Authorization": f"Bearer {token}"}
    )
    
    assert response.status_code == 200
    data = response.json()
    assert data["email"] == "admin@test.com"

def test_login_invalid_credentials(client):
    """Test login with wrong password."""
    response = client.post(
        "/auth/login",
        json={
            "email": "admin@test.com",
            "password": "WrongPassword"
        }
    )
    
    assert response.status_code == 401
    assert response.json()["error"]["code"] == "INVALID_CREDENTIALS"
```

## 6. Testing Attendance Check-in (Complex)

```python
# tests/test_attendance.py
import pytest
from app.services.attendance_service import AttendanceService
from datetime import datetime, timezone

@pytest.mark.asyncio
async def test_check_in_success(test_db, test_session, test_member):
    """Test successful check-in inside radius."""
    service = AttendanceService(test_db)
    
    record = await service.check_in(
        session_id=test_session.id,
        member_id=test_member.id,
        lat=12.9716,        # Same as venue
        lng=77.5946,
        gps_accuracy=12,    # Good accuracy
        photo_path="media/2026/08/01/test.jpg"
    )
    
    assert record.final_status == "present"
    assert record.distance_meters < 50  # Within radius

@pytest.mark.asyncio
async def test_check_in_outside_radius(test_db, test_session, test_member):
    """Test check-in outside radius is rejected."""
    service = AttendanceService(test_db)
    
    with pytest.raises(HTTPException) as exc_info:
        await service.check_in(
            session_id=test_session.id,
            member_id=test_member.id,
            lat=13.1939,      # ~20km away
            lng=77.6245,
            gps_accuracy=12,
            photo_path="media/2026/08/01/test.jpg"
        )
    
    assert exc_info.value.status_code == 422
    assert "OUTSIDE_RADIUS" in exc_info.value.detail

@pytest.mark.asyncio
async def test_check_in_duplicate(test_db, test_session, test_member):
    """Test duplicate check-in is rejected."""
    service = AttendanceService(test_db)
    
    # First check-in succeeds
    await service.check_in(
        session_id=test_session.id,
        member_id=test_member.id,
        lat=12.9716,
        lng=77.5946,
        gps_accuracy=12,
        photo_path="media/2026/08/01/test.jpg"
    )
    
    # Second check-in fails
    with pytest.raises(HTTPException) as exc_info:
        await service.check_in(
            session_id=test_session.id,
            member_id=test_member.id,
            lat=12.9716,
            lng=77.5946,
            gps_accuracy=12,
            photo_path="media/2026/08/01/test2.jpg"
        )
    
    assert exc_info.value.status_code == 409
    assert "DUPLICATE_CHECK_IN" in exc_info.value.detail
```

## 7. Testing Distance Calculation (GeoService)

```python
# tests/unit/test_geo_service.py
import pytest
from app.services.geo_service import GeoService

def test_haversine_distance():
    """Test Haversine distance calculation."""
    geo = GeoService()
    
    # Distance from Bangalore to Delhi: ~2,040 km
    distance = geo.haversine(
        lat1=12.9716,  # Bangalore
        lng1=77.5946,
        lat2=28.7041,  # Delhi
        lng2=77.1025
    )
    
    # Should be approximately 2,040,000 meters (±5%)
    assert 1_938_000 < distance < 2_142_000
    
    # Distance from point to itself should be ~0
    distance = geo.haversine(12.9716, 77.5946, 12.9716, 77.5946)
    assert distance < 1  # Less than 1 meter

def test_haversine_accuracy():
    """Test distance is within acceptable accuracy."""
    geo = GeoService()
    
    # Two points 100m apart
    distance = geo.haversine(
        12.9716, 77.5946,
        12.97175, 77.59485  # ~100m northeast
    )
    
    # Should be approximately 100 meters
    assert 90 < distance < 110
```

## 8. Test Database Cleanup

Tests use an in-memory SQLite database that's recreated for each test, so no cleanup needed. For real PostgreSQL tests (integration suite):

```python
@pytest.fixture
async def real_test_db():
    """Connect to real PostgreSQL for integration tests."""
    engine = create_async_engine(
        "postgresql+asyncpg://user:pass@localhost/test_db"
    )
    
    async with engine.begin() as conn:
        await conn.run_sync(Base.metadata.drop_all)
        await conn.run_sync(Base.metadata.create_all)
    
    async_session = sessionmaker(engine, class_=AsyncSession)
    
    async with async_session() as session:
        yield session
    
    # Cleanup
    async with engine.begin() as conn:
        await conn.run_sync(Base.metadata.drop_all)
    
    await engine.dispose()
```

## 9. Acceptance Criteria Tests (From PRD.md §7)

Map PRD acceptance criteria to test cases:

| PRD ID | Test | Covers |
|---|---|---|
| F01 | `test_check_in_success` | Eligible member checks in inside radius and within time |
| F02 | `test_check_in_duplicate` | Duplicate check-in blocked |
| F03 | `test_check_in_outside_radius` | Outside radius rejected or pending_verification |
| F04 | `test_check_in_poor_gps_accuracy` | GPS accuracy too low → pending_verification |
| F10 | `test_export_attendance_xlsx` | Excel report generated and matches database |

## 10. Running Tests in CI/CD

```bash
# Run tests with coverage report
pytest --cov=app --cov-report=html tests/

# Run only fast unit tests (skip integration)
pytest tests/unit/

# Run with specific markers
pytest -m "not integration" tests/

# Generate JUnit XML for CI tools
pytest --junitxml=test-results.xml tests/
```

## 11. Test Coverage Target

**Phase 1 goal: 70% coverage minimum**

```bash
pytest --cov=app tests/

# Output:
# Name                      Stmts   Miss  Cover
# ─────────────────────────────────────────────
# app/api/auth.py              20      2    90%
# app/api/sessions.py          35      5    86%
# app/services/auth_service.py 25      0   100%
# ...
# TOTAL                       300     90    70%
```

Higher priority to test:
- Auth logic (must be correct)
- Check-in validation (core business logic)
- Distance calculation (single point of failure)

Lower priority:
- Response formatting (caught by manual testing)
- Error messages (low impact)
