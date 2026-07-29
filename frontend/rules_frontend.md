# rules_frontend.md — Frontend Development Rules

This document defines the mandatory development standards for all frontend contributors and AI development agents working on the InnoTech Hub Attendance System.

Its purpose is to ensure the frontend remains consistent, maintainable, performant, and aligned with the approved API contract, backend implementation, business rules, and overall system architecture.

All frontend implementation should comply with these rules unless an approved architectural change supersedes them.

---

# 1. Project Structure

The frontend project should follow the architecture defined in `system_architecture.md`.

Organize code by responsibility rather than by individual pages.

Recommended structure:

```
src/
 ├── components/
 ├── pages/
 ├── hooks/
 ├── lib/
 ├── services/
 ├── contexts/
 ├── layouts/
 ├── types/
 ├── utils/
 └── assets/
```

General guidelines:

- Pages compose features.
- Components remain reusable.
- Business logic belongs in hooks or services.
- API communication belongs in shared client modules.
- Utility functions remain framework-independent whenever practical.

Avoid deeply nested component hierarchies when simpler composition is possible.

---

# 2. Technology Standards

The approved frontend technology stack is summarized below.

| Concern | Standard |
|----------|----------|
| Framework | React + TypeScript |
| Build Tool | Vite |
| Styling | Tailwind CSS |
| UI Components | shadcn/ui |
| Routing | React Router |
| Forms | React Hook Form + Zod |
| Server State | TanStack Query |
| Icons | lucide-react |
| Animation | Framer Motion (purposeful use only) |

These technologies are considered project standards.

Do not introduce additional UI libraries, styling frameworks, or state-management solutions without prior approval.

---

## Styling

Follow a utility-first approach using Tailwind CSS.

Guidelines:

- Prefer utility classes over custom CSS.
- Keep spacing consistent using the shared spacing scale.
- Reuse design tokens defined in `frontend_design_system.md`.
- Avoid inline styles except for dynamic values.
- Create reusable components before duplicating UI patterns.

---

## Components

Use shadcn/ui whenever an equivalent component already exists.

Typical examples include:

- Buttons
- Dialogs
- Cards
- Forms
- Tables
- Dropdowns
- Navigation
- Toast notifications

Do not recreate components solely for stylistic differences.

Extend existing components when necessary.

---

## Forms

All user input should be managed using:

- React Hook Form
- Zod validation

Validation rules should mirror the API contract whenever possible.

The frontend should provide immediate validation feedback while relying on the backend for authoritative validation.

---

## Server State

All backend communication should use TanStack Query.

Benefits include:

- Request caching
- Background refresh
- Loading states
- Error handling
- Automatic synchronization

Avoid custom data-fetching patterns unless a specific use case requires them.

---

## Motion

Animations should improve usability rather than decorate the interface.

Appropriate examples include:

- Page transitions
- Dialog transitions
- Loading indicators
- Status updates
- Notification appearance

Avoid excessive or distracting animations.

---

# 3. User Experience Standards

The application should provide a professional, responsive, and trustworthy user experience.

Visual quality should support usability rather than distract from it.

---

## Consistency

Maintain consistent:

- Typography
- Spacing
- Colors
- Component behavior
- Navigation
- Status indicators

Users should not encounter different interaction patterns for similar actions.

---

## Feedback

Every user action should receive visible feedback.

Examples include:

- Loading indicators
- Success messages
- Validation feedback
- Error notifications
- Empty states

Never leave users uncertain whether an operation is still in progress.

---

## Error Handling

Frontend errors should communicate meaningful information.

Display backend validation messages whenever available.

Avoid exposing:

- Stack traces
- Raw exceptions
- Debug information
- Generic "Something went wrong" messages without context

Errors should help users understand the next appropriate action.

---

## Empty States

Design meaningful empty states for situations such as:

- No events available
- No attendance history
- No notifications
- No assigned activities
- No search results

Whenever possible, guide users toward the next available action.

