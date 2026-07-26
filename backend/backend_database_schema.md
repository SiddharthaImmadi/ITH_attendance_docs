# backend_database_schema.md — PostgreSQL Schema & Migrations

> Phase 1 database design: tables, relationships, constraints, and migration strategy using Alembic.

## 1. Entity Relationship Diagram

```
┌─────────────────────┐
│      users          │
├─────────────────────┤
│ id (PK, UUID)       │
│ full_name           │
│ email (UNIQUE)      │
│ password_hash       │
│ role (enum)         │
│ created_at          │
└──────────┬──────────┘
           │
           │ created_by (FK)
           │
           ▼
┌─────────────────────┐         ┌──────────────────────┐
│    sessions         │◄────────┤ attendance_records   │
├─────────────────────┤         ├──────────────────────┤
│ id (PK, UUID)       │ 1:N     │ id (PK, UUID)        │
│ title               │         │ session_id (FK)      │
│ purpose             │         │ member_id (FK)       │
│ date                │         │ check_in_time        │
│ start_time          │         │ check_in_lat         │
│ end_time            │         │ check_in_lng         │
│ grace_period_min    │         │ distance_meters      │
│ venue_lat           │         │ gps_accuracy_meters  │
│ venue_lng           │         │ photo_reference      │
│ radius_meters       │         │ final_status (enum)  │
│ status (enum)       │         │ rejection_reason     │
│ created_by (FK)     │         │ created_at           │
│ created_at          │         │ UNIQUE(session_id,   │
└─────────────────────┘         │        member_id)    │
                                └──────────────────────┘
```

## 2. SQL DDL (Create Table Statements)

```sql
-- Enable UUID extension
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";

-- Users table
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    full_name VARCHAR(255) NOT NULL,
    email VARCHAR(255) NOT NULL UNIQUE,
    password_hash VARCHAR(255) NOT NULL,
    role VARCHAR(50) NOT NULL CHECK (role IN ('admin', 'member')),
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_users_email ON users(email);

-- Sessions table
CREATE TABLE sessions (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    title VARCHAR(255) NOT NULL,
    purpose TEXT,
    date DATE NOT NULL,
    start_time TIMESTAMPTZ NOT NULL,
    end_time TIMESTAMPTZ NOT NULL,
    grace_period_minutes INTEGER NOT NULL DEFAULT 10,
    venue_lat DOUBLE PRECISION NOT NULL,
    venue_lng DOUBLE PRECISION NOT NULL,
    radius_meters INTEGER NOT NULL CHECK (radius_meters > 0),
    status VARCHAR(50) NOT NULL DEFAULT 'scheduled' 
        CHECK (status IN ('scheduled', 'open', 'closed')),
    created_by UUID NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    FOREIGN KEY (created_by) REFERENCES users(id) ON DELETE RESTRICT
);

CREATE INDEX idx_sessions_created_by ON sessions(created_by);
CREATE INDEX idx_sessions_status ON sessions(status);

-- Attendance records table
CREATE TABLE attendance_records (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    session_id UUID NOT NULL,
    member_id UUID NOT NULL,
    check_in_time TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    check_in_lat DOUBLE PRECISION NOT NULL,
    check_in_lng DOUBLE PRECISION NOT NULL,
    distance_meters DOUBLE PRECISION NOT NULL,
    gps_accuracy_meters DOUBLE PRECISION,
    photo_reference VARCHAR(500) NOT NULL,
    final_status VARCHAR(50) NOT NULL 
        CHECK (final_status IN ('present', 'late', 'pending_verification', 'rejected')),
    rejection_reason VARCHAR(255),
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    FOREIGN KEY (session_id) REFERENCES sessions(id) ON DELETE CASCADE,
    FOREIGN KEY (member_id) REFERENCES users(id) ON DELETE CASCADE,
    UNIQUE(session_id, member_id)
);

CREATE INDEX idx_attendance_records_session_id ON attendance_records(session_id);
CREATE INDEX idx_attendance_records_member_id ON attendance_records(member_id);
CREATE INDEX idx_attendance_records_final_status ON attendance_records(final_status);
```

## 3. Table Descriptions

