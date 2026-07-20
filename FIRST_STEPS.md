# FIRST_STEPS.md — Your First Week as Frontend Developer

> A concrete, day-by-day checklist. Do these in order. You don't have to be perfect — you're
> learning as you go.

## Day 1: Understanding the project

**Time:** 1–2 hours

- [ ] Read `context.md` (5 min) — understand what the app is, who's building it, why
- [ ] Read `PRD.md` (10 min) — Phase 1 scope: login, sessions, check-in, report export. That's it.
- [ ] Read `system_architecture.md` (15 min) — tech stack (React/TS/Tailwind/shadcn, FastAPI, PostgreSQL)
- [ ] Read `API_contract.md` (5 min) — it's just an index now
- [ ] Read `rules_frontend.md` (10 min) — how you should write code, conventions, "premium feel" guideline
- [ ] Read `COLLABORATION_GUIDE.md` (15 min) — this file — how you'll work with backend dev
- [ ] Skim `development_roadmap.md` (5 min) — Phase 1 has 5 milestones, you're starting on Milestone 0

**Outcome:** You understand what you're building and how you'll work together.

---

## Day 2: Setting up your machine

**Time:** 2–3 hours

- [ ] Clone Repo B (the private dev repo with code)
  ```bash
  git clone <repo-b-url> attendance-app-dev
  cd attendance-app-dev
  git checkout frontend        # switch to frontend branch (don't work on main)
  ```

- [ ] Follow `deployment_guide.md` to set up your local environment
  - [ ] Install Node.js 20+ (check: `node --version`)
  - [ ] Install Python 3.11+ (check: `python --version`)
  - [ ] Install PostgreSQL 15+ (local or Docker)
  - [ ] Run backend setup (copy from guide)
  - [ ] Run frontend setup (copy from guide)

- [ ] Verify both are running
  - Backend: `http://localhost:8000/docs` (should show FastAPI docs)
  - Frontend: `http://localhost:5173` (should show a blank page or "Hello world")

- [ ] Send your first standup message to backend dev:
  ```
  [Siri, Frontend] — [date]
  ✅ Done: Set up dev environment, both frontend and backend running locally
  🔨 Working on: Scaffolding login page
  🚧 Blocker: None
  ```

**Outcome:** Your machine is ready. You have a running frontend and backend, even if they're
empty.

---

## Day 3: Build the login page (with mocks)

**Time:** 3–4 hours

Read before starting: `api-contract-01-admin-login.json` and `api-contract-02-member-login.json`

- [ ] Create `src/lib/api.ts` — a function that calls `/auth/login`
  - [ ] For now, use **mocks** (return fake data after a 200ms delay)
  - [ ] See COLLABORATION_GUIDE.md §6 for the mock pattern

- [ ] Create `src/pages/LoginPage.tsx`
  - [ ] Email and password input fields
  - [ ] "Login" button
  - [ ] Form validation with Zod (email must be valid format, password min 8 chars)
  - [ ] Show loading state while submit is in progress
  - [ ] Show error message if login fails
  - [ ] On success, store JWT in localStorage, redirect to dashboard

- [ ] Add `src/lib/auth.ts` — JWT helpers
  - [ ] `getToken()` — read from localStorage
  - [ ] `setToken(jwt)` — save to localStorage
  - [ ] `clearToken()` — delete from localStorage

- [ ] Style with Tailwind + shadcn components (Button, Input, Card)
  - [ ] Make it look intentional, not placeholder (see `rules_frontend.md` on "premium feel")

- [ ] Test locally
  - [ ] Type in email/password
  - [ ] Click login
  - [ ] See loading spinner
  - [ ] Get redirected (to a placeholder dashboard for now)
  - [ ] Refresh page — you should still be logged in (JWT in localStorage)

- [ ] Commit
  ```bash
  git add .
  git commit -m "feat(auth): login form with mock JWT storage"
  git push origin frontend
  ```

**Outcome:** Working login UI that stores a token. No real backend integration yet, but the shape
is right.

---

## Day 4: Open your first PR

**Time:** 1 hour

- [ ] Go to the repo (GitHub/GitLab)
- [ ] Click "New Pull Request"
- [ ] Set it up: `frontend` branch → `main` branch
- [ ] Title: `feat(auth): login form with JWT storage and validation`
- [ ] Description:
  ```
  Implements login page for both admin and member roles.

  - Email/password form with Zod validation
  - Mock JWT storage in localStorage
  - Loading and error states
  - Tested manually on localhost:5173

  Uses: api-contract-01-admin-login.json, api-contract-02-member-login.json
  ```