---

## Accessibility

Frontend development should follow the accessibility standards defined in `frontend_design_system.md`.

Every interactive component should support:

- Keyboard navigation
- Visible focus indicators
- Screen reader compatibility
- Appropriate color contrast

Accessibility should be treated as a core quality requirement rather than an enhancement.

---

# 4. Frontend Responsibilities

The frontend is responsible for presenting information, collecting user input, and communicating with the backend.

Business decisions remain the responsibility of the backend.

---

## Source of Truth

The backend is the authoritative source for:

- Attendance status
- Presence status
- Event state
- Validation results
- Permission decisions
- Report calculations
- Business rules

The frontend must never attempt to reproduce backend business logic.

---

## Client Responsibilities

The frontend is responsible for:

- Displaying backend data accurately.
- Collecting required user input.
- Validating basic input formats.
- Managing user interaction.
- Presenting loading and error states.
- Rendering timelines, reports, dashboards, and notifications.

---

## Business Logic

Do not calculate or infer backend decisions.

Examples include:

- Final attendance status
- Attendance duration
- Presence compliance
- Permission decisions
- Volunteer restrictions
- Administrative approvals

The frontend displays the values returned by the backend exactly as provided.

---

## Data Integrity

Do not modify backend responses to fit UI expectations.

Instead:

- Update the UI to match the API.
- Raise API inconsistencies through the approved development process.
- Record improvement suggestions in `enhancements.md` when appropriate.

Consistency between frontend and backend is more important than temporary UI convenience.

---

## Security Responsibilities

Frontend code must never:

- Expose secrets.
- Embed API keys.
- Store sensitive information insecurely.
- Trust client-side validation alone.
- Assume authorization without backend verification.

Authentication and authorization decisions always remain server-side.

---

# 5. API Integration Standards

All communication with the backend must follow the approved API contract.

The frontend should never bypass the shared API layer or communicate with backend endpoints directly from UI components.

---

## API Client

Maintain a single shared API client responsible for:

- Authentication
- Request configuration
- Authorization headers
- Error normalization
- Response parsing

Recommended location:

```
src/lib/api.ts
```

All frontend modules should use this shared client.

---

## Type Safety

Request and response models must accurately mirror the API contract.

Guidelines:

- Keep request and response types centralized.
- Update types whenever the API contract changes.
- Avoid duplicate interface definitions.
- Prefer shared reusable models over page-specific types.

Frontend types should never intentionally diverge from the approved API contract.

---

## Data Fetching

Components should not perform direct network requests.

Instead:

- Create reusable query hooks.
- Encapsulate API logic within service modules.
- Keep presentation components focused on rendering.

Typical pattern:

```
UI Component
      ↓
Custom Hook
      ↓
API Service
      ↓
Shared API Client
      ↓
Backend
```

This separation improves maintainability and testing.

---

## Authentication

The shared API layer should automatically:

- Attach authentication tokens.
- Handle token expiration.
- Normalize authentication failures.
- Redirect unauthenticated users when appropriate.

Authentication behavior should remain consistent throughout the application.

---

## Error Handling

API errors should be translated into meaningful user feedback.

Avoid exposing:

- Internal exception messages
- Stack traces
- Database errors
- Framework-specific error details

Display only information intended for end users.

---

## API Contract Compliance

The frontend must treat `API_contract.md` as the authoritative specification.

If an implementation differs from the documented contract:

1. Verify the discrepancy.
2. Do not silently work around it.
3. Raise the issue with the development team.
4. Update documentation only through the approved process.

Never "guess" API behavior.

---

# 6. Attendance & Event Workflows

The frontend is responsible for guiding users through attendance workflows while allowing the backend to make all attendance decisions.

---

## Attendance Lifecycle

Frontend workflows should accurately represent the lifecycle exposed by the backend.

Typical stages include:

- Event discovery
- Check-in
- Active participation
- Presence monitoring
- Leave and return events
- Check-out
- Attendance summary

