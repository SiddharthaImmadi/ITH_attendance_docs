# COLLABORATION_GUIDE.md — Frontend & Backend Coordination

> For the two developers (and their agents) to stay synchronized without stepping on each other's
> toes. This is **your operational playbook** — read it top to bottom once, then use it as a
> reference whenever you're unsure how to proceed.

## 1. Your role as Frontend Developer

**You own:** Everything the user sees and clicks on (React UI, forms, buttons, pages, camera
capture, GPS flow, loading states, error messages). You depend on the backend API existing and
matching the contract.

**You don't own:** The database, business logic on the server side, GPS validation, photo
validation, final status calculation. Those all happen server-side and you call them via the API.

## 2. How to contact the Backend Developer

Set up **one shared communication channel** and check it daily (Slack, Discord, WhatsApp, or
even a shared Google Doc called "Standup"). Every day, 1-2 messages max:

### Template: "Daily Standup" message
```
[Your name, Frontend] — [date]
✅ Done today: [what you shipped or finished]
🔨 Working on now: [current task]
🚧 Blocker / question: [if any]
```

### Example
```
[Siri, Frontend] — Jul 20
✅ Done: Scaffolded login page, wired form validation
🔨 Working on: Integrating /auth/login endpoint (need to see response shape once it's live)
🚧 Need: Backend dev to confirm response format matches api-contract-01 — does the JWT come back as `access_token` or `token`?
```

**Why this format?** Backend dev can see in 10 seconds what you're doing, if you're blocked, and
what you need from them. No long emails.

## 3. How to show progress (the `progress.md` file)

Update `progress.md` at the **end of every session** — this is your official progress log.

### Template entry
```markdown
## Session: [date] — [your name]

**Milestone working on:** [e.g., "Milestone 1 — Auth"]

**What got done:**
- Completed X
- Tested Y with Postman/[tool]
- Updated Z docs

**What's next:**
- Start on A
- Needs backend: [if any]

**Blockers:**
- None / [specific blocker + what's needed to unblock]
```

### Real example
```markdown
## Session: 2026-07-21 — Siri (Frontend)

**Milestone working on:** Milestone 1 — Auth (login page)

**What got done:**
- Built login form with email/password fields
- Added form validation (Zod schema)
- Wired up API client to `/auth/login` endpoint
- Tested with Postman — confirmed JWT response shape matches api-contract-01-admin-login.json
- Stored JWT in localStorage and added loading/error states

**What's next:**
- Build `/me` endpoint integration to auto-restore session on page reload
- Create role-based routing (if role === admin, go to admin dashboard; else member dashboard)

**Blockers:**
- None — backend endpoint is live and working
```

**Why?** When the backend dev reads this, they know exactly what you've done, what you're
about to do, and if you're waiting on them. Backend dev does the same thing on their end, so you
can both see the full picture in one doc.

**Don't wait for the backend to be done to start.** You can code against the API contract as if
the endpoint exists — use mock responses in the frontend (Postman, fetch mock, or MSW) and swap
them for real once the backend is ready. See §6 below.

## 4. The Git workflow (no committing directly to `main`)

### You work on the `frontend` branch

```bash
git clone <repo-b-url>
cd attendance-app-dev
git checkout frontend         # your working branch
git pull origin frontend      # stay up to date
# ... write code ...
git add .
git commit -m "feat(login): add login form with validation"
git push origin frontend
```

### When a feature is done, open a Pull Request (PR) into `main`

1. Go to the repo on GitHub/GitLab.
2. Click "New Pull Request" from `frontend` into `main`.
3. **Title:** `feat(scope): description`, e.g. `feat(auth): login form + session restore`
4. **Description:** Include
   - What you built
   - Link to the relevant `api-contract-*.json` file(s) you're using
   - Any tests you added
   - Any issues you hit and how you resolved them
5. **Assign the backend developer as a reviewer.**

### The backend developer reviews and either:

- ✅ Approves → merge to `main`
- 💬 Asks questions → you respond, push more commits to the PR
- ❌ Suggests changes → discuss in the PR comments, then update

### After merge to `main`:
```bash
git checkout main
git pull origin main
git checkout frontend
git merge main              # bring main's changes back into your branch
```

This keeps your `frontend` branch fresh for the next feature.

## 5. The API Contract is your north star

**Before you write a line of UI code**, read the relevant JSON file(s):

- Building login? → Read `api-contract-01-admin-login.json` and `api-contract-02-member-login.json`
- Building check-in? → Read `api-contract-08-gps-checkin.json` and `api-contract-09-live-photo.json`

**If the contract seems unclear or wrong:**

1. Add a comment in your daily standup: "Does the login response actually include `expires_in_minutes`?"
2. Backend dev confirms or fixes the JSON.
3. You code against the updated contract.

**Never guess at the shape of the response.** The JSON files exist so you don't have to.

## 6. Building before the backend is ready (mocking)

The backend is slower to start (database setup, migrations, validation logic). You don't have to
wait idle. Use **mock responses**:

```typescript
// src/lib/api.ts — mock mode
const API_URL = process.env.VITE_API_BASE_URL;
const USE_MOCKS = process.env.VITE_USE_MOCKS === "true";

export async function login(email: string, password: string) {
  if (USE_MOCKS) {
    // Fake a 200ms delay to feel real
    await new Promise(r => setTimeout(r, 200));
    return {
      access_token: "fake-jwt-token-for-testing",
      token_type: "bearer",
      expires_in_minutes: 60,
      user: {
        id: "fake-admin-id",
        full_name: "B. Sankar",
        email,
        role: "admin"
      }
    };
  }
  // Real API call
  const response = await fetch(`${API_URL}/auth/login`, { ... });
  return response.json();
}
```

In `.env.local`:
```
VITE_USE_MOCKS=true            # set to false once backend is live
VITE_API_BASE_URL=http://localhost:8000
```

**Benefit:** You can finish the entire login UI (form, validation, loading state, error
handling) while the backend is still being built. Once the backend endpoint is ready, flip the
mock flag to false and test against the real thing. No rewriting.

## 7. Daily sync checklist (do this every day)

- ☐ Read backend dev's standup message → understand what they finished
- ☐ Check if any new API contract files were merged to `main` → merge them into `frontend`
- ☐ Write your standup message → send it
- ☐ Push your code to `frontend` (even if incomplete, as long as it doesn't break things)
- ☐ Update `progress.md` at the end of the day

## 8. Common scenarios & what to do

### Scenario: Backend dev says "Login endpoint is live"
1. Merge `main` into your `frontend` branch.
2. Flip `USE_MOCKS` to false.
3. Test your login form against the real endpoint.
4. Report back: "Works" or "Got error X, is that expected?"

### Scenario: You're blocked on something the backend needs to build
1. Add it to your standup: "🚧 Can't start check-in UI until GPS validation is done."
2. Backend dev sees it and can prioritize accordingly.
3. In the meantime, work on a different milestone.

### Scenario: You notice the API contract is wrong or unclear
1. Don't guess. Ask in the standup: "api-contract-08 says `gps_accuracy_meters` — is this a
   number or a string?"
2. Wait for clarification.
3. Once confirmed, code against the correct version.

### Scenario: Your code depends on something from backend (e.g., session list) but they're behind
1. Use mocks or hardcoded test data for now.
2. Leave a TODO: `// TODO: replace with real API once /sessions endpoint is ready`
3. When they're done, find the TODO and swap it out.

## 9. Code review process (when you PR into `main`)

The backend dev reviews your PR. They're looking for:

- ☐ Does the code match the API contract shapes?
- ☐ Are you handling errors correctly (using the error codes from api-contract-12)?
- ☐ Is the loading/error/success flow there?
- ☐ Any obvious bugs or typos?

**You can also ask them questions in the PR:** "Does it matter if I validate email on the client,
or should I always trust the server's validation?" They'll answer right in the PR comments.

## 10. How to structure your commit messages

Use this format (same as backend dev):

```
type(scope): short description

Optional longer explanation if needed.
```

**Types:** `feat` (new feature), `fix` (bug fix), `chore` (tooling, setup), `docs` (doc update)

**Scope:** which part of the app — `auth`, `login`, `checkin`, `report`, `ui`, etc.

**Examples:**
- `feat(auth): implement JWT token storage in localStorage`
- `fix(login): clear form after successful login`
- `chore(setup): add react-query and msw for mocking`
- `docs(checkin): add notes on live camera capture permission flow`

## 11. When to sync in real-time (not just daily)

Message the backend dev **immediately** (not in standup) if:

- ❌ You found a critical bug in their code (they should fix ASAP)
- 🚨 The API response shape is completely different from the contract
- ⚠️ You need them to merge something to `main` RIGHT NOW so you can unblock
- ❓ You have a quick clarification question (5-min back-and-forth, not a standup item)

Everything else can wait for the daily standup message.

## 12. Tools you'll use

| Tool | Purpose | How to use |
|---|---|---|
| Git (`frontend` branch) | Version control, track your code changes | Commit daily, PR when done |
| Postman / Insomnia | Test backend endpoints before wiring them into React | Make requests to `localhost:8000`, check responses match contract |
| MSW (Mock Service Worker) or fetch mocks | Mock API responses locally | Use while waiting for backend, remove mocks once real endpoint is live |
| React DevTools | Debug component state | Chrome extension, inspect your component props/state |
| `progress.md` | Shared status log | Update at end of every session |
| Daily standup message | Quick sync with backend dev | Write one message per day, 30 seconds max |

## 13. What success looks like

By the end of a milestone (e.g., Milestone 1 — Auth):

- ✅ Your frontend code is on the `frontend` branch
- ✅ It's been PRd and merged into `main` (code reviewed)
- ✅ You tested it against a real backend endpoint (or mocks before backend was ready)
- ✅ You updated `progress.md` to show it's done
- ✅ Backend dev confirmed in their standup that the integration works from their side too
- ✅ Both of you agreed that Milestone 1 is "done done" — not just written, but tested

## 14. First week: expected timeline

**Day 1–2:** Read all docs, set up your dev environment, write "Hello world" React component.

**Day 3:** Start Milestone 1 — Auth. Build login form (with mocks).

**Day 4:** Wait for backend to have `/auth/login` live, test it, finish Milestone 1 with real endpoint.

**Day 5:** Start Milestone 2 — Sessions. Backend might still be building `/sessions`, so use mocks.

This way you're always moving forward, never fully blocked.

---

**Questions?** Ask in your daily standup message. This guide exists so you don't feel lost — use it.