### users
| Column | Type | Notes |
|---|---|---|
| `id` | UUID | Primary key, auto-generated |
| `full_name` | VARCHAR(255) | User's display name |
| `email` | VARCHAR(255) | Unique, used for login |
| `password_hash` | VARCHAR(255) | bcrypt/argon2 hash, never plaintext |
| `role` | VARCHAR(50) enum | `admin` or `member` |
| `created_at` | TIMESTAMPTZ | Account creation time, server time |

**Constraints:**
- Email must be unique (prevents duplicate accounts)
- Role is enum (database enforces valid values)

### sessions
| Column | Type | Notes |
|---|---|---|
| `id` | UUID | Primary key |
| `title` | VARCHAR(255) | Session name (e.g., "Workshop Attendance") |
| `purpose` | TEXT | Optional description |
| `date` | DATE | Session date in YYYY-MM-DD |
| `start_time` | TIMESTAMPTZ | When check-ins open (UTC) |
| `end_time` | TIMESTAMPTZ | When session ends |
| `grace_period_minutes` | INTEGER | Minutes after start_time before marked "late" |
| `venue_lat` | DOUBLE PRECISION | Venue latitude (decimal degrees) |
| `venue_lng` | DOUBLE PRECISION | Venue longitude |
| `radius_meters` | INTEGER | Allowed radius from venue |
| `status` | VARCHAR(50) enum | `scheduled`, `open`, or `closed` |
| `created_by` | UUID | Foreign key to `users.id` (admin who created) |
| `created_at` | TIMESTAMPTZ | Server time, creation timestamp |

**Constraints:**
- `radius_meters > 0` (enforced at DB level)
- `created_by` references `users` table
- `start_time` must be before `end_time` (enforced in application layer, not DB)

### attendance_records
| Column | Type | Notes |
|---|---|---|
| `id` | UUID | Primary key |
| `session_id` | UUID | Foreign key to session |
| `member_id` | UUID | Foreign key to user (who checked in) |
| `check_in_time` | TIMESTAMPTZ | Server time of check-in |
| `check_in_lat` | DOUBLE PRECISION | GPS latitude at check-in |
| `check_in_lng` | DOUBLE PRECISION | GPS longitude |
| `distance_meters` | DOUBLE PRECISION | Computed distance from venue |
| `gps_accuracy_meters` | DOUBLE PRECISION | GPS accuracy reported by device |
| `photo_reference` | VARCHAR(500) | Path to stored photo (e.g., `media/2026/08/01/uuid.jpg`) |
| `final_status` | VARCHAR(50) enum | `present`, `late`, `pending_verification`, `rejected` |
| `rejection_reason` | VARCHAR(255) | Why it was rejected (if applicable) |
| `created_at` | TIMESTAMPTZ | Record creation time |

**Constraints:**
- **UNIQUE(session_id, member_id)** — prevents duplicate check-ins from same member in same session (enforced at DB level)
- `final_status` is enum (DB enforces valid values)
- Cascading delete on both FKs (if session deleted, all its attendance records deleted)

## 4. Migration Strategy (Alembic)

**Initial setup:**
```bash
alembic init alembic
```

**For each schema change:**
```bash
alembic revision --autogenerate -m "add users table"
alembic upgrade head
```

**Migration file structure:**
```python
# alembic/versions/0001_initial_schema.py
def upgrade():
    op.create_table('users', ...)
    op.create_index(...)

def downgrade():
    op.drop_table('users')
    op.drop_index(...)
```

**Version files are numbered and dated:**
- `001_initial_schema.py`
- `002_add_sessions_table.py`
- `003_add_attendance_records_table.py`

**In production, run migrations on deploy:**
```bash
alembic upgrade head
```

## 5. Phase 1 Data Volume Expectations

| Table | Rows | Notes |
|---|---|---|
| users | 10–100 | Admin(s) + test members |
| sessions | 5–20 | Created during testing |
| attendance_records | 20–200 | 4 check-ins per session × 5–50 sessions |

**Storage:** <1 MB without photos. Photos stored separately on disk.

## 6. Indexes

Created for performance:
- `idx_users_email` — fast login lookup
- `idx_sessions_created_by` — list admin's sessions quickly
- `idx_sessions_status` — filter by status quickly
- `idx_attendance_records_session_id` — list check-ins for a session
- `idx_attendance_records_member_id` — list member's history
- `idx_attendance_records_final_status` — filter by status (for reports)