The frontend should always reflect the current backend state.

---

## Check-In Experience

The check-in experience should be clear, predictable, and trustworthy.

Typical flow:

1. Display event information.
2. Explain required permissions.
3. Request required device permissions.
4. Collect required evidence.
5. Submit attendance request.
6. Display progress.
7. Display backend result.

The frontend should never assume a successful outcome before the backend confirms it.

---

## Presence Monitoring

When attendance monitoring is active:

- Display current monitoring status.
- Present presence history chronologically.
- Clearly indicate leave and return events.
- Refresh monitoring information using the approved synchronization strategy.

Presence information should always originate from backend responses.

---

## Check-Out Experience

Check-out should provide the same level of clarity as check-in.

Recommended flow:

1. Initiate check-out.
2. Submit request.
3. Display progress.
4. Display attendance summary.
5. Return user to the appropriate post-event experience.

Attendance calculations remain server-side.

---

## Administrative Interfaces

Administrative views should prioritize operational awareness.

Typical information includes:

- Active events
- Current attendance
- Presence status
- Members requiring attention
- Notifications
- Reports

Administrative dashboards should remain readable even when displaying large datasets.

---

## Real-Time Updates

The frontend should synchronize changing information efficiently.

Guidelines:

- Refresh only changing data.
- Reuse cached information whenever possible.
- Avoid duplicate polling from multiple components.
- Keep refresh intervals appropriate for the feature.

Real-time behavior should balance responsiveness and performance.

---

# 7. Testing Standards

Testing helps ensure frontend reliability as the application evolves.

Testing should cover both user interactions and integration with backend workflows.

---

## Unit Testing

Use:

- Vitest
- React Testing Library

Unit tests should focus on:

- Components
- Custom hooks
- Utility functions
- Validation logic

---

## Workflow Testing

Verify important user workflows including:

- Authentication
- Event browsing
- Check-in
- Presence monitoring
- Check-out
- Attendance history
- Notifications
- Administrative dashboards

---

## Error Scenarios

Test situations including:

- Network failures
- Validation failures
- Authentication expiration
- Missing permissions
- Backend rejection
- Empty responses

Users should always receive meaningful feedback.

---

## Loading States

Verify that asynchronous operations display:

- Loading indicators
- Disabled actions where appropriate
- Progress feedback
- Successful completion

The interface should never appear unresponsive during background operations.

---

## Regression Testing

When adding new features, verify that existing functionality continues to operate correctly.

Particular attention should be given to:

- Authentication
- Navigation
- Shared components
- API integration
- Attendance workflows

---

# 8. Development Workflow

Consistent development practices improve collaboration and maintainability.

---

## Branch Strategy

Development should follow the project's approved Git workflow.

Feature work should be isolated into appropriate branches before merging into the main branch through code review.

---

## Commit Messages

Use consistent commit messages.

Recommended format:

```
type(scope): short summary
```

Examples:

```
feat(attendance): add attendance summary page

fix(auth): handle expired access token

refactor(events): simplify event filters

docs(frontend): update component guidelines
```

---

## Pull Requests

Every pull request should:

- Clearly describe the change.
- Reference affected documentation where appropriate.
- Identify related API contract updates.
- Include testing performed.
- Remain focused on a single logical change.

Avoid combining unrelated features into a single pull request.

---

## Documentation

Whenever frontend behavior changes:

Review whether updates are required for:

- API Contract
- Frontend Documentation
- Design System
- Component Guidelines
- TODO documentation
- Development roadmap

Documentation should evolve alongside implementation.

---

## Code Reviews

Reviews should verify:

- Architectural consistency
- API compliance
- Accessibility
- Performance
- Reusability
- Maintainability
- Documentation updates

Review comments should prioritize long-term code quality over personal coding preferences.

---

# 9. Agent Responsibilities & Change Management

These rules apply to AI development agents and frontend contributors working on the project.

