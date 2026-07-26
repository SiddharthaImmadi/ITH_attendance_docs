# GitHub Push Rules & Team Workflow

> Rules for pushing code as a 2-person team (1 frontend, 1 backend developer). Follow these to avoid conflicts, lost work, and merge chaos.

## 1. Branch Structure (Never Push Directly to Main)

```
main                    — Integration branch (production-ready, always working)
├── frontend             — Frontend developer's long-lived branch
└── backend              — Backend developer's long-lived branch
```

**Golden Rule:** You have ONE long-lived branch. Push ONLY to your branch, NEVER to main.

### Branch Responsibilities
- **frontend branch** — Contains all frontend code (React, Tailwind, components, pages)
- **backend branch** — Contains all backend code (FastAPI, services, routes, migrations)
- **main branch** — Only merged code that BOTH sides agree works end-to-end. Locked for direct pushes.

---

## 2. Commit Message Convention

All commits must follow this format. No exceptions.

```
type(scope): short description

Longer explanation (optional, max 72 chars per line)
```

### Type
Use ONLY these types:
- `feat` — New feature (new component, new endpoint, new service)
- `fix` — Bug fix
- `chore` — Setup, tooling, dependencies (no feature/business logic change)
- `docs` — Documentation only (no code change)
- `refactor` — Restructure code (no logic change, no feature)
- `test` — Test-only changes

### Scope
Keep it short (1-3 words):
- Frontend: `auth`, `login`, `sessions`, `checkin`, `ui`, `forms`, `routing`, `state`
- Backend: `auth`, `sessions`, `attendance`, `reports`, `db`, `migrations`, `validation`, `services`

### Examples
```
feat(auth): implement JWT token storage in localStorage
fix(checkin): correct distance calculation for edge case
docs(api): update endpoint documentation
chore(setup): install react-query dependency
test(attendance): add unit tests for haversine formula
```

### Bad Examples (Don't Do This)
```
fix stuff                           ✗ No type/scope
feat: added login                   ✗ Lowercase, vague scope
Update main                         ✗ No type/scope
🎉 New awesome feature 🚀           ✗ No conventional format
```

---

## 3. Daily Workflow: The Push Checklist

Before every push, do this:

```
□ I am on the correct branch (frontend or backend)
□ I ran my code locally and tested it
□ I ran linting/formatter (if applicable)
□ All tests pass (if I wrote any)
□ I updated progress.md with what I finished
□ My commit message follows the convention
□ I wrote a meaningful commit message (not "wip" or "fix")
□ I am NOT pushing to main
```

---

## 4. Push to Your Branch (Daily)

Push your work to YOUR branch every day, even if incomplete.

```bash
# Frontend developer
git checkout frontend
git add .
git commit -m "type(scope): description"
git push origin frontend

# Backend developer
git checkout backend
git add .
git commit -m "type(scope): description"
git push origin backend
```

**Why daily?** Backup + context for the other person + easy to see what you're doing.

---

## 5. Pull Request Process (When a Milestone is Done)

When you finish a milestone (e.g., Milestone 1: Auth), open a PR into main.

### Step 1: Sync With Main First
```bash
git checkout main
git pull origin main
git checkout your_branch
git merge main
# Resolve any conflicts
```

### Step 2: Create a Pull Request
Go to GitHub → Pull Requests → New Pull Request.

**From:** your_branch (frontend or backend)
**Into:** main

### Step 3: PR Title & Description

**Title format:**
```
feat(frontend): complete Milestone 1 — Auth & Session Restore
```

**Description template (copy-paste this):**
```
## Milestone Completed
Milestone 1 — Authentication

## What's New
- [ ] Admin login with email/password
- [ ] JWT token storage and session restore
- [ ] Role-based routing (admin → /admin, member → /member)
- [ ] Logout functionality

## Tests
- [x] Tested login with valid credentials
- [x] Tested invalid password error
- [x] Tested session restore on page reload
- [x] Tested logout clears token

## Relevant Docs
- See `API_contract.md` § Authentication
- See `rules_frontend.md` (or `rules_backend.md`)

## Checklist
- [x] Code follows project conventions
- [x] I tested this locally
- [x] No console errors or warnings
- [x] Updated progress.md
- [x] Updated changelog.md (if applicable)

## Other Notes
Any blockers, questions, or things the other dev should know about this milestone.
```

### Step 4: Request Code Review From Other Developer
Assign the other developer as reviewer. In the GitHub PR interface, add them.

**Frontend developer:** Assign backend developer
**Backend developer:** Assign frontend developer

### Step 5: Review Checklist (For the Reviewer)

When the other person opens a PR, review it:

```
□ Does this match what's in the milestone checklist?
□ Are there any obvious bugs or logic issues?
□ Did they update progress.md and changelog.md?
□ Does it follow the coding conventions (rules_frontend.md or rules_backend.md)?
□ Is the commit history clean (no 100 WIP commits)?
□ Can I understand what changed by reading the description?
□ Do we need to align on API contract changes?
```

If good → Approve and merge.
If issues → Leave comments, request changes, author fixes and re-requests review.

### Step 6: Merge to Main
Once approved, the author merges to main (GitHub button or command line):

```bash
git checkout main
git merge your_branch
git push origin main
```

After merge, pull the latest main locally:
```bash
git checkout main
git pull origin main
```

---

## 6. Handling API Contract Changes

The API contract (API_contract.md) is the contract between frontend and backend.

**Rule: Never change the contract without telling the other person first.**

### If You Need to Change the Contract

1. **Update API_contract.md on main** (not on your branch)
2. **Open an issue or Slack message** to the other dev: "I'm updating the POST /sessions response to include venue_label. See API_contract.md."
3. **Wait for acknowledgment** before either of you codes against the new shape
4. **Commit the change to main** with a clear message:
   ```
   docs(api): add venue_label field to POST /sessions response
   ```
5. **Both developers sync:** `git pull origin main`
6. **Then code against the new contract**

---

## 7. Handling Merge Conflicts

Conflicts happen when both people edit the same file. Here's how:

### Case 1: You Have Conflicts When Pulling

```bash
git status
# Shows "both modified: src/lib/api.ts"
```

**Open the file and look for conflict markers:**
```
<<<<<<< HEAD
// Your version
=======
// Their version
>>>>>>> origin/main
```

**Fix it by keeping one, both, or merging manually.** Then:
```bash
git add .
git commit -m "chore(merge): resolve conflicts from main"
git push origin your_branch
```

### Case 2: Conflict in PR (When Merging to Main)

GitHub will show "This branch has conflicts that must be resolved before merging."

**Option A: Resolve on Command Line**
```bash
git checkout main
git pull origin main
git checkout your_branch
git merge main
# Fix conflicts
git push origin your_branch
# Refresh GitHub PR — conflicts resolved
```

**Option B: Communicate with Other Dev**
If you're not sure which version to keep, ask the other person. One of you might have the "right" version.

### How to Avoid Conflicts
- Push daily (keeps branches fresh)
- Don't both edit the same file on the same day without telling each other
- Pull main into your branch frequently (at least before opening a PR)

---

## 8. What NOT to Do

❌ **Never push directly to main**
```bash
git push origin main  # WRONG
```

❌ **Never force-push to main**
```bash
git push -f origin main  # NEVER
```

❌ **Never commit to main if you're the backend developer**
```bash
git checkout main
git add .
git commit -m "quick fix"  # NO
```

