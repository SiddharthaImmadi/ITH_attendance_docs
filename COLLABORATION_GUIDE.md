# COLLABORATION_GUIDE.md — Team Collaboration Guide

> This guide defines how the frontend developer, backend developer, and their AI assistants
> collaborate throughout the project. It establishes the communication workflow, Git workflow,
> documentation workflow, API workflow, and milestone completion process so development remains
> coordinated, predictable, and easy to maintain.

---

# 1. Purpose

This project is developed by two developers working in parallel:

- Backend Developer
- Frontend Developer

Both developers use AI development assistants (ChatGPT, Claude Code, Kiro, etc.) to assist with
planning, implementation, debugging, documentation, and code review.

The objective of this guide is to ensure that everyone—human or AI—follows the same workflow.

---

# 2. Team Responsibilities

## Backend Developer

Owns:

- API implementation
- Database
- Business logic
- Authentication
- GPS validation
- Attendance validation
- Report generation
- Database migrations

Primary references:

- `rules_backend.md`
- `API_contract.md`
- `system_architecture.md`

---

## Frontend Developer

Owns:

- React application
- User interface
- Camera integration
- GPS collection
- Forms
- User experience
- API integration
- Error handling
- Loading states

Primary references:

- `rules_frontend.md`
- `API_contract.md`
- `system_architecture.md`

---

## Shared Responsibilities

Both developers are responsible for:

- Following the PRD
- Following the roadmap
- Maintaining documentation
- Reviewing API contracts
- Code reviews
- Integration testing
- Milestone completion
- Updating project progress

No feature is considered complete until both frontend and backend agree that the integration works.

---

# 3. Documentation First Workflow

Documentation is the project's source of truth.

Before implementing a feature:

1. Confirm the feature exists in the current milestone.
2. Read the relevant documentation.
3. Review the API contract.
4. Implement the feature.
5. Test the feature.
6. Update documentation if required.

Never implement undocumented features.

If documentation and implementation disagree:

- Stop implementation.
- Review the documentation.
- Update documentation first if necessary.
- Continue implementation.

---

# 4. Team Communication

Maintain one shared communication channel.

Examples:

- Slack
- Discord
- WhatsApp
- Microsoft Teams
- Shared Google Doc

Communication should be short, focused, and actionable.

---

## Daily Standup Template

```
[Name] — [Date]

✅ Completed:
- ...

🔨 Working On:
- ...

🚧 Blockers:
- ...
```

Example:

```
Bhargav — Jul 21

✅ Completed
- Login page
- API integration

🔨 Working On
- Session creation

🚧 Blocker
- Waiting for /sessions endpoint
```

Standups should communicate:

- completed work
- current work
- blockers

Nothing more.

---

# 5. Progress Tracking

At the end of every work session:

Update:

- `progress.md`

If documentation changed:

Update:

- `changelog.md`

Progress updates should include:

- milestone
- completed work
- current work
- blockers
- next tasks

The progress log is the project's historical record.

---

# 6. Development Workflow

Development follows milestones.

For every milestone:

```
Documentation
        ↓
API Contract
        ↓
Backend Implementation
        ↓
Frontend Integration
        ↓
Testing
        ↓
Documentation Update
        ↓
Milestone Complete
```

Never skip steps.

---

# 7. Git Workflow

Branches:

```
main
backend
frontend
```

Backend developer works only on:

```
backend
```

Frontend developer works only on:

```
frontend
```

Never commit directly to `main`.

Example:

```bash
git checkout frontend

git pull origin frontend

# work

git add .

git commit -m "feat(auth): login page"

git push origin frontend
```

After completing a feature:

- Create a Pull Request.
- Request review.
- Merge only after approval.

After merge:

```bash
git checkout main
git pull origin main

git checkout frontend
git merge main
```

Backend follows the same workflow using the `backend` branch.

---

# 8. API Contract Workflow

`API_contract.md` is the communication contract between frontend and backend.

Before implementation:

- Read the contract.
- Understand request fields.
- Understand response fields.
- Understand validation rules.
- Understand error responses.

Never guess.

If the contract changes:

1. Update `API_contract.md`
2. Commit documentation.
3. Inform both developers.
4. Continue implementation.

Documentation always changes before implementation.

---

# 9. Parallel Development

Frontend does not need to wait for backend.

Frontend may use:

- Mock Service Worker (MSW)
- Mock API responses
- Static JSON
- Temporary test data

Once backend endpoints are available:

- Replace mocks.
- Test against real APIs.
- Remove temporary implementations.

---

# 10. Daily Collaboration Checklist

Every work session:

☐ Pull latest changes

☐ Read teammate's progress

☐ Check updated documentation

☐ Review API changes

☐ Complete assigned milestone work

☐ Push commits

☐ Update progress.md

---

# 11. Common Collaboration Scenarios

## Backend endpoint becomes available

- Pull latest changes.
- Replace mocks.
- Test integration.
- Report results.

---

## Waiting on another developer

If blocked:

- Record blocker.
- Inform teammate.
- Continue another independent task.

Avoid idle time.

---

## API contract unclear

Never guess.

Clarify first.

Update the contract if required.

Continue development afterward.

---

## Integration failure

If frontend and backend disagree:

1. Verify API contract.
2. Verify request.
3. Verify response.
4. Verify documentation.
5. Resolve together.

Documentation should always reflect the final agreed behavior.

---

# 12. Pull Request Process

Every completed feature should be reviewed.

A PR should include:

- Feature summary
- Documentation updates
- API changes
- Testing performed
- Known limitations

Reviewer verifies:

- Documentation consistency
- API consistency
- Code quality
- Integration
- Milestone completion

Only then should it merge into `main`.

---

# 13. Commit Message Convention

Format:

```
type(scope): short description
```

Types:

- feat
- fix
- docs
- refactor
- test
- chore

Examples:

```
feat(auth): implement login API

fix(report): correct attendance calculation

docs(api): update attendance endpoints

refactor(session): simplify validation logic
```

---

# 14. Real-Time Communication

Contact your teammate immediately when:

- Critical bugs are discovered.
- API behavior differs from documentation.
- A merge is urgently required.
- Clarification is needed before continuing.

Everything else can wait until the daily standup.

---

# 15. AI Assistant Collaboration

AI assistants should support—not replace—developer decision making.

Before writing code, an AI assistant should:

- Read the relevant documentation.
- Understand the current milestone.
- Follow project architecture.
- Follow coding standards.
- Respect the API contract.

AI assistants should never:

- Invent undocumented features.
- Change project scope.
- Modify architecture without approval.
- Ignore existing documentation.

When suggesting improvements:

- Record deferred ideas in `enhancements.md`.
- Keep implementation within the current milestone.

---

# 16. Development Tools

| Tool | Purpose |
|------|---------|
| Git | Version control |
| GitHub | Pull Requests & Reviews |
| PostgreSQL | Database |
| FastAPI | Backend |
| React | Frontend |
| Postman / Insomnia | API testing |
| MSW / Mock APIs | Frontend development |
| React DevTools | UI debugging |
| progress.md | Progress tracking |
| API_contract.md | Frontend/Backend agreement |
| Documentation | Project source of truth |

---

# 17. Milestone Completion Checklist

A milestone is complete only when:

- ☐ Backend implementation is finished.
- ☐ Frontend implementation is finished.
- ☐ API integration works.
- ☐ Documentation is updated.
- ☐ Tests pass.
- ☐ No known blockers remain.
- ☐ Progress is recorded.
- ☐ Both developers agree the milestone is complete.

---

# 18. Collaboration Principles

Every developer and AI assistant should follow these principles:

1. Documentation before implementation.
2. API contract before integration.
3. Communication before assumptions.
4. Small milestones over large unfinished features.
5. Review before merge.
6. Test before completion.
7. Update documentation continuously.
8. Keep the project runnable after every milestone.
9. Preserve consistency between documentation, implementation, and architecture.
10. Build together, review together, and complete milestones together.

---

The goal of this guide is simple:

> Every developer and every AI assistant should be able to join the project at any point, read this
> document, and immediately understand how the team collaborates to deliver each milestone
> successfully.