The objective is to maintain architectural consistency while preventing undocumented or incompatible changes.

---

## Respect the Approved Architecture

Frontend implementation must remain consistent with the approved project architecture.

Do not independently:

- Replace approved libraries.
- Introduce new architectural patterns.
- Reorganize project structure.
- Change routing conventions.
- Replace state-management solutions.
- Modify authentication flow.
- Introduce incompatible UI frameworks.

Architectural decisions require explicit approval.

---

## Respect the API Contract

The frontend must always implement the published API contract.

Do not:

- Assume undocumented fields exist.
- Ignore required request fields.
- Invent response properties.
- Hardcode values that belong to the backend.

If implementation and documentation disagree:

1. Stop.
2. Verify the discrepancy.
3. Report the issue.
4. Wait for clarification before continuing.

The API contract remains the single source of truth for frontend-backend communication.

---

## Respect Backend Business Logic

Business rules belong to the backend.

The frontend must never duplicate logic such as:

- Attendance calculation
- Presence validation
- Event eligibility
- Approval decisions
- Volunteer restrictions
- Administrative permissions
- Report calculations

The frontend presents backend decisions without modification.

---

## Documentation First

Before implementing a significant feature, review the relevant project documentation.

Typical references include:

- API_contract.md
- system_architecture.md
- frontend_design_system.md
- frontend_component_guidelines.md
- backend_api_implementation.md
- backend_database_schema.md

Implementation should remain consistent with approved documentation.

---

## Incremental Development

Large features should be implemented incrementally.

Recommended approach:

1. Explain the implementation plan.
2. Wait for approval when required.
3. Implement a logical milestone.
4. Verify functionality.
5. Update documentation if necessary.
6. Continue with the next milestone.

Avoid introducing multiple unrelated changes simultaneously.

---

# 10. Enhancements & Proposed Improvements

Improvement ideas are encouraged but should follow the project's approval process.

---

## Recording Suggestions

When a contributor identifies a possible improvement that is outside the current scope:

- Do not silently implement it.
- Record the proposal in `enhancements.md`.
- Explain the rationale.
- Describe expected benefits.
- Identify any architectural impact.

Implementation should occur only after approval.

---

## Examples

Examples include:

- Introducing a new component library
- Changing routing architecture
- Replacing state management
- Modifying project folder structure
- Significant UI redesign
- Performance optimizations requiring architectural changes

Minor bug fixes and non-behavioral refactoring do not require enhancement proposals.

---

## Backward Compatibility

Whenever possible:

- Extend existing components.
- Preserve existing APIs.
- Avoid unnecessary breaking changes.
- Maintain compatibility with approved documentation.

Breaking changes require explicit review and approval.

---

# 11. Scope Management

This document defines ongoing frontend engineering standards rather than phase-specific implementation rules.

Individual project milestones determine which features are implemented, while these rules govern *how* they are implemented.

---

## Current Scope

Frontend implementation may include features described in the approved project documentation, including:

- Authentication
- Event management
- Attendance workflows
- Presence monitoring
- Administrative dashboards
- Reports
- Notifications
- Activity management
- Shared UI components

Only implement features that have been approved for the current development milestone.

---

## Future Features

Future capabilities should not be partially implemented in advance.

Examples include:

- Experimental interfaces
- Placeholder workflows
- Unapproved administrative tools
- Unsupported backend functionality

If backend support does not yet exist, avoid creating incomplete frontend implementations.

---

## Consistency Across Modules

Every new module should follow the same standards for:

- Component structure
- API integration
- Error handling
- Loading states
- Accessibility
- Testing
- Documentation

Avoid creating feature-specific conventions that differ from the rest of the application.

---

# 12. Core Engineering Principles

The following principles guide all frontend development.

---

## Consistency

Prefer extending existing patterns over introducing new ones.

Reusable components and shared design patterns improve maintainability.

---

## Simplicity

Choose the simplest implementation that satisfies project requirements.

