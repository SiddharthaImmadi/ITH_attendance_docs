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