- [ ] Assign backend dev as reviewer
- [ ] Assign yourself too (so you get notified of comments)
- [ ] Click "Create Pull Request"

**What happens next:** Backend dev reviews. They might ask questions or approve. Meanwhile, you
can start on the next thing — don't wait.

---

## Day 5: Get current user endpoint, finish Milestone 1

**Time:** 2–3 hours

Read: `api-contract-03-get-current-user.json`

- [ ] Add to `src/lib/api.ts`
  - [ ] `getMe()` function that calls `GET /me` with the JWT
  - [ ] Returns user profile (id, name, email, role)
  - [ ] Use mock first (return a fake admin object)

- [ ] Create `src/pages/Dashboard.tsx` (placeholder)
  - [ ] Show "Welcome, [name]!"
  - [ ] Show role (Admin or Member)
  - [ ] Logout button (clears token, redirects to login)

- [ ] Create `src/lib/hooks/useAuth.ts` — a custom React hook
  - [ ] On page load, call `getMe()` to restore session
  - [ ] If no token or getMe fails, user is logged out
  - [ ] Use this in your app layout to show login if logged out, dashboard if logged in

- [ ] Create `src/pages/AdminDashboard.tsx` and `src/pages/MemberDashboard.tsx`
  - [ ] Both super simple for now (just text saying "Admin" or "Member")
  - [ ] Router redirects based on `user.role`

- [ ] Test end-to-end
  - [ ] Log in → see dashboard (admin or member based on mock data)
  - [ ] Refresh → still logged in (session restored from localStorage)
  - [ ] Logout button works → back to login

- [ ] Update `progress.md`
  ```
  ## Session: 2026-07-21 — Siri (Frontend)

  **Milestone:** Milestone 1 — Auth (complete)

  **What got done:**
  - Login form with validation
  - JWT storage and session restore
  - Role-based dashboard routing
  - Tested end-to-end with mocks

  **What's next:**
  - Wait for backend login endpoint to be live (so we can test real creds)
  - Then Milestone 2 — Session creation

  **Blockers:** None yet
  ```

- [ ] Commit and PR
  ```bash
  git add .
  git commit -m "feat(auth): add session restore and role-based dashboard routing"
  git push origin frontend
  ```

**Outcome:** Milestone 1 (Auth) is done on the frontend side. You're waiting for the backend to
have `/auth/login` live so you can test the real integration.

---

## Day 6–7: Start Milestone 2, use mocks while backend catches up

**Time:** 2–3 hours per day

Read: `api-contract-04-session-creation.json` and `api-contract-05-session-list.json`

By now, the backend dev is probably working on sessions. While they build, you build the UI:

- [ ] Create `src/pages/AdminSessionsPage.tsx`
  - [ ] List of sessions (mock data for now: title, status, date)
  - [ ] "Create New Session" button

- [ ] Create `src/pages/CreateSessionPage.tsx`
  - [ ] Form: title, purpose, date, start time, end time, grace period minutes, venue lat/lng, radius
  - [ ] Validate with Zod
  - [ ] On submit, call mock `createSession()` → returns a session object with id
  - [ ] Redirect to session detail
  - [ ] Show loading/error states

- [ ] Create `src/pages/SessionDetailPage.tsx`
  - [ ] Show session info
  - [ ] Show list of who checked in (mock: just empty list for now)
  - [ ] "Export Excel" button (disabled for now, just shows a placeholder)

- [ ] Test with mocks
  - [ ] Create a session → see it in the list
  - [ ] Click on it → see details
  - [ ] Everything feels smooth (no janky loading, clear errors if it fails)

**Meanwhile:** Check your standup with backend dev — are they close to `/sessions` endpoints?

**Outcome:** Milestone 2 frontend is mostly done. When backend has endpoints live, you flip the
mock switch and test the real thing.

---

## What to avoid in Week 1

❌ Don't write all the code without testing it — test as you go.

❌ Don't skip the PR process. Even if it's "just a login form," PR it. Code review catches bugs
early.

❌ Don't ignore the API contract. If you're not 100% sure what the response looks like, ask
before coding.

❌ Don't let yourself get stuck for more than 1 hour without asking backend dev for help. That's
what the daily standup is for.

❌ Don't commit directly to `main`. Always use your `frontend` branch.

---

## Measuring your progress

At the end of Week 1, you should have:

✅ Milestone 0 (environment setup) — done
✅ Milestone 1 (auth/login/session restore) — done, PRd, waiting for code review
✅ Milestone 2 (sessions) — started, mostly done with mocks, waiting for backend endpoints

That's success. You don't need to be perfect; you just need to be moving forward and talking to
the backend dev every day.