Avoid unnecessary abstraction or premature optimization.

---

## Maintainability

Write code that is easy to understand, review, test, and extend.

Favor readability over clever implementations.

---

## Performance

Optimize thoughtfully.

Prioritize:

- Efficient rendering
- Appropriate caching
- Reduced unnecessary requests
- Lazy loading where appropriate
- Shared state reuse

Performance improvements should never compromise correctness.

---

## Accessibility

Accessibility is a mandatory quality requirement.

Every feature should support:

- Keyboard navigation
- Screen readers
- Focus management
- Adequate color contrast
- Responsive layouts

Accessibility should be considered throughout development rather than added afterward.

---

## Reliability

Frontend behavior should remain predictable even when backend operations fail.

Users should always receive:

- Clear feedback
- Meaningful error messages
- Appropriate loading indicators
- Recoverable workflows

Unexpected failures should degrade gracefully rather than leaving the application in an inconsistent state.

---

## Collaboration

Frontend development is part of a larger system.

Implementation decisions should remain aligned with:

- Backend architecture
- API contract
- Business rules
- Design system
- Shared documentation

Successful collaboration depends on consistency across the entire project.

---

# Conclusion

This document establishes the engineering standards for frontend development within the InnoTech Hub Attendance System.

It defines expectations for project structure, technology choices, API integration, user experience, development workflow, testing, documentation, and collaboration.

All contributors should follow these standards to ensure the frontend remains consistent, maintainable, accessible, and aligned with the project's approved architecture and documentation.

As the application evolves, these rules should be refined through the project's documentation process while preserving architectural consistency and long-term maintainability.

# 13. Phase 3 Activity Layer Rules

Phase 3 extends the frontend rules to support activity management, volunteer assignments, progress updates, evidence submission, administrator review, activity templates, and activity reporting.

All existing frontend engineering rules remain applicable.

The backend remains authoritative for all Activity Layer business decisions.

---

## 13.1 Activity Authority

The frontend must never independently determine:

- activity lifecycle state;
- assignment eligibility;
- assignment conflicts;
- assignment status;
- submission validity;
- review outcomes;
- volunteer permissions;
- template validity.

These decisions belong to the backend.

The frontend displays and reacts to backend responses.

---

## 13.2 Activity Creation

Activities may only be created by administrators through the application.

The frontend must not provide activity creation functionality to members.

Activity creation forms should:

- collect only fields defined by the API contract;
- validate basic input format;
- display backend validation errors;
- prevent accidental duplicate submissions.

If an administrator creates an activity by mistake, the UI should expose only the lifecycle actions permitted by the backend.

---

## 13.3 Activity Independence

Every newly created activity is an independent record.

Activities generated from templates must also become independent activities.

The frontend must not maintain a live relationship where changes to a template modify previously generated activities.

---

## 13.4 Activity Lifecycle

The frontend must represent the activity lifecycle exactly as returned by the backend.

Do not infer lifecycle transitions locally.

Actions should only be displayed when appropriate for the current backend state.

Examples include:

- Edit
- Publish
- Cancel
- Archive
- Assign

After a successful action, refresh the affected activity state.

---

## 13.5 Assignment Rules

Only administrators may assign activities.

The frontend should support:

- individual assignment;
- approved bulk assignment operations;
- assignment conflict feedback.

If the backend reports an assignment conflict, the frontend must block completion of that assignment until the conflict is resolved.

The frontend must not override or ignore backend conflict responses.

---

## 13.6 Assignment Notifications

Members should receive notifications for new activity assignments.

Activity assignment notifications should:

- clearly identify the activity;
- navigate to the related assignment when selected.

Do not generate member notifications for unrelated Activity Layer operations unless required by the approved notification contract.

---

## 13.7 Late Joining Members

A member who joins an event after activities have already started must not automatically inherit existing work.

Their activity assignment should be determined after discussion with the administrator.

The frontend should display only assignments returned by the backend.