❌ **Never skip the PR process** (even if it's "just one file")
Every change to main goes through a PR so the other person knows what's coming.

❌ **Never merge your own PR without the other person reviewing**
Self-merging defeats the purpose of code review.

❌ **Never rebase main after it's been pushed**
Other people are pulling from it. Rebasing rewrites history and breaks their branches.

---

## 9. Daily Communication Checklist

Every day, tell the other person:

```
[Your Name, Frontend/Backend] — [Date]

✅ Done today: [specific feature or fix]
🔨 Working on now: [current task]
🚧 Blocker/question: [if any]

Example:
[Siri, Frontend] — Jul 21
✅ Done: Completed login form with validation
🔨 Working on: Session list component
🚧 Blocker: Need to know if session.created_at is included in GET /sessions response
```

Use: Slack, Discord, daily standup message in the repo README, or a shared doc.

---

## 10. Syncing With Main Regularly

Don't wait until your PR is done to sync with main. Do it weekly (or whenever the other person merges).

```bash
git checkout your_branch
git pull origin main
# If conflicts, resolve and commit
git push origin your_branch
```

This keeps your branch fresh and prevents massive conflicts later.

---

## 11. GitHub Branch Protection (Optional, for Later)

If you want to enforce these rules technically (not just by convention):

Go to GitHub repo → Settings → Branches → Add rule for main:
- [ ] Require pull request reviews before merging (min 1 review)
- [ ] Dismiss stale pull request approvals
- [ ] Require status checks to pass before merging (if you set up CI)

This prevents accidental direct pushes to main.

---

## 12. Reverting a Mistake

If you pushed bad code to your branch:

```bash
# Option A: Amend the last commit (if not pushed yet)
git add .
git commit --amend --no-edit
git push origin your_branch -f

# Option B: Revert a pushed commit
git revert HEAD  # Creates a new commit that undoes the last one
git push origin your_branch

# Option C: Hard reset (only if no one has pulled your branch yet)
git reset --hard HEAD~1
git push origin your_branch -f
```

**For main:** Never force-push. Always use `git revert`.

---

## 13. Tagging Releases (After Phase 1 Complete)

After Phase 1 is done and merged to main:

```bash
git tag -a v0.1.0 -m "Phase 1 release: Login, Sessions, Check-in, Reports"
git push origin v0.1.0
```

Tags show up under GitHub Releases.

---

## 14. Code Review Etiquette

As the reviewer:
- Be respectful and constructive (this is their first time shipping a project)
- If you see a style issue (not a bug), use a suggestion comment instead of "request changes"
- If it's a logic bug, ask for clarification before assuming they're wrong
- Approve within 24 hours if possible (don't block them)

As the author:
- Don't take feedback personally — it's about the code
- If you disagree with a review comment, say so — discuss it
- Push fixes and re-request review (don't wait for the reviewer to re-check)

---

## 15. Milestone Completion Checklist (Before Opening PR)

Before you say "I'm done with this milestone," verify:

```
□ All tasks in frontend_todo.md (or backend_todo.md) for this milestone are checked
□ Code is tested locally
□ No console errors or warnings
□ Follows conventions in rules_frontend.md (or rules_backend.md)
□ Updated progress.md: marked milestone as complete
□ Updated changelog.md: added entry for this milestone
□ Commit history is clean (meaningful messages, no 50 WIP commits)
□ I've synced with main (pulled latest)
□ I've informed the other dev what's coming in the PR
□ I'm ready for code review
```

---

## Summary: The Simple Version

1. **You have one branch:** `frontend` or `backend`. Work there.
2. **Push daily:** Keep your branch fresh with daily commits.
3. **Follow commit conventions:** `type(scope): description`
4. **Open a PR when done:** Get the other person to review.
5. **Sync with main regularly:** Pull main into your branch weekly.
6. **Never force-push main:** Ever.
7. **Update docs:** progress.md, changelog.md when you finish.
8. **Communicate:** Tell the other person what you're doing.

---

## Quick Reference

```bash
# Daily workflow
git checkout your_branch
git add .
git commit -m "type(scope): description"
git push origin your_branch

# Sync with main (weekly or before PR)
git fetch origin
git merge origin/main
# Resolve conflicts if any
git push origin your_branch

# Open PR to main (when milestone done)
# — Use GitHub UI or: git push origin your_branch && open GitHub PR

# After PR approved, merge on GitHub UI
# Then sync locally: git checkout main && git pull origin main
```

That's it. Follow these rules, and the codebase stays clean and the team stays happy.

---

# 16. Phase 2 Development Workflow

Phase 2 marks the transition from project planning to active implementation.

The Git workflow established in this document remains unchanged. This section defines how the
development workflow integrates with the Phase 2 documentation process.

## Documentation Before Implementation

Before beginning a new feature:

- Read the relevant project documentation.
- Verify that the feature is within the current milestone.
- Ensure the implementation follows the documented architecture and API contract.
- Clarify documentation first if implementation requirements are ambiguous.

Implementation should never redefine project requirements.

---

## Feature Development Workflow

Every feature should follow the same lifecycle.

```
Documentation
      ↓
Planning
      ↓
Implementation
      ↓
Local Testing
      ↓
Commit
      ↓
Push to Personal Branch
      ↓
Code Review
      ↓
Merge to main
```

Avoid skipping intermediate steps, even for small changes.

---

## Documentation Synchronization

Whenever a completed implementation changes project documentation:

- Update `progress.md`
- Update `changelog.md` (if user-visible functionality changes)
- Update the relevant implementation TODO document
- Update documentation only when the implementation changes the documented behavior

Do not update documentation for code that has not yet been implemented.

---

## API-First Development

Backend and frontend development continue to work independently through the shared
`API_contract.md`.

If an API modification becomes necessary:

1. Review the proposed change.
2. Update the API contract.
3. Inform the other developer.
4. Synchronize both branches.
5. Begin implementation.

The API contract remains the single source of truth for communication between frontend and backend.

---

## Milestone Completion

A milestone is considered complete only when:

- Implementation is complete.
- Local testing has been completed.
- Documentation has been updated.
- The feature has been reviewed.
- The Pull Request has been approved.
- The changes have been merged into `main`.

Completing code alone does not complete a milestone.

---

## Phase 2 Goal

The objective of Phase 2 is to incrementally build the Attendance & Activity Tracking Application
while maintaining a clean Git history, synchronized documentation, and a stable integration branch.

Every commit should move the project toward a deployable and reviewable state.
