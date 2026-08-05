# deployment_guide.md — Local Setup (Phase 1)

> Phase 1 runs **locally only** (project decision). Cloud deployment is future work — a placeholder
> is left at the bottom of this file for when that becomes relevant (Phase 4 territory, per
> Rulebook/Working Book "production hardening"/"deployment").

## 1. Prerequisites

| Tool | Version (suggested) | Purpose |
|---|---|---|
| Python | 3.11+ | Backend |
| Node.js | 20+ | Frontend |
| PostgreSQL | 15+ | Database (local install or Docker) |
| Git | any recent | Version control |

## 2. Database setup

Option A — local PostgreSQL install:
```bash
createdb attendance_app_dev
```

Option B — Docker (simpler, recommended for two beginners so both machines match):
```bash
docker run --name attendance-db -e POSTGRES_PASSWORD=devpassword \
  -e POSTGRES_DB=attendance_app_dev -p 5432:5432 -d postgres:15
```

## 3. Backend setup

```bash
cd backend
python -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate
pip install -r requirements.txt

cp .env.example .env            # then fill in DATABASE_URL and JWT_SECRET_KEY
alembic upgrade head            # apply migrations
uvicorn app.main:app --reload --port 8000
```

`backend/.env` (not committed):
```
DATABASE_URL=postgresql://postgres:devpassword@localhost:5432/attendance_app_dev
JWT_SECRET_KEY=<generate a random string, e.g. `openssl rand -hex 32`>
JWT_EXPIRE_MINUTES=60
MEDIA_ROOT=./media
```

Backend now serves at `http://localhost:8000`, with interactive docs at
`http://localhost:8000/docs` (cross-check against `API_contract.md`).

## 4. Frontend setup

```bash
cd frontend
npm install
cp .env.example .env.local       # set VITE_API_BASE_URL
npm run dev
```

`frontend/.env.local` (not committed):
```
VITE_API_BASE_URL=http://localhost:8000
```

Frontend now serves at `http://localhost:5173` (Vite default).

## 5. Seed data (recommended for local testing)

Create one admin and one member account for manual testing before writing seed scripts:
```bash
cd backend
python -m app.scripts.seed_dev_users   # create this script early — see enhancements.md if it doesn't exist yet
```
If this script doesn't exist yet, create it as one of the first backend tasks — every session of
manual testing otherwise starts from an empty database.

## 6. Running both together

Two terminal tabs: one running `uvicorn ...` in `backend/`, one running `npm run dev` in
`frontend/`. No process manager needed for Phase 1 — keep it simple.

## 7. Verifying the setup works

1. Open `http://localhost:5173`, log in as the seeded admin.
2. Create a session with a venue near your current location (use your real coordinates for
   realistic GPS testing, or a mock location tool during development).
3. Log in as the seeded member in a second browser/incognito window.
4. Check in; confirm the record appears in the admin's session detail view.
5. Export the Excel report; confirm it opens and contains the check-in.

This mirrors the "test a sample session with administrator and member accounts before production
use" step in Working Book §4.1.

## 8. Future: cloud deployment (not yet in scope)

Placeholder for Phase 4. When this becomes relevant, this section should cover: chosen host for
backend + Postgres (e.g. Render/Railway/Fly.io), chosen host for frontend (e.g. Vercel/Netlify),
environment variable management in that host, HTTPS, and backup strategy for the database
(Rulebook §11.3, §13). Do not build CI/CD or infra-as-code for this until the project reaches that
phase.


## 9. Phase 2 Local Development

Phase 2 continues to use the same local development environment established during Phase 1.

No additional infrastructure is required. Existing backend, frontend and PostgreSQL setup
procedures remain unchanged.

## 10. Database Migration Workflow

During Phase 2, new database tables and schema changes will be introduced incrementally.

Recommended workflow:

```bash
# Create a migration
alembic revision --autogenerate -m "describe change"

# Review generated migration

# Apply migration
alembic upgrade head
```

Never edit an already applied migration unless the database is recreated for development.

---

## 11. Backend Development Verification

Before pushing backend changes:

- [ ] Backend starts successfully.
- [ ] Database migrations execute successfully.
- [ ] FastAPI documentation loads (`/docs`).
- [ ] New endpoints respond correctly.
- [ ] Existing endpoints continue working.
- [ ] No unexpected server errors appear.

---

## 12. Frontend Development Verification

Before pushing frontend changes:

- [ ] Frontend starts successfully.
- [ ] Application communicates with the backend.
- [ ] No console errors exist.
- [ ] UI behaves correctly.
- [ ] Loading and error states are verified.
- [ ] Existing functionality remains unaffected.