---

## 13.8 Assignment Start

Once a member starts an assigned activity, the member cannot independently abandon or switch that activity.

The frontend should not provide an unsupported:

- abandon action;
- switch activity action;
- self-reassignment action.

Any reassignment decision must follow backend and administrator rules.

---

## 13.9 Progress Updates

Members may submit progress updates only for activities assigned to them.

Progress updates should:

- be displayed chronologically;
- preserve previously submitted updates;
- clearly communicate synchronization state;
- become part of the activity history after successful synchronization.

The frontend must not rewrite historical progress records.

---

## 13.10 Evidence Requirements

Activity evidence may include photographs and videos.

Current limits are:

- maximum 10 photographs;
- maximum 2 videos;
- maximum 60 seconds per video.

The frontend should communicate these limits before submission.

Client-side validation may prevent obviously invalid selections, but backend validation remains authoritative.

---

## 13.11 Evidence Storage State

Before successful synchronization, locally captured evidence may be represented as pending.

The interface should distinguish between:

- locally stored;
- uploading;
- synchronized;
- failed.

Never display unsynchronized evidence as successfully stored on the server.

---

## 13.12 Offline Activity Updates

When connectivity is unavailable, supported activity updates should be stored locally.

The frontend should:

1. preserve supported pending updates locally;
2. clearly indicate that synchronization is pending;
3. detect restored connectivity;
4. attempt synchronization;
5. update the UI after backend confirmation.

The frontend must never treat locally stored data as authoritative backend state.

---

## 13.13 Offline Evidence Failure

If locally stored evidence is lost before successful synchronization, it cannot be treated as submitted evidence.

The frontend should clearly inform the member that the evidence must be captured again.

Do not create placeholder evidence records for files that no longer exist.

---

## 13.14 Activity Submission

When the member completes an activity, the submission is sent to the backend for review.

The frontend must:

- display submission progress;
- prevent accidental duplicate submissions;
- wait for backend confirmation;
- refresh assignment state after success.

Do not mark an activity as submitted before backend confirmation.

---

## 13.15 Submission Immutability

Once a submission has been successfully completed, the member cannot edit that submitted version.

The member may continue to:

- view the submission;
- view submitted evidence;
- view review information.

Editing controls must not remain available for a completed submission.

---

## 13.16 Review States

The Activity Layer review workflow uses:

- `NEEDS_CHANGES`
- `VERIFIED`

The frontend must not introduce a separate `REJECTED` review state.

`NEEDS_CHANGES` means the member is allowed to correct the identified problems and submit the activity again.

---

## 13.17 Needs Changes

When the backend returns `NEEDS_CHANGES`:

- administrator remarks are mandatory;
- remarks must be clearly displayed to the member;
- the required corrections should remain visible;
- the member must be allowed to continue the activity;
- the member must be allowed to resubmit after corrections.

The frontend must preserve previous submission and review history for viewing.

---

## 13.18 Verified Activities

When an activity is verified:

- the final submission remains read-only;
- submitted evidence remains viewable;
- review information remains viewable;
- administrator remarks should be displayed when provided.

The frontend must not provide further editing or resubmission actions for a verified activity.

---

## 13.19 Administrator Remarks

Administrator remarks should be presented according to review outcome.

For `NEEDS_CHANGES`:

- remarks are mandatory;
- remarks should receive strong visual emphasis.

For `VERIFIED`:

- remarks may contain the administrator's completion feedback.

Frontend validation may enforce required remarks for `NEEDS_CHANGES`, but the backend remains authoritative.

---

## 13.20 Evidence History

Evidence attached to successfully submitted activity records remains associated with those records.

The frontend must not provide controls that imply historical submitted evidence can be silently replaced or removed.

Evidence history should remain viewable where authorized.

---

## 13.21 Activity History

Members may view their own Activity History.

Activity History is read-only.

The frontend should:

- use the documented Activity History endpoint;
- display records chronologically;
- support documented pagination;
- render backend-provided categories and metadata appropriately.

Do not construct an authoritative Activity History solely from local frontend state.

---

## 13.22 Administrator Review Queue

The administrator review interface should prioritize activities requiring review.

The frontend may provide:

- filtering;
- sorting;
- pagination;
- search where supported.

Filtering must use documented API capabilities.

Do not invent unsupported backend filters.

---

## 13.23 Review Responsibility

For the current scope, there is one administrator/reviewer.

The frontend should therefore avoid introducing:

- reviewer assignment workflows;
- reviewer selection;
- multi-reviewer coordination;
- reviewer workload balancing.

These capabilities require explicit future approval.

---

## 13.24 Activity Templates

Only administrators may manage activity templates.

The frontend may provide approved operations for:

- creating templates;
- editing templates;
- applying templates;
- viewing templates.

Applying a template creates new independent activities.

Changes to a template must not visually imply that previously created activities will also change.

---

## 13.25 Activity Reports

Activity reporting should use backend-generated report data.

The frontend is responsible for:

- requesting reports;
- displaying generation progress;
- downloading completed reports;
- communicating failures.

The frontend must not reproduce authoritative report calculations locally.

Detailed filtering may be performed after export where the approved reporting workflow expects CSV-based analysis.

---

## 13.26 Bulk Operations

Approved administrator workflows may support bulk operations.

Bulk interfaces must:

- clearly show selected records;
- show the number of selected records;
- require confirmation for destructive actions;
- display partial or complete backend failures accurately.

The frontend must not assume every item in a bulk request succeeded unless confirmed by the backend.

---

## 13.27 Lazy Loading

Lazy loading should be used where it improves initial load performance without harming core workflows.

Good candidates include:

- Activity Management;
- Activity Templates;
- Review Queue;
- Activity Reports;
- large evidence previews.

Frequently used critical screens should not be fragmented unnecessarily.

Lazy loading is a performance technique, not a business requirement.

---

## 13.28 Loading and Duplicate Actions

While a mutating Activity Layer request is being processed:

- provide visible loading feedback;
- disable the triggering action when duplicate execution would be unsafe;
- avoid submitting the same request multiple times.

After completion, synchronize the affected server state.

---

## 13.29 Backend Error Handling

Activity Layer errors should follow the same API error standards as the rest of the application.

Examples include:

- assignment conflict;
- invalid activity state;
- evidence limit exceeded;
- unauthorized operation;
- submission conflict;
- review conflict.

Display the backend-provided user-safe explanation when available.

Never silently bypass a backend error.

---

## 13.30 Activity Permissions

Member interfaces may expose only member-authorized operations.

Administrator interfaces may expose only administrator-authorized operations.

Hiding an action in the UI is not security.

The backend must still authorize every operation.

---

## 13.31 API Contract Protection

Activity Layer development must follow `API_contract.md` exactly.

Frontend contributors and AI agents must never independently modify the API contract.

If the frontend requires something not currently provided:

1. Stop the affected implementation.
2. Explain the requirement.
3. Record a proposed improvement in `enhancements.md` where appropriate.
4. Wait for approval.
5. Continue only after the approved documentation process is complete.

Do not modify frontend behavior to create an undocumented API contract.

---

## 13.32 Phase 3 Frontend Principles

All Activity Layer frontend development should follow these principles:

- backend state is authoritative;
- API contracts are never changed without approval;
- members interact only with their own assignments;
- administrators control activity management and review;
- submissions become read-only after completion;
- `NEEDS_CHANGES` supports correction and resubmission;
- `REJECTED` is not part of the review workflow;
- evidence limits are clearly communicated;
- offline state is never presented as synchronized state;
- assignment conflicts must be resolved rather than bypassed;
- templates create independent activities;
- historical information remains viewable;
- UI behavior remains consistent with the existing design system;
- accessibility remains mandatory;
- frontend business logic remains minimal.
