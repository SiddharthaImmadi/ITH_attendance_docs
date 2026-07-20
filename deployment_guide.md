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