---

## 13. Full Integration Verification

Before considering a milestone complete:

1. Start PostgreSQL.
2. Start the FastAPI backend.
3. Start the React frontend.
4. Verify authentication.
5. Verify current milestone functionality.
6. Verify previous milestone functionality.
7. Confirm no regressions were introduced.

Every milestone should leave the application in a runnable state.

---

## 14. Future Deployment

Local-first development remains the project strategy until Phase 4.

Cloud deployment, CI/CD, production infrastructure, backups and deployment automation remain
outside the scope of Phase 2 and will be introduced during the Production Hardening phase.

---

# 15. Production Environment

Phase 4 introduces production deployment guidance for the InnoTech Hub Attendance System.

The deployment architecture should remain provider-agnostic, allowing the application to be deployed using any supported infrastructure that satisfies the project's operational requirements.

Typical deployment architecture includes:

- Backend application
- Frontend application
- PostgreSQL database
- Object/file storage where applicable
- HTTPS-enabled reverse proxy
- Backup storage

Deployment providers may change over time without affecting the application architecture.

---

# 16. Backend Deployment

The backend should be deployed as an independent service.

Deployment requirements include:

- Production Python environment
- Installed project dependencies
- Environment variables
- Database connectivity
- HTTPS support
- Migration execution before application startup

Before exposing the backend publicly:

- Apply all pending database migrations.
- Verify application startup.
- Verify API documentation loads successfully where enabled.
- Verify authentication endpoints.
- Verify production logging.

The backend must never start against an outdated database schema.

---

# 17. Frontend Deployment

The frontend should be deployed independently of the backend.

Deployment requirements include:

- Production build generation
- Correct backend API URL
- HTTPS configuration
- Environment variable configuration

Before deployment:

```bash
npm install
npm run build
```

Verify the generated production build before publishing.

Frontend deployments should reference the production backend rather than local development services.

---

# 18. Production Configuration

Production environments should use dedicated configuration values.

Typical production configuration includes:

- Database connection string
- JWT secret
- Token expiration settings
- Allowed CORS origins
- Media storage location
- Logging configuration

Production secrets must never be committed to version control.

Development and production environments should remain completely independent.

---

# 19. Database Backup & Recovery

Production deployments must support reliable backup and recovery procedures.

Recommended practices include:

- Scheduled database backups
- Manual backup capability
- Automatic backup verification
- Administrator-controlled restoration
- Recovery logging

All backup operations should follow the documented backend backup workflow introduced during Phase 4.

Recovery procedures should be verified periodically.

---

# 20. Deployment Verification

Every production deployment should be verified before being considered complete.

Recommended verification checklist:

- [ ] Backend service is running.
- [ ] Frontend is accessible.
- [ ] HTTPS is functioning correctly.
- [ ] Database connectivity is verified.
- [ ] Database migrations completed successfully.
- [ ] Authentication works correctly.
- [ ] Attendance workflows function correctly.
- [ ] Presence Monitoring functions correctly.
- [ ] Activity Management functions correctly.
- [ ] Offline synchronization functions correctly.
- [ ] Audit & Security features function correctly.
- [ ] Backup Management functions correctly.
- [ ] Existing functionality remains unaffected.

Production deployment should only be considered successful after all verification steps pass.

---

# 21. Monitoring & Maintenance

Production deployments should include operational monitoring.

Recommended monitoring areas include:

- Application availability
- Backend logs
- Frontend errors
- Database health
- Synchronization failures
- Backup execution
- Backup verification
- Security events

Monitoring solutions remain deployment-provider independent.

The chosen monitoring platform may change without affecting application functionality.

---

# 22. Rollback Strategy

If a deployment introduces unexpected issues:

1. Stop further deployment activities.
2. Restore the previous stable application version.
3. Verify application availability.
4. Verify database integrity.
5. Investigate the deployment issue.
6. Correct the issue.
7. Repeat deployment verification before redeployment.

Rollback procedures should prioritize restoring service availability while preserving application data.

Production deployments should always allow recovery to the previous stable release.

---

# 23. Phase 4 Production Readiness Checklist

Before considering the system production-ready, verify:

- [ ] Production environment is configured.
- [ ] HTTPS is enabled.
- [ ] Environment variables are configured securely.
- [ ] Database migrations are current.
- [ ] Backup scheduling is configured.
- [ ] Backup verification succeeds.
- [ ] Recovery procedure has been tested.
- [ ] Offline synchronization is verified.
- [ ] Audit & Security functionality is verified.
- [ ] Existing application functionality remains unaffected.
- [ ] Documentation is current.
- [ ] All relevant tests pass.