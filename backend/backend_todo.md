# Backend Development To-Do List — Phase 1

## Milestone 0: Environment & Scaffolding

- [ ] Create FastAPI project structure (app/, alembic/, tests/, requirements.txt)
- [ ] Set up Python virtual environment
- [ ] Install core dependencies (fastapi, uvicorn, sqlalchemy, psycopg2, alembic, pydantic, pytest, etc.)
- [ ] Create .env.example with all required variables
- [ ] Set up environment variable loading (pydantic-settings)
- [ ] Configure PostgreSQL database connection
- [ ] Initialize Alembic for migrations
- [ ] Create app/main.py with FastAPI app initialization
- [ ] Add CORS middleware for frontend (localhost:5173)
- [ ] Verify backend runs on localhost:8000
- [ ] Test /docs endpoint shows OpenAPI interface

## Milestone 1: Authentication (Login & Session Management)

### Database Setup
- [ ] Create users table migration (id, full_name, email, password_hash, role, created_at)
- [ ] Run migration to create users table
- [ ] Add unique constraint on email
- [ ] Create index on email for fast login lookups

### Core Authentication Services
- [ ] Implement password hashing with bcrypt/argon2 (app/core/security.py)
- [ ] Create hash_password() function
- [ ] Create verify_password() function
- [ ] Generate JWT_SECRET_KEY (document in .env.example)
- [ ] Implement create_access_token() function with expiration
- [ ] Implement verify_token() function for JWT validation
- [ ] Create oauth2_scheme using FastAPI security

### Login Endpoint (POST /auth/login)
- [ ] Create AuthService class in app/services/auth_service.py
- [ ] Implement login() method (query user by email, verify password, generate JWT)
- [ ] Create Pydantic schemas for request/response (LoginRequest, LoginResponse, UserRead)
- [ ] Create route handler in app/api/auth.py
- [ ] Handle error cases (INVALID_CREDENTIALS)
- [ ] Return JWT token + user profile on success
- [ ] Test with admin user (seeded manually)
- [ ] Test with member user (seeded manually)
- [ ] Verify response matches API_contract.md

### Get Current User Endpoint (GET /me)
- [ ] Create get_current_user dependency (validates JWT, fetches user from DB)
- [ ] Create route handler in app/api/auth.py
- [ ] Return current user profile
- [ ] Handle 401 error (invalid/expired token)
- [ ] Test with valid and expired tokens

### Seed Data
- [ ] Create app/scripts/seed_dev_users.py script
- [ ] Generate one admin user (email: admin@innotech-hub.com, password: AdminPass123!)
- [ ] Generate one member user (email: member@innotech-hub.com, password: MemberPass123!)
- [ ] Document how to run seed script

## Milestone 2: Session Management

### Database Setup
- [ ] Create sessions table migration (id, title, purpose, date, start_time, end_time, grace_period_minutes, venue_lat, venue_lng, radius_meters, status, created_by, created_at)
- [ ] Add foreign key constraint on created_by (references users.id)
- [ ] Add unique constraint on (session_id, member_id) for attendance_records (placeholder for Milestone 3)
- [ ] Create indexes on created_by, status
- [ ] Run migrations

### Session Service (app/services/sessions_service.py)
- [ ] Implement SessionsService class with dependency injection (AsyncSession db)
- [ ] Create create() method (validate time range, create record, return)
- [ ] Create get_by_id() method (with ownership check for admin)
- [ ] Create list_for_user() method (admin gets all their sessions, member gets only open)
- [ ] Create get_detail() method (fetch session + nested attendance_records)
- [ ] Create close() method (verify not already closed, update status)
- [ ] Add validation: end_time must be after start_time
- [ ] Raise HTTPException with proper error codes

### Session API Routes (app/api/sessions.py)
- [ ] Create Pydantic schemas (SessionCreate, SessionRead, SessionDetail)
- [ ] POST /sessions endpoint (admin only)
  - [ ] Verify admin role
  - [ ] Call service.create()
  - [ ] Return 201 with session object
- [ ] GET /sessions endpoint (role-scoped list)
  - [ ] Admin gets all their sessions
  - [ ] Member gets only open sessions
  - [ ] Return list of session summaries
- [ ] GET /sessions/{id} endpoint (admin only)
  - [ ] Verify admin role
  - [ ] Call service.get_detail()
  - [ ] Return session with nested attendance records
  - [ ] Handle 404 (not found or not owned)
