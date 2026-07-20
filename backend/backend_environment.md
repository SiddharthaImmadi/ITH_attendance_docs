# backend_environment.md — Environment Variables & Configuration

> All environment variables required by the backend (and shared between frontend/backend), with defaults and generation steps.

## 1. Backend-Only Variables

These control backend behavior and secrets:

| Variable | Type | Default | Required | Notes |
|---|---|---|---|---|
| `DATABASE_URL` | string | none | **YES** | PostgreSQL connection string |
| `JWT_SECRET_KEY` | string | none | **YES** | Secret key for signing JWT tokens (min 32 chars) |
| `JWT_ALGORITHM` | string | `HS256` | no | JWT signing algorithm |
| `JWT_EXPIRE_MINUTES` | int | `60` | no | Token expiration time |
| `ENVIRONMENT` | string | `development` | no | `development`, `staging`, or `production` |
| `LOG_LEVEL` | string | `INFO` | no | `DEBUG`, `INFO`, `WARNING`, `ERROR` |
| `MEDIA_ROOT` | string | `./media` | no | Directory for storing photo evidence |

## 2. Shared Variables (Frontend + Backend)

These must match between frontend and backend:

| Variable | Where | Type | Notes |
|---|---|---|---|
| `API_BASE_URL` | Backend: `CORS_ORIGINS` | string | Allowed frontend URL (for CORS) — typically `http://localhost:5173` (dev) |
| `FRONTEND_URL` | Backend: `CORS_ORIGINS` | string | Used in error responses, email links (future) |

## 3. Example `.env` Files

### Backend `.env` (git-ignored, local development)

```
# Database
DATABASE_URL=postgresql://postgres:devpassword@localhost:5432/attendance_app_dev

# JWT
JWT_SECRET_KEY=your-random-secret-key-min-32-chars-long
JWT_ALGORITHM=HS256
JWT_EXPIRE_MINUTES=60

# CORS / Frontend
CORS_ORIGINS=http://localhost:5173,http://127.0.0.1:5173

# App config
ENVIRONMENT=development
LOG_LEVEL=DEBUG
MEDIA_ROOT=./media
```

### `.env.example` (committed to repo, for documentation)

```
# PostgreSQL connection string
# Format: postgresql://[user]:[password]@[host]:[port]/[database]
DATABASE_URL=postgresql://postgres:your_password@localhost:5432/attendance_app_dev

# JWT secret — generate with: python -c "import secrets; print(secrets.token_urlsafe(32))"
JWT_SECRET_KEY=your_secret_key_here_min_32_chars

# JWT settings
JWT_ALGORITHM=HS256
JWT_EXPIRE_MINUTES=60

# Frontend URL (for CORS allow-list)
CORS_ORIGINS=http://localhost:5173,http://127.0.0.1:5173

# Environment: development, staging, production
ENVIRONMENT=development

# Logging level: DEBUG, INFO, WARNING, ERROR
LOG_LEVEL=DEBUG

# Directory to store uploaded photos
MEDIA_ROOT=./media
```

## 4. How to Generate `JWT_SECRET_KEY`

```bash
# Option 1: Python
python -c "import secrets; print(secrets.token_urlsafe(32))"

# Option 2: OpenSSL
openssl rand -hex 32

# Option 3: Linux/Mac
head -c 32 /dev/urandom | base64
```

Output example:
```
eJyLjgUAARUBuQ==  # (actual output will vary)
```

Copy the output into your `.env` file as the value of `JWT_SECRET_KEY`.

## 5. PostgreSQL Connection String Format

```
postgresql://[user]:[password]@[host]:[port]/[database]
```

**Examples:**

Local (Docker):
```
postgresql://postgres:devpassword@localhost:5432/attendance_app_dev
```

Local (native install):
```
postgresql://postgres:your_pg_password@127.0.0.1:5432/attendance_app_dev
```

Remote (future, production):
```
postgresql://user:password@db.example.com:5432/attendance_app_prod
```

## 6. Loading Environment Variables in Code