UNIQUE constraint on `(session_id, member_id)` also acts as an index.

## 7. Timezone Handling

- All times stored as `TIMESTAMPTZ` (timezone-aware)
- All times are in UTC internally (server time)
- Frontend converts to local time for display (client-side responsibility)
- Example: `2026-08-01T09:00:00Z` = 9 AM UTC

## 8. Backup & Recovery

- PostgreSQL WAL (Write-Ahead Logging) enabled for durability
- Regular dumps: `pg_dump attendance_app_dev > backup.sql`
- Test restore: `psql < backup.sql` (as part of deployment verification)


## 9. Phase 2 Schema Extensions

> Phase 2 extends the existing schema to support attendance lifecycle management, presence monitoring, early check-out approval, and enhanced reporting.

---

## 9.1 Updated Entity Relationship Diagram

```
users
   │
   │
sessions
   │
   │
attendance_records
   │
   ├──────────────┐
   │              │
   ▼              ▼
presence_events   early_checkout_requests
```

Attendance records remain the primary entity.

Presence events and early check-out requests reference an attendance record.

---

## 9.2 attendance_records Enhancements

Extend the existing table with additional lifecycle fields.

| Column | Type | Notes |
|----------|------|------|
| check_out_time | TIMESTAMPTZ | Server timestamp of check-out |
| attendance_duration | INTEGER | Duration in minutes |
| attendance_state | VARCHAR(50) | Current attendance lifecycle state |

Possible attendance states:

- active
- pending_approval
- approved
- rejected
- completed

---

## 9.3 presence_events Table

Stores chronological attendance activity.

| Column | Type | Notes |
|----------|------|------|
| id | UUID | Primary key |
| attendance_id | UUID | FK → attendance_records.id |
| event_type | VARCHAR(50) | entered, left, returned, checkout |
| latitude | DOUBLE PRECISION | Recorded latitude |
| longitude | DOUBLE PRECISION | Recorded longitude |
| recorded_at | TIMESTAMPTZ | Server timestamp |

Indexes:

- attendance_id
- recorded_at

---

## 9.4 early_checkout_requests Table

Stores administrator approval workflow.

| Column | Type | Notes |
|----------|------|------|
| id | UUID | Primary key |
| attendance_id | UUID | FK → attendance_records.id |
| reason | TEXT | Member explanation |
| approval_status | VARCHAR(50) | pending, approved, rejected |
| approval_remark | TEXT | Administrator comments |
| approved_by | UUID | FK → users.id |
| approved_at | TIMESTAMPTZ | Decision timestamp |
| created_at | TIMESTAMPTZ | Request timestamp |

Indexes:

- attendance_id
- approval_status
- approved_by

---

## 9.5 Relationships

```
attendance_records

1

↓

N

presence_events
```

```
attendance_records

1

↓

0..1

early_checkout_requests
```

One attendance record may contain multiple presence events.

An attendance record may have at most one early check-out request.

---

## 9.6 Constraints

Attendance lifecycle should enforce:

- one attendance record per member per session;
- one early check-out request per attendance record;
- chronological presence events;
- valid attendance state transitions.

Business rules remain enforced by the service layer.

---

## 9.7 Additional Indexes

Recommended indexes:

```
attendance_state

approval_status

recorded_at

attendance_id

approved_by
```

These improve monitoring and reporting performance.

---

## 9.8 Migration Strategy

Create a dedicated Phase 2 migration.

Example:

```
004_phase2_attendance_extensions.py
```

Migration responsibilities:

- extend attendance_records;
- create presence_events;
- create early_checkout_requests;
- create indexes;
- create foreign keys.

Existing Phase 1 data should remain valid after migration.

---

## 9.9 Expected Data Growth

| Table | Expected Growth |
|----------|----------------|
| attendance_records | Moderate |
| presence_events | High |
| early_checkout_requests | Low |

Presence events are expected to become the largest table.

Indexes should be maintained accordingly.

---

## 9.10 Design Principles

The database should continue to follow these principles:

- Normalize related data.
- Preserve referential integrity.
- Keep attendance_records as the primary attendance entity.
- Store chronological activity separately.
- Support efficient reporting and monitoring.
- Ensure all schema changes are delivered through Alembic migrations.