- [ ] PATCH /sessions/{id}/close endpoint (admin only)
  - [ ] Verify admin role
  - [ ] Call service.close()
  - [ ] Return 200 with updated session
  - [ ] Handle 409 (already closed)
  - [ ] Handle 404 (not found or not owned)

### Testing
- [ ] Unit test session service (create, get, close, list)
- [ ] Integration test all session endpoints
- [ ] Test admin can only see their own sessions
- [ ] Test member sees only open sessions
- [ ] Test duplicate session creation (allowed, different admins can have same title)

## Milestone 3: Attendance & Check-in

### Database Setup
- [ ] Create attendance_records table migration (id, session_id, member_id, check_in_time, check_in_lat, check_in_lng, distance_meters, gps_accuracy_meters, photo_reference, final_status, rejection_reason, created_at)
- [ ] Add foreign keys on session_id and member_id (cascade delete)
- [ ] Add unique constraint on (session_id, member_id) to prevent duplicates
- [ ] Create indexes on session_id, member_id, final_status
- [ ] Run migrations

### Geo Service (app/services/geo_service.py)
- [ ] Implement GeoService class
- [ ] Create haversine() method (calculate distance between two lat/lng points)
- [ ] Test with known coordinate pairs (distance calculation accuracy)

### Attendance Service (app/services/attendance_service.py)
- [ ] Implement AttendanceService class with dependency injection
- [ ] Create check_in() method with full validation:
  - [ ] Verify session exists and status is open
  - [ ] Check for duplicate check-in (same member, same session)
  - [ ] Verify time window (check-in time is during session window)
  - [ ] Calculate distance using Haversine
  - [ ] Determine final_status based on distance and time:
    - [ ] Inside radius + on-time = "present"
    - [ ] Inside radius + late (after grace period) = "late"
    - [ ] Outside radius + acceptable accuracy = reject with OUTSIDE_RADIUS error
    - [ ] Poor GPS accuracy (>30m) = "pending_verification" (not an error)
  - [ ] Create AttendanceRecord in DB
  - [ ] Return record
- [ ] Create get_member_history() method (fetch all check-ins for member, ordered by recent first)
- [ ] Add proper error handling with HTTPException

### File Storage (Photo Evidence)
- [ ] Create media directory structure (media/YYYY/MM/DD/)
- [ ] Implement photo file saving (save to disk with UUID filename)
- [ ] Store photo_reference path in database (not binary)
- [ ] Ensure photo_reference is returned in API response

### Check-in API Routes (app/api/attendance.py)
- [ ] Create Pydantic schemas (CheckInRequest, AttendanceRecordRead, etc.)
- [ ] POST /attendance/check-in endpoint (member only)
  - [ ] Parse multipart form-data (session_id, lat, lng, gps_accuracy_meters, photo)
  - [ ] Verify member role
  - [ ] Validate photo (JPEG/PNG only, max 5MB)
  - [ ] Save photo to disk
  - [ ] Call service.check_in()
  - [ ] Return 201 with record
  - [ ] Handle error cases:
    - [ ] 422 OUTSIDE_RADIUS (distance > radius + tolerance)
    - [ ] 422 SESSION_NOT_OPEN (session not in open status or time window)
    - [ ] 409 DUPLICATE_CHECK_IN (member already checked in to this session)
    - [ ] 400 INVALID_PHOTO_TYPE (not JPEG/PNG)
    - [ ] 400 PHOTO_TOO_LARGE (>5MB)
    - [ ] 201 with pending_verification (GPS accuracy too low)
- [ ] GET /attendance/history endpoint (member only)
  - [ ] Fetch all check-ins for current member
  - [ ] Enrich with session titles
  - [ ] Return list ordered by recent first

### Testing
- [ ] Unit test geo service (Haversine distance)
- [ ] Unit test attendance service (all validation paths)
- [ ] Integration test check-in endpoint (success, outside radius, duplicate, poor accuracy)
- [ ] Test photo file handling (save, verify size limits, mime type)
- [ ] Test member can only access their own history

## Milestone 4: Reporting

### Database Setup
- [ ] Ensure attendance_records table is set up (from Milestone 3)