```python
# app/core/config.py
from pydantic_settings import BaseSettings
import os

class Settings(BaseSettings):
    database_url: str = os.getenv("DATABASE_URL")
    jwt_secret_key: str = os.getenv("JWT_SECRET_KEY")
    jwt_algorithm: str = os.getenv("JWT_ALGORITHM", "HS256")
    jwt_expire_minutes: int = int(os.getenv("JWT_EXPIRE_MINUTES", "60"))
    cors_origins: list = os.getenv("CORS_ORIGINS", "").split(",")
    environment: str = os.getenv("ENVIRONMENT", "development")
    log_level: str = os.getenv("LOG_LEVEL", "INFO")
    media_root: str = os.getenv("MEDIA_ROOT", "./media")

settings = Settings()

# Usage:
# from app.core.config import settings
# print(settings.jwt_expire_minutes)  # 60
```

## 7. CORS Configuration

```python
# app/main.py
from fastapi.middleware.cors import CORSMiddleware
from app.core.config import settings

app.add_middleware(
    CORSMiddleware,
    allow_origins=settings.cors_origins,  # ["http://localhost:5173", ...]
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```

This allows the frontend running on `localhost:5173` to make cross-origin requests to the backend on `localhost:8000`.

## 8. Secret Handling Best Practices

| Practice | Implementation |
|---|---|
| Never commit `.env` | Add to `.gitignore`: `backend/.env` |
| Version `.env.example` instead | Commit `.env.example` with placeholders |
| Use strong secrets | Min 32 random characters (use `secrets` module) |
| Rotate regularly (production) | Change `JWT_SECRET_KEY` and redeploy every 6 months |
| Don't log secrets | Never print or log `JWT_SECRET_KEY`, passwords, tokens |
| Use environment-specific values | Dev ≠ staging ≠ production secrets |

## 9. Logging Configuration

```python
# app/core/logging.py
import logging
from app.core.config import settings

logging.basicConfig(
    level=settings.log_level,
    format="%(asctime)s - %(name)s - %(levelname)s - %(message)s"
)

logger = logging.getLogger(__name__)
```

**Log levels:**
- `DEBUG` — detailed info, useful for development
- `INFO` — general info (logins, check-ins, session closes)
- `WARNING` — something unexpected but not an error
- `ERROR` — error occurred but app continues

## 10. Media Directory

Photos are stored in `MEDIA_ROOT` (default: `./media`):

```
media/
├── 2026/
│   ├── 08/
│   │   ├── 01/
│   │   │   ├── uuid1.jpg
│   │   │   └── uuid2.jpg
│   │   └── 02/
│   │       └── uuid3.jpg
```

**In development:** Create `media/` folder in project root. `.gitignore` it (photos are large).

**In production:** Store in S3 or similar (not included in Phase 1).

## 11. Database Connection Pooling

```python
# app/core/database.py
from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession
from sqlalchemy.pool import NullPool, QueuePool

engine = create_async_engine(
    settings.database_url,
    poolclass=QueuePool,
    pool_size=5,            # Max 5 connections in pool
    max_overflow=10,        # Allow up to 10 temporary connections
    pool_recycle=3600,      # Recycle connections every 1 hour
    echo=False              # Set to True for SQL logging (DEBUG only)
)
```

**For local dev with SQLite (if you want to switch):**
```python
engine = create_async_engine(
    "sqlite+aiosqlite:///./attendance.db",
    poolclass=NullPool,  # SQLite doesn't need connection pooling
    echo=False
)
```

## 12. Environment Variable Checklist

Before running the backend, verify:

```bash
# Check backend/.env exists
ls -la backend/.env

# Check required variables are set
echo $DATABASE_URL
echo $JWT_SECRET_KEY

# Test database connection
python -m alembic upgrade head  # Run migrations

# Start backend
uvicorn app.main:app --reload --port 8000
```

If you get "DATABASE_URL not set" error, copy `.env.example` to `.env` and fill in the values:
```bash
cp backend/.env.example backend/.env
# Edit backend/.env with your local database URL and generated JWT secret
```