### Reports Service (app/services/reports_service.py)
- [ ] Implement ReportsService class
- [ ] Create generate_attendance_xlsx() method:
  - [ ] Fetch session (verify admin ownership)
  - [ ] Fetch all attendance records for session
  - [ ] Create workbook with openpyxl
  - [ ] Sheet 1 (Attendance Summary): columns = Member ID, Name, Session, Date, Check-in Time, Distance (m), Final Status
  - [ ] Sheet 2 (Photo Evidence): columns = Member ID, Session ID, Photo Reference, Capture Time
  - [ ] Return workbook object

### Reports API Routes (app/api/reports.py)
- [ ] Create GET /reports/attendance.xlsx endpoint (admin only)
  - [ ] Parse query parameter session_id
  - [ ] Verify admin role
  - [ ] Call service.generate_attendance_xlsx()
  - [ ] Return binary file response with correct content-type
  - [ ] Handle 404 (session not found or not owned by admin)
  - [ ] Handle 403 (non-admin attempted export)

### Testing
- [ ] Test report generation (verify sheet names, columns, data)
- [ ] Test file download (verify content-type, filename)
- [ ] Test admin can only export their own session's reports

## Milestone 5: Error Handling & Polish

### Global Error Handling
- [ ] Implement global exception handler in app/main.py
- [ ] Return standard error envelope (error object with code, message, details)
- [ ] Handle all error codes from API_contract.md (INVALID_CREDENTIALS, OUTSIDE_RADIUS, etc.)
- [ ] Log errors appropriately (no sensitive data in logs)
- [ ] Return generic error messages (don't leak internal details)

### Input Validation
- [ ] Ensure all routes validate input with Pydantic schemas
- [ ] Server-side validation of business rules (not trusting client)
- [ ] Validate radius > 0
- [ ] Validate end_time > start_time
- [ ] Validate email format
- [ ] Validate password strength (if enforced)

### Database Constraints
- [ ] Verify unique constraint on (session_id, member_id) prevents duplicates at DB level
- [ ] Verify foreign key constraints cascade correctly
- [ ] Verify indexes are created (email, session created_by, etc.)

### Logging & Monitoring
- [ ] Configure logging to file and console
- [ ] Log authentication events (login success/failure)
- [ ] Log business-critical events (check-in success, session close)
- [ ] Log errors with appropriate severity

### Documentation
- [ ] Verify all endpoints match API_contract.md exactly
- [ ] Document how to run migrations
- [ ] Document how to seed test users
- [ ] Document environment variables required
- [ ] Update progress.md with completion status
- [ ] Add entry to changelog.md for Phase 1 completion

## Integration Testing (Before Merge)

- [ ] Test full login flow (valid credentials, invalid credentials)
- [ ] Test session creation (admin only, validate time range)
- [ ] Test session list (admin sees theirs, member sees open)
- [ ] Test session detail (admin can view, member cannot)
- [ ] Test check-in flow end-to-end (inside radius, outside radius, duplicate, poor accuracy)
- [ ] Test attendance history (member sees their records)
- [ ] Test Excel export (admin only, correct sheet structure)
- [ ] Test 401 unauthorized (invalid/expired token on protected endpoints)
- [ ] Test 403 forbidden (member trying to access admin endpoint)
- [ ] Test 404 not found (accessing non-existent session)
- [ ] Test 422 validation errors (bad input)
- [ ] Test concurrent check-ins (verify duplicate constraint catches race conditions)
- [ ] Test database transactions (verify rollback on errors)

## Performance & Security

- [ ] Verify passwords are never logged or returned
- [ ] Verify JWT_SECRET_KEY is loaded from environment
- [ ] Verify passwords are hashed (bcrypt/argon2)
- [ ] Verify all timestamps use server time (not client time)
- [ ] Verify database connection pooling is configured
- [ ] Verify rate limiting is considered (for future, not Phase 1)
- [ ] Verify SQL injection is prevented (SQLAlchemy parameterized queries)
- [ ] Load test with expected user count

## Deployment Readiness

- [ ] Verify all environment variables are documented (.env.example)
- [ ] Verify migrations run successfully (alembic upgrade head)
- [ ] Verify backend starts without errors
- [ ] Verify CORS is configured correctly
- [ ] Verify health check endpoint works
- [ ] Test with both SQLite (dev) and PostgreSQL (production-like)

# Backend Development To-Do List — Phase 2

## Milestone 6: Attendance Lifecycle

### Database Updates

- [ ] Create migration for Phase 2 attendance enhancements
- [ ] Add check_out_time to attendance_records
- [ ] Add attendance_duration field
- [ ] Add attendance_state field
- [ ] Add early_checkout_reason field
- [ ] Add approval_status field
- [ ] Add approval_remark field
- [ ] Add approved_by foreign key
- [ ] Add approved_at timestamp
- [ ] Create indexes for approval_status
- [ ] Run migrations

### Attendance Service

- [ ] Extend AttendanceService for attendance lifecycle
- [ ] Implement get_active_session()
- [ ] Implement get_attendance_status()
- [ ] Implement complete_check_out()
- [ ] Calculate attendance duration
- [ ] Generate attendance summary
- [ ] Validate attendance state transitions
- [ ] Prevent duplicate check-out
- [ ] Prevent invalid state changes

### Attendance API

- [ ] Create GET /attendance/active endpoint
- [ ] Create POST /attendance/check-out endpoint
- [ ] Create GET /attendance/summary endpoint
- [ ] Verify member authorization
- [ ] Return standardized response models
- [ ] Handle invalid attendance states

### Testing

- [ ] Test complete attendance lifecycle
- [ ] Test attendance duration calculation
- [ ] Test attendance summary generation
- [ ] Test invalid attendance transitions

---

## Milestone 7: Presence Monitoring

### Presence Tracking

- [ ] Create PresenceMonitoringService
- [ ] Detect inside/outside venue
- [ ] Record presence events
- [ ] Calculate current presence status
- [ ] Prevent duplicate presence events
- [ ] Maintain chronological event order

### Presence Timeline

- [ ] Create presence_events table
- [ ] Create SQLAlchemy model
- [ ] Create Alembic migration
- [ ] Create Pydantic schemas
- [ ] Create timeline retrieval service
- [ ] Return ordered timeline events

### Presence API

- [ ] Create GET /attendance/timeline endpoint
- [ ] Return chronological events
- [ ] Validate member ownership
- [ ] Validate administrator access

### Testing

- [ ] Test presence event creation
- [ ] Test timeline ordering
- [ ] Test presence calculations
- [ ] Test authorization

---

## Milestone 8: Early Check-out Workflow

### Database

- [ ] Create early check-out request model
- [ ] Create migration
- [ ] Add foreign key relationships
- [ ] Add indexes

### Service Layer

- [ ] Implement request_early_checkout()
- [ ] Validate active attendance
- [ ] Validate request reason
- [ ] Prevent duplicate requests
- [ ] Implement approve_request()
- [ ] Implement reject_request()
- [ ] Notify attendance service of decisions

### API

- [ ] Create POST /attendance/early-checkout
- [ ] Create GET /attendance/approval-status
- [ ] Create POST /attendance/approve
- [ ] Create POST /attendance/reject

### Validation

- [ ] Require approval reason
- [ ] Restrict approval to administrators
- [ ] Prevent approval of completed attendance
- [ ] Prevent duplicate approvals

### Testing

- [ ] Test approval workflow
- [ ] Test rejection workflow
- [ ] Test invalid requests
- [ ] Test permission enforcement

---

## Milestone 9: Administrator Monitoring

### Monitoring Service

- [ ] Create MonitoringService
- [ ] Calculate live attendance statistics
- [ ] Fetch active members
- [ ] Fetch members outside venue
- [ ] Fetch completed attendance
- [ ] Fetch pending approvals

### API

- [ ] Create GET /monitoring/live
- [ ] Create GET /monitoring/statistics
- [ ] Create GET /monitoring/pending-requests

### Performance

- [ ] Optimize monitoring queries
- [ ] Eliminate N+1 queries
- [ ] Add required indexes
- [ ] Optimize joins

### Testing

- [ ] Test monitoring endpoints
- [ ] Test administrator permissions
- [ ] Test large attendance datasets

---

## Milestone 10: Attendance Reporting

### Reporting Service

- [ ] Extend attendance reports
- [ ] Include attendance duration
- [ ] Include check-out time
- [ ] Include presence timeline
- [ ] Include approval information
- [ ] Include attendance summary

### Export

- [ ] Extend Excel export
- [ ] Include new Phase 2 fields
- [ ] Verify workbook formatting
- [ ] Validate generated reports

### Testing

- [ ] Test updated report generation
- [ ] Test exported workbook structure
- [ ] Test report accuracy

---

## Milestone 11: Backend Polish

### Error Handling

- [ ] Add Phase 2 error codes
- [ ] Standardize approval errors
- [ ] Standardize attendance lifecycle errors
- [ ] Improve validation responses

### Performance

- [ ] Profile attendance services
- [ ] Optimize timeline queries
- [ ] Optimize monitoring endpoints
- [ ] Review database indexes

### Security

- [ ] Verify administrator authorization
- [ ] Verify member ownership validation
- [ ] Verify approval security
- [ ] Review endpoint protection

---

## Phase 2 Integration Testing

- [ ] Test Active Attendance
- [ ] Test attendance lifecycle
- [ ] Test presence monitoring
- [ ] Test timeline generation
- [ ] Test early check-out request
- [ ] Test administrator approval
- [ ] Test administrator rejection
- [ ] Test attendance summary
- [ ] Test monitoring dashboard APIs
- [ ] Test reporting enhancements
- [ ] Test concurrent attendance operations

---

## Documentation

- [ ] Verify implementation matches API_contract.md
- [ ] Verify implementation follows rules_backend.md
- [ ] Update database documentation
- [ ] Update progress.md
- [ ] Add Phase 2 changelog entry

# Backend Development To-Do List — Phase 3

## Milestone 12: Activity Management

### Database

- [ ] Create activities table migration
- [ ] Add foreign key relationship to events
- [ ] Add created_by foreign key
- [ ] Add cancelled_by foreign key
- [ ] Create activity enums
- [ ] Create indexes
- [ ] Run migrations

### Activity Service

- [ ] Create ActivityService
- [ ] Implement create_activity()
- [ ] Implement update_activity()
- [ ] Implement publish_activity()
- [ ] Implement cancel_activity()
- [ ] Implement archive_activity()
- [ ] Validate activity lifecycle
- [ ] Prevent invalid state transitions

### API

- [ ] Create POST /activities
- [ ] Create GET /activities
- [ ] Create GET /activities/{id}
- [ ] Create PATCH /activities/{id}
- [ ] Create POST /activities/{id}/publish
- [ ] Create POST /activities/{id}/cancel
- [ ] Create POST /activities/{id}/archive

### Testing

- [ ] Test activity creation
- [ ] Test activity publication
- [ ] Test activity cancellation
- [ ] Test activity archival
- [ ] Test activity retrieval

---

## Milestone 13: Activity Assignment

### Database

- [ ] Create activity_assignments table
- [ ] Create foreign key relationships
- [ ] Create unique constraint (activity_id, user_id)
- [ ] Create indexes
- [ ] Run migrations

### Service

- [ ] Create ActivityAssignmentService
- [ ] Implement assign_member()
- [ ] Implement remove_assignment()
- [ ] Implement get_assignments()
- [ ] Validate duplicate assignments
- [ ] Validate assignment conflicts

### API

- [ ] Create POST /activities/{id}/assign
- [ ] Create GET /activities/{id}/assignments
- [ ] Create GET /members/me/assignments

### Testing

- [ ] Test assignment creation
- [ ] Test duplicate prevention
- [ ] Test multiple volunteer assignments
- [ ] Test authorization

---

## Milestone 14: Activity Progress & Evidence

### Database

- [ ] Create activity_progress_updates table
- [ ] Create activity_evidence table
- [ ] Create indexes
- [ ] Run migrations

### Service

- [ ] Create ActivityProgressService
- [ ] Implement add_progress()
- [ ] Implement upload_photo()
- [ ] Implement upload_video()
- [ ] Implement submit_for_review()
- [ ] Validate evidence limits
- [ ] Optimize uploaded files

### File Storage

- [ ] Create media/activities/photos
- [ ] Create media/activities/videos
- [ ] Store metadata only in database
- [ ] Verify optimized file storage

### API

- [ ] Create POST /assignments/{id}/progress
- [ ] Create GET /assignments/{id}/progress
- [ ] Create POST /assignments/{id}/submit

### Testing

- [ ] Test progress creation
- [ ] Test evidence upload
- [ ] Test upload limits
- [ ] Test activity submission
- [ ] Test immutable evidence after submission

---

## Milestone 15: Activity Review

### Database

- [ ] Create activity_reviews table
- [ ] Create foreign keys
- [ ] Create indexes
- [ ] Run migrations

### Service

- [ ] Create ActivityReviewService
- [ ] Implement verify_activity()
- [ ] Implement request_changes()
- [ ] Implement get_pending_reviews()
- [ ] Implement get_review_history()

### API

- [ ] Create GET /activities/review/pending
- [ ] Create POST /reviews/{id}/verify
- [ ] Create POST /reviews/{id}/needs-changes

### Validation

- [ ] Require remarks for Needs Changes
- [ ] Preserve review history
- [ ] Prevent invalid review states

### Testing

- [ ] Test verification
- [ ] Test needs changes
- [ ] Test review history
- [ ] Test authorization

---

## Milestone 16: Activity Templates

### Database

- [ ] Create activity_templates table
- [ ] Create activity_template_items table
- [ ] Create indexes
- [ ] Run migrations

### Service

- [ ] Create ActivityTemplateService
- [ ] Implement create_template()
- [ ] Implement update_template()
- [ ] Implement apply_template()
- [ ] Implement get_templates()

### API

- [ ] Create POST /activity-templates
- [ ] Create GET /activity-templates
- [ ] Create PATCH /activity-templates/{id}
- [ ] Create POST /activity-templates/{id}/apply

### Validation

- [ ] Validate template structure
- [ ] Generate independent activities
- [ ] Preserve existing activities

### Testing

- [ ] Test template creation
- [ ] Test template updates
- [ ] Test template application
- [ ] Test generated activities

---

## Milestone 17: Activity Reporting

### Reporting Service

- [ ] Extend activity reports
- [ ] Include assignments
- [ ] Include progress history
- [ ] Include evidence references
- [ ] Include review status
- [ ] Include reviewer remarks

### Export

- [ ] Extend Excel export
- [ ] Verify workbook formatting
- [ ] Validate generated reports

### Testing

- [ ] Test report generation
- [ ] Test report accuracy
- [ ] Test exported workbook structure

---

## Milestone 18: Backend Polish

### Error Handling

- [ ] Add Phase 3 error codes
- [ ] Standardize activity errors
- [ ] Standardize assignment errors
- [ ] Standardize review errors
- [ ] Improve validation responses

### Performance

- [ ] Optimize activity queries
- [ ] Optimize assignment queries
- [ ] Optimize review queries
- [ ] Review database indexes
- [ ] Verify lazy loading where appropriate

### Security

- [ ] Verify administrator authorization
- [ ] Verify volunteer ownership
- [ ] Verify review permissions
- [ ] Review endpoint protection

---

## Phase 3 Integration Testing

- [ ] Test complete activity lifecycle
- [ ] Test multiple volunteer assignments
- [ ] Test progress updates
- [ ] Test evidence uploads
- [ ] Test review workflow
- [ ] Test template generation
- [ ] Test reporting enhancements
- [ ] Test concurrent assignments
- [ ] Test concurrent submissions
- [ ] Test concurrent reviews

---

## Documentation

- [ ] Verify implementation matches API_contract.md
- [ ] Verify implementation follows rules_backend.md
- [ ] Verify database schema matches implementation
- [ ] Verify architecture documentation
- [ ] Verify services documentation
- [ ] Update backend_testing.md
- [ ] Update progress.md
- [ ] Add Phase 3 changelog entry

---

## Milestone 19: Final Verification & Release Readiness

### Documentation Verification

- [ ] Verify implementation matches PRD.md
- [ ] Verify implementation matches API_contract.md
- [ ] Verify implementation matches backend_database_schema.md
- [ ] Verify implementation matches backend_architecture.md
- [ ] Verify implementation matches backend_services.md
- [ ] Verify implementation matches backend_api_implementation.md
- [ ] Verify implementation matches backend_testing.md
- [ ] Verify implementation follows rules_backend.md

### Database Verification

- [ ] Verify all migrations execute successfully
- [ ] Verify foreign key relationships
- [ ] Verify indexes are created
- [ ] Verify enum definitions
- [ ] Verify constraints
- [ ] Verify rollback migrations

### API Verification

- [ ] Verify every endpoint matches API_contract.md
- [ ] Verify request models
- [ ] Verify response models
- [ ] Verify HTTP status codes
- [ ] Verify error responses
- [ ] Verify authentication and authorization

### Business Rule Verification

- [ ] Verify activity lifecycle
- [ ] Verify assignment workflow
- [ ] Verify progress update workflow
- [ ] Verify evidence validation
- [ ] Verify review workflow
- [ ] Verify template generation
- [ ] Verify reporting workflow

### Testing Verification

- [ ] Run complete unit test suite
- [ ] Run complete integration test suite
- [ ] Verify coverage target is achieved
- [ ] Verify performance benchmarks
- [ ] Verify concurrency scenarios
- [ ] Verify error handling scenarios

### Security Verification

- [ ] Verify administrator permissions
- [ ] Verify volunteer permissions
- [ ] Verify ownership validation
- [ ] Verify protected endpoints
- [ ] Verify file upload validation
- [ ] Verify input validation

### Performance Verification

- [ ] Review database query performance
- [ ] Verify index usage
- [ ] Verify lazy loading where appropriate
- [ ] Verify evidence optimization
- [ ] Verify report generation performance

### Frontend Integration Readiness

- [ ] Verify API documentation is complete
- [ ] Verify OpenAPI documentation
- [ ] Verify frontend contract compatibility
- [ ] Verify response consistency
- [ ] Verify pagination
- [ ] Verify filtering and sorting

### Repository Verification

- [ ] Cross-check implementation with Repo A documentation
- [ ] Verify API contracts have not been modified
- [ ] Verify backward compatibility
- [ ] Review changelog
- [ ] Update progress.md
- [ ] Prepare implementation for frontend integration

### Release Checklist

- [ ] All milestones completed
- [ ] All documentation updated
- [ ] All tests passing
- [ ] No known critical defects
- [ ] Ready for frontend development
- [ ] Ready for system integration

# Backend Development To-Do List — Phase 4

## Milestone 20: Offline Synchronization

### Database

- [ ] Verify synchronization uses existing business tables
- [ ] Confirm no offline queue is stored in PostgreSQL
- [ ] Create migration updates if synchronization metadata is required
- [ ] Verify database consistency during synchronization

### Synchronization Service

- [ ] Create SynchronizationService
- [ ] Implement process_sync()
- [ ] Implement validate_queue()
- [ ] Implement replay_operations()
- [ ] Implement resolve_conflicts()
- [ ] Preserve operation ordering
- [ ] Generate synchronization results
- [ ] Prevent duplicate synchronized operations

### API

- [ ] Create synchronization endpoint
- [ ] Accept complete synchronization queue
- [ ] Validate synchronization payload
- [ ] Return synchronization result summary
- [ ] Handle synchronization conflicts
- [ ] Handle partial synchronization failures

### Validation

- [ ] Validate synchronization payload integrity
- [ ] Validate authenticated user
- [ ] Validate operation ordering
- [ ] Validate duplicate operations
- [ ] Preserve business validation rules

### Testing

- [ ] Test successful synchronization
- [ ] Test partial failures
- [ ] Test conflict resolution
- [ ] Test duplicate operations
- [ ] Test queue ordering
- [ ] Test large synchronization batches

---

## Milestone 21: Audit Logging

### Database

- [ ] Extend audit_logs table
- [ ] Add previous_values
- [ ] Add new_values
- [ ] Add retention configuration
- [ ] Create required indexes
- [ ] Run migrations

### Audit Service

- [ ] Create AuditLogService
- [ ] Implement record_event()
- [ ] Implement get_audit_history()
- [ ] Implement get_entity_history()
- [ ] Preserve immutable audit records

### Integration

- [ ] Integrate authentication audit logging
- [ ] Integrate attendance audit logging
- [ ] Integrate activity audit logging
- [ ] Integrate administrator audit logging
- [ ] Integrate backup audit logging

### Validation

- [ ] Verify immutable audit history
- [ ] Verify before and after values
- [ ] Verify actor information
- [ ] Verify timestamps

### Testing

- [ ] Test audit record creation
- [ ] Test immutable audit history
- [ ] Test before and after value recording
- [ ] Test audit retrieval
- [ ] Test administrator access

---

## Milestone 22: Spoofing Detection

### Database

- [ ] Create spoofing_events table
- [ ] Create indexes
- [ ] Run migrations

### Spoofing Detection Service

- [ ] Create SpoofingDetectionService
- [ ] Implement evaluate_location()
- [ ] Implement detect_mock_location()
- [ ] Implement detect_impossible_travel()
- [ ] Implement evaluate_device()
- [ ] Implement generate_assessment()

### Security

- [ ] Record suspicious events
- [ ] Calculate repeated suspicious behaviour
- [ ] Notify administrators
- [ ] Preserve spoofing event history

### Validation

- [ ] Validate device profile
- [ ] Validate location consistency
- [ ] Validate configurable spoofing threshold
- [ ] Generate detailed spoofing assessment

### Testing

- [ ] Test mock GPS detection
- [ ] Test impossible travel detection
- [ ] Test abnormal movement detection
- [ ] Test repeated suspicious behaviour
- [ ] Test administrator notifications

---

## Milestone 23: Backup & Recovery

### Database

- [ ] Create backup_metadata table
- [ ] Create indexes
- [ ] Run migrations

### Backup Service

- [ ] Create BackupService
- [ ] Implement manual backup execution
- [ ] Implement scheduled backup execution
- [ ] Implement verify_backup()
- [ ] Implement get_backup_status()
- [ ] Persist backup metadata

### Recovery

- [ ] Verify backup integrity
- [ ] Preserve previously verified backups
- [ ] Handle verification failures
- [ ] Notify administrators

### Validation

- [ ] Validate backup completion
- [ ] Validate backup metadata
- [ ] Validate automatic backup verification

### Testing

- [ ] Test manual backup execution
- [ ] Test scheduled backup execution
- [ ] Test backup verification
- [ ] Test backup failure handling
- [ ] Test backup metadata persistence

---

## Milestone 24: Production Hardening

### Authentication Security

- [ ] Implement login rate limiting
- [ ] Implement temporary account locking
- [ ] Implement administrator account unlock
- [ ] Implement administrator JWT revocation
- [ ] Revoke all active user sessions

### Background Processing

- [ ] Create BackgroundProcessingService
- [ ] Implement Immediate Background Tasks
- [ ] Implement Scheduled Background Tasks
- [ ] Configure background job scheduler
- [ ] Verify task isolation

### Environment Configuration

- [ ] Add Phase 4 environment variables
- [ ] Configure synchronization settings
- [ ] Configure backup settings
- [ ] Configure spoofing thresholds
- [ ] Configure audit retention

### Performance

- [ ] Review synchronization performance
- [ ] Optimize audit queries
- [ ] Optimize spoofing evaluation
- [ ] Optimize scheduled background jobs

### Security

- [ ] Verify administrator authorization
- [ ] Verify protected production endpoints
- [ ] Verify audit log protection
- [ ] Verify backup protection

### Testing

- [ ] Test login rate limiting
- [ ] Test session revocation
- [ ] Test scheduled background jobs
- [ ] Test background processing
- [ ] Test production configuration

---

## Milestone 25: Final Verification & Production Readiness

### Documentation Verification

- [ ] Verify implementation matches PRD.md
- [ ] Verify implementation matches API_contract.md
- [ ] Verify implementation matches backend_database_schema.md
- [ ] Verify implementation matches backend_architecture.md
- [ ] Verify implementation matches backend_services.md
- [ ] Verify implementation matches backend_api_implementation.md
- [ ] Verify implementation matches backend_authentication.md
- [ ] Verify implementation matches backend_environment.md
- [ ] Verify implementation matches backend_testing.md
- [ ] Verify implementation follows rules_backend.md

### Production Verification

- [ ] Verify offline synchronization
- [ ] Verify audit logging
- [ ] Verify spoofing detection
- [ ] Verify backup execution
- [ ] Verify backup verification
- [ ] Verify production configuration

### Security Verification

- [ ] Verify login rate limiting
- [ ] Verify administrator session revocation
- [ ] Verify administrator permissions
- [ ] Verify member permissions
- [ ] Verify protected production endpoints

### Testing Verification

- [ ] Run complete unit test suite
- [ ] Run complete integration test suite
- [ ] Verify 95% backend coverage target
- [ ] Verify performance benchmarks
- [ ] Verify recovery scenarios

### Repository Verification

- [ ] Cross-check implementation with Repository A documentation
- [ ] Verify API contracts have not been modified
- [ ] Verify backward compatibility
- [ ] Update progress.md
- [ ] Add Phase 4 changelog entry

### Production Readiness Checklist

- [ ] All Phase 4 milestones completed
- [ ] All documentation updated
- [ ] All tests passing
- [ ] Backup verification successful
- [ ] Production configuration validated
- [ ] Ready for deployment