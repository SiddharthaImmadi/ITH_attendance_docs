# Frontend Development Roadmap

This document serves as the implementation roadmap for the frontend of the InnoTech Hub Attendance System.

Tasks are organized by functional area rather than project phase. This allows new features to be added without restructuring the roadmap and keeps implementation aligned with the current backend architecture and API contract.

---

# 1. Foundation

## Project Setup

- [ ] Create React + TypeScript project using Vite
- [ ] Configure Tailwind CSS
- [ ] Configure shadcn/ui
- [ ] Configure React Router
- [ ] Configure TanStack Query
- [ ] Configure React Hook Form
- [ ] Configure Zod
- [ ] Configure Framer Motion
- [ ] Configure lucide-react
- [ ] Configure ESLint and formatting tools

---

## Project Structure

- [ ] Create project folder structure
- [ ] Configure layouts
- [ ] Configure reusable component folders
- [ ] Configure hooks
- [ ] Configure API services
- [ ] Configure shared utilities
- [ ] Configure shared types
- [ ] Configure authentication context

---

## Environment

- [ ] Configure environment variables
- [ ] Configure API base URL
- [ ] Configure development environment
- [ ] Verify backend connectivity
- [ ] Verify CORS configuration

---

## Shared Infrastructure

- [ ] Create shared API client
- [ ] Configure request interceptors
- [ ] Configure authentication handling
- [ ] Configure centralized error handling
- [ ] Configure React Query provider
- [ ] Configure application routing
- [ ] Configure protected routes

---

# 2. Authentication

## Login

- [ ] Create Login page
- [ ] Build authentication form
- [ ] Implement validation
- [ ] Implement loading states
- [ ] Display backend validation messages
- [ ] Handle authentication failures
- [ ] Support responsive layouts

---

## Authentication State

- [ ] Create authentication context
- [ ] Store access tokens securely
- [ ] Restore authenticated sessions
- [ ] Fetch current user profile
- [ ] Handle expired authentication
- [ ] Support logout
- [ ] Protect authenticated routes

---

## Authorization

- [ ] Configure role-based routing
- [ ] Create administrator layouts
- [ ] Create member layouts
- [ ] Restrict unauthorized pages
- [ ] Handle permission failures

---

# 3. Event Management

## Event Dashboard

- [ ] Create administrator dashboard
- [ ] Display event overview
- [ ] Display upcoming events
- [ ] Display active events
- [ ] Display completed events
- [ ] Display event statistics

---

## Event Management

- [ ] Create event creation page
- [ ] Create event editing page
- [ ] Display event details
- [ ] Validate event information
- [ ] Submit event updates
- [ ] Display backend validation errors

---

## Event Listing

- [ ] Display searchable event list
- [ ] Filter by event status
- [ ] Filter by date
- [ ] Sort events
- [ ] Support pagination where appropriate

---

## Event Details

- [ ] Display event information
- [ ] Display attendance summary
- [ ] Display assigned members
- [ ] Display event timeline
- [ ] Display administrative actions

---

## Boundary Configuration

- [ ] Display configured attendance boundary
- [ ] Support approved boundary visualization
- [ ] Display boundary metadata
- [ ] Validate boundary configuration before submission

---

## Member Event View

- [ ] Display available events
- [ ] Display active event
- [ ] Display completed events
- [ ] Display event schedule
- [ ] Display participation history

---

# 4. Attendance Management

Attendance is the core workflow of the application. All attendance information displayed by the frontend must originate from backend responses.

---

## Check-In Experience

### Check-In Preparation

- [ ] Display event information before check-in
- [ ] Explain required permissions
- [ ] Display privacy notice before requesting device access
- [ ] Validate event eligibility
- [ ] Display attendance requirements

---

### Device Permissions

- [ ] Request location permission
- [ ] Request camera permission
- [ ] Handle permission denial gracefully
- [ ] Allow retry after denied permissions
- [ ] Display permission guidance

---

### Attendance Capture

- [ ] Capture live device location
- [ ] Capture required attendance evidence
- [ ] Display captured information before submission
- [ ] Allow recapture when applicable
- [ ] Validate required inputs before submission

---

### Attendance Submission

- [ ] Submit attendance request
- [ ] Display loading state
- [ ] Prevent duplicate submissions
- [ ] Handle network failures
- [ ] Handle backend validation failures
- [ ] Display successful attendance confirmation
- [ ] Display backend rejection reasons

---

## Attendance Status

- [ ] Display current attendance status
- [ ] Display attendance timestamps
- [ ] Display backend status messages
- [ ] Display attendance history
- [ ] Display attendance summary

---

## Attendance History

- [ ] Display chronological attendance records
- [ ] Filter attendance history
- [ ] Search attendance history
- [ ] Display attendance details
- [ ] Display attendance outcomes
- [ ] Support pagination where appropriate

---

## Attendance Administration

- [ ] Display participant attendance list
- [ ] Display attendance statistics
- [ ] Filter attendance by status
- [ ] Search participants
- [ ] Display attendance details
- [ ] Refresh attendance information
- [ ] Support attendance exports

---

# 5. Presence Monitoring

Presence monitoring provides continuous visibility into participant status during active events.

The frontend displays monitoring information while the backend remains responsible for all attendance decisions.

---

## Active Event

- [ ] Display active event information
- [ ] Display monitoring status
- [ ] Display current presence state
- [ ] Display monitoring indicators
- [ ] Display event progress

---

## Presence Timeline

- [ ] Create reusable Presence Timeline component
- [ ] Display events chronologically
- [ ] Display timestamps
- [ ] Display backend messages
- [ ] Group related timeline events where appropriate
- [ ] Support automatic refresh

---

## Leave & Return

- [ ] Display leave events
- [ ] Display return events
- [ ] Display backend warnings
- [ ] Display monitoring status changes
- [ ] Display current participant state
- [ ] Display administrative remarks when available

---

## Presence Synchronization

- [ ] Refresh monitoring data using approved synchronization strategy
- [ ] Avoid duplicate refresh requests
- [ ] Refresh only active monitoring information
- [ ] Reuse cached monitoring data
- [ ] Recover automatically after temporary failures

---

## Monitoring Dashboard

- [ ] Display members currently present
- [ ] Display members temporarily outside
- [ ] Display members awaiting action
- [ ] Display members who completed attendance
- [ ] Display monitoring summary
- [ ] Display monitoring alerts
- [ ] Support filtering and search

---

## Attendance Completion

- [ ] Initiate attendance completion
- [ ] Display confirmation dialog
- [ ] Submit completion request
- [ ] Display loading state
- [ ] Display attendance summary
- [ ] Return user to appropriate dashboard

---

## Attendance Summary

- [ ] Display attendance overview
- [ ] Display event information
- [ ] Display check-in details
- [ ] Display check-out details
- [ ] Display attendance timeline
- [ ] Display backend attendance outcome
- [ ] Display administrative remarks when applicable

---

## Monitoring Error Handling

- [ ] Handle monitoring refresh failures
- [ ] Handle synchronization failures
- [ ] Handle connectivity interruptions
- [ ] Display meaningful recovery messages
- [ ] Retry failed refresh operations when appropriate
- [ ] Preserve existing monitoring information during temporary failures

---

## Monitoring Performance

- [ ] Prevent unnecessary re-renders
- [ ] Optimize dashboard refresh
- [ ] Optimize timeline rendering
- [ ] Reuse cached monitoring data
- [ ] Avoid duplicate polling
- [ ] Verify performance with large participant lists

---

# 6. Activity Management

Activity management enables administrators to create, assign, monitor, review, and report on event-related work while allowing members to complete assigned activities and submit supporting evidence.

All Activity Layer behavior must follow the approved API contract and backend business rules.

---

## Administrator Activity Management

### Activity Listing

- [ ] Create Activity Management page
- [ ] Display activities
- [ ] Display activity status
- [ ] Display activity priority
- [ ] Display activity category
- [ ] Display associated event
- [ ] Display assignment summary
- [ ] Support search
- [ ] Filter by status
- [ ] Filter by priority
- [ ] Filter by category
- [ ] Filter by event where supported
- [ ] Support pagination
- [ ] Preserve filters during navigation where appropriate
- [ ] Display loading states
- [ ] Display empty states
- [ ] Display API errors

---

### Activity Creation

- [ ] Create Activity Creation page
- [ ] Build activity creation form
- [ ] Validate required fields
- [ ] Display backend validation errors
- [ ] Prevent duplicate submissions
- [ ] Display creation progress
- [ ] Handle successful creation
- [ ] Navigate to appropriate activity view after creation

Activity creation must only be available inside the application to authorized administrators.

---

### Activity Details

- [ ] Create administrator Activity Details page
- [ ] Display activity information
- [ ] Display activity status
- [ ] Display activity priority
- [ ] Display category
- [ ] Display associated event
- [ ] Display assignments
- [ ] Display activity timeline
- [ ] Display submission summary
- [ ] Display evidence summary
- [ ] Display review information
- [ ] Display only backend-supported actions

---

### Activity Editing

- [ ] Support editing where permitted
- [ ] Pre-populate existing activity information
- [ ] Validate updated information
- [ ] Display backend validation errors
- [ ] Prevent duplicate update requests
- [ ] Refresh activity state after successful update

---

### Activity Lifecycle

- [ ] Display activity lifecycle state
- [ ] Implement Publish action
- [ ] Implement Cancel action
- [ ] Implement Archive action
- [ ] Display confirmation for destructive actions
- [ ] Disable duplicate lifecycle requests
- [ ] Refresh activity information after lifecycle changes
- [ ] Handle invalid state transitions returned by backend

The frontend must never infer lifecycle transitions independently.

---

## Activity Assignment

### Volunteer Assignment

- [ ] Create assignment interface
- [ ] Display eligible volunteers returned by backend
- [ ] Support individual assignment
- [ ] Support approved bulk assignment operations
- [ ] Display currently assigned volunteers
- [ ] Display assignment status
- [ ] Prevent accidental duplicate requests
- [ ] Refresh assignment state after successful changes

---

### Assignment Conflicts

- [ ] Display assignment conflict clearly
- [ ] Display backend-provided conflict reason
- [ ] Block conflicting assignment until resolved
- [ ] Preserve unaffected selections where appropriate
- [ ] Allow administrator to resolve conflict and retry
- [ ] Never bypass backend conflict validation

---

### Late Joining Members

- [ ] Display late-joining members where applicable
- [ ] Do not automatically assign existing work
- [ ] Allow administrator to determine appropriate assignment
- [ ] Display only backend-confirmed assignments

---

### Bulk Assignment

- [ ] Support selecting multiple eligible volunteers
- [ ] Display selected volunteer count
- [ ] Clearly display bulk action
- [ ] Confirm potentially destructive bulk operations
- [ ] Display backend results accurately
- [ ] Handle partial failures where supported

---

## Member Activities

### My Activities

- [ ] Create My Activities page
- [ ] Display assigned activities
- [ ] Display activity title
- [ ] Display associated event
- [ ] Display priority
- [ ] Display assignment status
- [ ] Display latest progress
- [ ] Group or filter assignments by status
- [ ] Support pagination where required
- [ ] Display empty state when no activities are assigned
- [ ] Display loading and error states

---

### Member Activity Details

- [ ] Create member Activity Details page
- [ ] Display activity instructions
- [ ] Display activity information
- [ ] Display priority
- [ ] Display assignment status
- [ ] Display submission requirements
- [ ] Display progress timeline
- [ ] Display evidence
- [ ] Display review feedback
- [ ] Display only currently permitted actions

---

### Activity Start

- [ ] Implement activity start workflow where required by API
- [ ] Display backend-confirmed state
- [ ] Prevent unsupported activity abandonment
- [ ] Prevent unsupported self-reassignment
- [ ] Refresh assignment state after successful start

Once work has started, the frontend must not provide unsupported abandon or self-switch actions.

---

## Progress Tracking

### Progress Updates

- [ ] Create progress update interface
- [ ] Allow member to enter progress information
- [ ] Validate required fields
- [ ] Submit progress through approved API
- [ ] Prevent duplicate submissions
- [ ] Display synchronization status
- [ ] Refresh progress history after successful synchronization

---

### Progress Timeline

- [ ] Create reusable Activity Progress Timeline
- [ ] Display progress updates chronologically
- [ ] Display timestamps
- [ ] Display descriptions
- [ ] Display associated evidence
- [ ] Preserve historical entries
- [ ] Display loading state
- [ ] Display empty state

Historical progress entries must remain read-only.

---

## Evidence Management

### Evidence Capture

- [ ] Create evidence capture/upload interface
- [ ] Support approved photo capture
- [ ] Support approved video capture
- [ ] Display evidence requirements before capture
- [ ] Display selected evidence previews
- [ ] Allow removal before successful submission where permitted

---

### Evidence Limits

- [ ] Enforce basic client-side maximum of 10 photographs
- [ ] Enforce basic client-side maximum of 2 videos
- [ ] Validate maximum video duration of 60 seconds
- [ ] Display current photo count
- [ ] Display current video count
- [ ] Display backend validation failures

Backend validation remains authoritative.

---

### Evidence Upload

- [ ] Display upload progress
- [ ] Display individual upload state
- [ ] Distinguish pending and synchronized evidence
- [ ] Handle upload failures
- [ ] Support retry where appropriate
- [ ] Never display failed evidence as synchronized
- [ ] Refresh backend evidence state after successful upload

---

### Evidence Gallery

- [ ] Create reusable Evidence Gallery
- [ ] Display photo previews
- [ ] Display video previews
- [ ] Display evidence type
- [ ] Display synchronization state where relevant
- [ ] Support accessible media controls
- [ ] Support responsive layouts

---

## Offline Activity Support

### Local Pending State

- [ ] Detect connectivity loss
- [ ] Preserve supported pending progress updates locally
- [ ] Preserve supported pending evidence locally
- [ ] Clearly display pending synchronization state
- [ ] Avoid representing local data as server-confirmed data

---

### Synchronization

- [ ] Detect restored connectivity
- [ ] Automatically attempt synchronization
- [ ] Display synchronization progress
- [ ] Refresh backend state after successful synchronization
- [ ] Handle synchronization failures
- [ ] Allow retry where appropriate
- [ ] Prevent duplicate synchronization

---

### Lost Evidence

- [ ] Detect when locally referenced evidence is no longer available where possible
- [ ] Inform member that lost evidence must be captured again
- [ ] Do not create placeholder evidence
- [ ] Do not mark missing evidence as submitted

---

## Activity Submission

### Submission Preparation

- [ ] Display submission summary
- [ ] Display progress history
- [ ] Display evidence summary
- [ ] Display required submission information
- [ ] Validate basic submission requirements
- [ ] Provide final submission confirmation

---

### Submission

- [ ] Submit completed activity for review
- [ ] Display submission progress
- [ ] Prevent duplicate submissions
- [ ] Wait for backend confirmation
- [ ] Display successful submission confirmation
- [ ] Handle backend validation errors
- [ ] Refresh assignment state after submission

---

### Submitted State

- [ ] Make completed submission read-only
- [ ] Remove editing controls
- [ ] Remove evidence deletion controls
- [ ] Preserve evidence viewing
- [ ] Preserve progress viewing
- [ ] Display review status

---

## Activity Review

### Review Queue

- [ ] Create administrator Review Queue
- [ ] Display submissions awaiting review
- [ ] Display member information
- [ ] Display activity information
- [ ] Display submission time
- [ ] Display review status
- [ ] Support documented filtering
- [ ] Support documented sorting
- [ ] Support pagination
- [ ] Display empty state when no reviews are pending

---

### Review Details

- [ ] Create Activity Review page
- [ ] Display submitted progress
- [ ] Display submitted evidence
- [ ] Display previous review history
- [ ] Display member information
- [ ] Display activity information
- [ ] Display review controls

---

### Verify Activity

- [ ] Implement Verify action
- [ ] Support administrator completion remark
- [ ] Display confirmation
- [ ] Prevent duplicate review actions
- [ ] Refresh review and assignment state after success
- [ ] Display Verified state to member

---

### Needs Changes

- [ ] Implement Needs Changes action
- [ ] Require administrator remarks
- [ ] Validate remarks before submission
- [ ] Display backend validation errors
- [ ] Refresh review state after success
- [ ] Display remarks prominently to member
- [ ] Allow member to continue activity
- [ ] Allow corrected resubmission

Do not implement a separate `REJECTED` review workflow.

---

### Resubmission

- [ ] Display previous administrator remarks
- [ ] Preserve previous submission history
- [ ] Allow member to make required corrections
- [ ] Allow new progress where applicable
- [ ] Allow required evidence updates where permitted
- [ ] Submit corrected activity
- [ ] Preserve previous review history
- [ ] Refresh review state after resubmission

---

### Verified State

- [ ] Display Verified status
- [ ] Display administrator remark where available
- [ ] Keep final submission read-only
- [ ] Keep submitted evidence viewable
- [ ] Keep review history viewable
- [ ] Remove further submission actions

---

## Activity History

- [ ] Create member Activity History page
- [ ] Use documented Activity History endpoint
- [ ] Display history chronologically
- [ ] Display activity category
- [ ] Display activity type
- [ ] Display title
- [ ] Display description
- [ ] Display timestamp
- [ ] Render documented metadata
- [ ] Support documented pagination
- [ ] Keep Activity History read-only

---

## Activity Templates

### Template Management

- [ ] Create administrator Template Management page
- [ ] Display available templates
- [ ] Display template details
- [ ] Search templates
- [ ] Filter templates where supported
- [ ] Display empty state

---

### Template Creation

- [ ] Create template creation form
- [ ] Validate template information
- [ ] Submit template
- [ ] Display backend validation errors
- [ ] Prevent duplicate submissions
- [ ] Refresh template list after creation

---

### Template Editing

- [ ] Support editing templates where permitted
- [ ] Pre-populate template information
- [ ] Validate changes
- [ ] Submit updates
- [ ] Refresh template state after success

---

### Apply Template

- [ ] Create Apply Template workflow
- [ ] Display template preview
- [ ] Display generated activity information before confirmation where supported
- [ ] Submit template application request
- [ ] Display progress
- [ ] Refresh activity list after success
- [ ] Treat generated activities as independent records

---

## Activity Reporting

### Activity Reports

- [ ] Display activity reporting options
- [ ] Request backend-generated activity reports
- [ ] Display report generation state
- [ ] Display report availability
- [ ] Download completed reports
- [ ] Handle generation failures
- [ ] Handle download failures

---

### Report Integrity

- [ ] Verify frontend uses backend-generated report data
- [ ] Do not calculate authoritative activity statistics locally
- [ ] Verify downloaded report matches backend response
- [ ] Support CSV-based filtering outside the application where applicable

---

## Activity Notifications

- [ ] Support activity assignment notifications
- [ ] Display assignment notification clearly
- [ ] Navigate assignment notification to related activity
- [ ] Display review-related notification only where supported by approved API behavior
- [ ] Refresh relevant activity state after notification navigation

---

## Activity Loading & Error States

- [ ] Create activity list loading state
- [ ] Create activity detail loading state
- [ ] Create review queue loading state
- [ ] Create template loading state
- [ ] Create evidence upload progress state
- [ ] Handle activity loading failures
- [ ] Handle assignment conflicts
- [ ] Handle evidence failures
- [ ] Handle submission conflicts
- [ ] Handle review conflicts
- [ ] Provide recovery actions where appropriate

---

## Activity Performance

- [ ] Lazy load Activity Management module
- [ ] Lazy load Activity Templates module
- [ ] Lazy load Review Queue module
- [ ] Lazy load Activity Reports module
- [ ] Lazy load large evidence previews where appropriate
- [ ] Optimize activity list rendering
- [ ] Optimize evidence gallery rendering
- [ ] Avoid unnecessary activity refetches
- [ ] Configure appropriate TanStack Query caching
- [ ] Verify performance with large activity datasets

---

## Activity Accessibility

- [ ] Verify keyboard navigation
- [ ] Verify activity status is not communicated by color alone
- [ ] Verify priority is not communicated by color alone
- [ ] Verify evidence controls are accessible
- [ ] Verify video controls are keyboard accessible
- [ ] Verify upload progress is announced appropriately
- [ ] Verify review feedback is accessible
- [ ] Verify timeline reading order
- [ ] Verify focus management after dialogs and submissions

---

## Activity Testing

### Unit Tests

- [ ] Test Activity components
- [ ] Test Assignment components
- [ ] Test Progress components
- [ ] Test Evidence components
- [ ] Test Review components
- [ ] Test Template components
- [ ] Test Activity hooks
- [ ] Test Activity validation

---

### Integration Tests

- [ ] Test activity creation
- [ ] Test activity publishing
- [ ] Test individual assignment
- [ ] Test bulk assignment
- [ ] Test assignment conflict handling
- [ ] Test activity start
- [ ] Test progress submission
- [ ] Test evidence upload
- [ ] Test activity submission
- [ ] Test Needs Changes workflow
- [ ] Test corrected resubmission
- [ ] Test verification
- [ ] Test template application
- [ ] Test activity reporting
- [ ] Test Activity History
- [ ] Test offline synchronization

---

### Permission Tests

- [ ] Verify members cannot create activities
- [ ] Verify members cannot assign volunteers
- [ ] Verify members cannot review submissions
- [ ] Verify members cannot manage templates
- [ ] Verify members cannot access another member's assignments
- [ ] Verify administrators can access approved management workflows
- [ ] Verify unauthorized responses are handled correctly

---

## Activity Final Verification

Before completing Activity Management implementation:

- [ ] Verify UI matches `frontend_ui_spec.md`
- [ ] Verify routing matches `frontend_routing.md`
- [ ] Verify state behavior matches `frontend_state_management.md`
- [ ] Verify components follow `frontend_component_guidelines.md`
- [ ] Verify visual behavior follows `frontend_design_system.md`
- [ ] Verify implementation follows `rules_frontend.md`
- [ ] Verify all requests match `API_contract.md`
- [ ] Verify backend remains authoritative
- [ ] Verify Activity Layer tests pass
- [ ] Verify existing attendance functionality remains unaffected

# 7. Reporting

Reporting provides administrators with clear summaries while relying entirely on backend-generated data.

---

## Attendance Reports

- [ ] Display attendance summaries
- [ ] Display participant statistics
- [ ] Display attendance outcomes
- [ ] Display attendance timelines
- [ ] Display event summaries
- [ ] Display report filters

---

## Report Details

- [ ] Display detailed report information
- [ ] Display participant breakdown
- [ ] Display attendance history
- [ ] Display backend-generated metrics
- [ ] Support pagination for large reports

---

## Report Export

- [ ] Export supported report formats
- [ ] Display export progress
- [ ] Handle export failures
- [ ] Display successful export confirmation
- [ ] Verify exported data integrity

---

# 8. Notifications

Notifications keep members and administrators informed of important system events.

---

## Notification Center

- [ ] Create Notification Center
- [ ] Display notification list
- [ ] Display unread notifications
- [ ] Mark notifications as read
- [ ] Support notification filtering
- [ ] Support notification search

---

## Notification Types

- [ ] Attendance notifications
- [ ] Event notifications
- [ ] Administrative announcements
- [ ] Activity notifications
- [ ] Approval notifications
- [ ] Reminder notifications
- [ ] System notifications

---

## Notification Experience

- [ ] Display toast notifications
- [ ] Display persistent notifications when required
- [ ] Support notification badges
- [ ] Group related notifications
- [ ] Navigate users to relevant pages

---

# 9. Administrative Dashboard

Administrative interfaces should provide operational visibility while remaining responsive and easy to navigate.

---

## Dashboard Overview

- [ ] Display summary cards
- [ ] Display active events
- [ ] Display attendance overview
- [ ] Display monitoring overview
- [ ] Display pending administrative actions
- [ ] Display recent activity

---

## Event Administration

- [ ] View event details
- [ ] Manage event lifecycle
- [ ] Monitor attendance
- [ ] Review participant information
- [ ] Access event reports

---

## Attendance Monitoring

- [ ] Display live attendance summary
- [ ] Display participant statuses
- [ ] Display members requiring attention
- [ ] Display attendance timeline
- [ ] Refresh monitoring information

---

## Administrative Actions

- [ ] Review pending requests
- [ ] Display supporting information
- [ ] Approve eligible requests
- [ ] Reject requests with backend-supported responses
- [ ] Refresh administrative data after actions

---

## Dashboard Performance

- [ ] Lazy load dashboard modules
- [ ] Optimize large tables
- [ ] Optimize filtering
- [ ] Optimize searching
- [ ] Minimize unnecessary refresh operations
- [ ] Verify dashboard responsiveness on large datasets

---

# 10. Quality Assurance

Quality assurance ensures that every frontend feature is reliable, maintainable, and consistent with the approved architecture.

---

## User Experience Validation

- [ ] Verify all pages follow the design system
- [ ] Verify consistent spacing and typography
- [ ] Verify responsive layouts
- [ ] Verify consistent navigation
- [ ] Verify reusable component usage
- [ ] Verify loading states across the application
- [ ] Verify empty states
- [ ] Verify success and error feedback

---

## Error Handling

- [ ] Verify network error handling
- [ ] Verify authentication failures
- [ ] Verify authorization failures
- [ ] Verify validation errors
- [ ] Verify API timeout handling
- [ ] Verify unexpected backend failures
- [ ] Verify graceful recovery where appropriate

---

## Performance

- [ ] Enable route-based code splitting
- [ ] Lazy load large modules
- [ ] Optimize images and static assets
- [ ] Minimize unnecessary component re-renders
- [ ] Optimize React Query caching
- [ ] Verify bundle size
- [ ] Verify Core Web Vitals
- [ ] Verify dashboard performance with large datasets

---

## Accessibility

- [ ] Verify keyboard navigation
- [ ] Verify visible focus indicators
- [ ] Verify screen reader compatibility
- [ ] Verify semantic HTML usage
- [ ] Verify accessible form validation
- [ ] Verify icon accessibility
- [ ] Verify dialog accessibility
- [ ] Verify notification accessibility
- [ ] Verify responsive accessibility

---

## Browser Compatibility

- [ ] Test latest Chrome
- [ ] Test latest Firefox
- [ ] Test latest Safari
- [ ] Test latest Microsoft Edge
- [ ] Test Android Chrome
- [ ] Test iOS Safari
- [ ] Verify camera functionality where required
- [ ] Verify location services where required
- [ ] Verify responsive layouts across devices

---

# 11. Testing

Testing should validate both individual components and complete user workflows.

---

## Unit Testing

- [ ] Test reusable components
- [ ] Test custom hooks
- [ ] Test utility functions
- [ ] Test form validation
- [ ] Test shared services

---

## Integration Testing

- [ ] Verify authentication workflow
- [ ] Verify event management workflow
- [ ] Verify attendance workflow
- [ ] Verify presence monitoring workflow
- [ ] Verify activity workflow
- [ ] Verify reporting workflow
- [ ] Verify notification workflow
- [ ] Verify administrator workflow

---

## End-to-End Validation

- [ ] Verify complete member journey
- [ ] Verify complete administrator journey
- [ ] Verify frontend and backend integration
- [ ] Verify API contract compliance
- [ ] Verify route protection
- [ ] Verify permission handling
- [ ] Verify application recovery after failures

---

## Regression Testing

- [ ] Verify existing functionality after changes
- [ ] Verify shared components remain compatible
- [ ] Verify API integrations remain functional
- [ ] Verify reusable hooks remain stable
- [ ] Verify navigation remains consistent
- [ ] Verify dashboard functionality

---

# 12. Documentation & Project Maintenance

Documentation should evolve together with implementation.

Whenever frontend functionality changes, review whether project documentation also requires updates.

---

## Documentation Review

- [ ] Verify API contract alignment
- [ ] Verify frontend UI specification
- [ ] Verify component guidelines
- [ ] Verify design system
- [ ] Verify routing documentation
- [ ] Verify state management documentation
- [ ] Verify frontend rules
- [ ] Verify system architecture references

---

## Project Tracking

- [ ] Update project progress
- [ ] Update changelog
- [ ] Record implementation notes where appropriate
- [ ] Record approved enhancement proposals
- [ ] Archive completed roadmap items

---

## Final Verification

Before completing a major feature or milestone, verify:

- [ ] Documentation is current
- [ ] Tests pass
- [ ] Accessibility requirements are satisfied
- [ ] Performance remains acceptable
- [ ] API contract compliance is maintained
- [ ] Backend integration is verified
- [ ] UI consistency is preserved
- [ ] Code review feedback has been addressed

---

# Completion Guidelines

A roadmap item should only be marked complete when:

- The implementation is finished.
- The feature complies with the approved API contract.
- The implementation follows the frontend architecture.
- Appropriate testing has been completed.
- Documentation has been updated where necessary.
- The feature is ready for review or release.

Avoid marking work as complete based solely on implementation. A feature is considered complete only after it satisfies the project's quality, testing, and documentation standards.

---

# Roadmap Principles

This roadmap is intended to be a living development document.

As the project evolves:

- Add new functional areas instead of creating separate phase-specific roadmaps.
- Archive completed work rather than deleting it.
- Keep tasks aligned with the latest approved architecture and API contract.
- Update roadmap items whenever new approved features are introduced.
- Maintain a feature-oriented structure to support long-term project growth.

The roadmap should remain the primary reference for frontend implementation progress and should evolve alongside the project's documentation and architecture.

# Phase 3 — Activity Management

Phase 3 introduces the frontend Activity Layer.

Implementation should follow the milestones below in order. Each milestone should be completed, tested, and verified against the approved documentation before moving to the next milestone.

---

# Milestone 12 — Activity Foundation & Administration

## Activity Foundation

- [ ] Add Activity Layer routes
- [ ] Add Activity API integration
- [ ] Define Activity TypeScript types
- [ ] Define Assignment TypeScript types
- [ ] Define Progress TypeScript types
- [ ] Define Evidence TypeScript types
- [ ] Define Review TypeScript types
- [ ] Define Template TypeScript types
- [ ] Configure Activity-related TanStack Query keys
- [ ] Add Activity loading states
- [ ] Add Activity error states
- [ ] Add Activity empty states

---

## Activity Listing

- [ ] Create Activity Management page
- [ ] Display activities
- [ ] Display activity status
- [ ] Display activity priority
- [ ] Display activity category
- [ ] Display associated event
- [ ] Display assignment summary
- [ ] Support search
- [ ] Filter by status
- [ ] Filter by priority
- [ ] Filter by category
- [ ] Filter by event where supported
- [ ] Support pagination
- [ ] Preserve filters during navigation where appropriate

---

## Activity Creation

- [ ] Create Activity Creation page
- [ ] Build activity creation form
- [ ] Validate required fields
- [ ] Display backend validation errors
- [ ] Prevent duplicate submissions
- [ ] Display creation progress
- [ ] Handle successful creation
- [ ] Navigate to appropriate activity view after creation

Activity creation must only be available to authorized administrators.

---

## Activity Details

- [ ] Create administrator Activity Details page
- [ ] Display activity information
- [ ] Display activity status
- [ ] Display activity priority
- [ ] Display category
- [ ] Display associated event
- [ ] Display assignments
- [ ] Display activity timeline
- [ ] Display submission summary
- [ ] Display evidence summary
- [ ] Display review information
- [ ] Display only backend-supported actions

---

## Activity Editing

- [ ] Support editing where permitted
- [ ] Pre-populate existing activity information
- [ ] Validate updated information
- [ ] Display backend validation errors
- [ ] Prevent duplicate update requests
- [ ] Refresh activity state after successful update

---

## Activity Lifecycle

- [ ] Display backend-provided lifecycle state
- [ ] Implement Publish action
- [ ] Implement Cancel action
- [ ] Implement Archive action
- [ ] Display confirmation for destructive actions
- [ ] Disable duplicate lifecycle requests
- [ ] Refresh activity state after lifecycle changes
- [ ] Handle invalid state transitions returned by backend

The frontend must never determine Activity lifecycle transitions independently.

---

## Milestone 12 Verification

- [ ] Verify administrator-only Activity management
- [ ] Verify Activity routes
- [ ] Verify API requests match `API_contract.md`
- [ ] Verify search and filters
- [ ] Verify pagination
- [ ] Verify loading, empty, and error states
- [ ] Verify lifecycle actions
- [ ] Run relevant tests
- [ ] Cross-check implementation with documentation

---

# Milestone 13 — Activity Assignment

## Volunteer Assignment

- [ ] Create assignment interface
- [ ] Display eligible volunteers returned by backend
- [ ] Display currently assigned volunteers
- [ ] Display assignment status
- [ ] Support individual assignment
- [ ] Prevent accidental duplicate assignment requests
- [ ] Refresh assignment state after successful changes

---

## Bulk Assignment

- [ ] Support selecting multiple eligible volunteers
- [ ] Display selected volunteer count
- [ ] Support approved bulk assignment operation
- [ ] Clearly display bulk actions
- [ ] Confirm potentially destructive bulk operations
- [ ] Display backend results accurately
- [ ] Handle partial failures where supported

---

## Assignment Conflicts

- [ ] Detect backend assignment conflict responses
- [ ] Display assignment conflict clearly
- [ ] Display backend-provided conflict reason
- [ ] Block conflicting assignment until resolved
- [ ] Preserve unaffected selections where appropriate
- [ ] Allow administrator to resolve conflict and retry
- [ ] Never bypass backend conflict validation

---

## Late-Joining Members

- [ ] Display late-joining members where applicable
- [ ] Do not automatically assign existing work
- [ ] Allow administrator to determine appropriate assignment
- [ ] Display only backend-confirmed assignments

---

## Assignment Notifications

- [ ] Support activity-assignment notifications
- [ ] Clearly identify the assigned activity
- [ ] Navigate notification to the related activity
- [ ] Refresh assignment information after navigation where required

---

## Assignment Permissions

- [ ] Verify members cannot assign activities
- [ ] Verify members cannot self-assign activities
- [ ] Verify members cannot reassign themselves
- [ ] Verify unauthorized responses are handled correctly

---

## Milestone 13 Verification

- [ ] Test individual assignment
- [ ] Test bulk assignment
- [ ] Test assignment conflicts
- [ ] Test late-joining member workflow
- [ ] Test assignment notification navigation
- [ ] Test assignment permissions
- [ ] Run relevant tests
- [ ] Cross-check implementation with documentation

---

# Milestone 14 — Member Activities & Progress Tracking

## My Activities

- [ ] Create My Activities page
- [ ] Display assigned activities
- [ ] Display activity title
- [ ] Display associated event
- [ ] Display priority
- [ ] Display assignment status
- [ ] Display latest progress
- [ ] Group or filter assignments by status
- [ ] Support pagination where required
- [ ] Display empty state when no activities are assigned
- [ ] Display loading state
- [ ] Display error state

---

## Member Activity Details

- [ ] Create member Activity Details page
- [ ] Display activity instructions
- [ ] Display activity information
- [ ] Display priority
- [ ] Display assignment status
- [ ] Display submission requirements
- [ ] Display progress timeline
- [ ] Display evidence
- [ ] Display review feedback
- [ ] Display only currently permitted actions

---

## Activity Start

- [ ] Implement Activity start workflow where required by API
- [ ] Display backend-confirmed state
- [ ] Prevent unsupported activity abandonment
- [ ] Prevent unsupported self-reassignment
- [ ] Refresh assignment state after successful start

Once a member starts an assigned activity, the frontend must not provide unsupported abandon or self-switch actions.

---

## Progress Updates

- [ ] Create progress update interface
- [ ] Allow member to enter progress information
- [ ] Validate required fields
- [ ] Submit progress through approved API
- [ ] Prevent duplicate submissions
- [ ] Display synchronization status
- [ ] Refresh progress history after successful synchronization

---

## Progress Timeline

- [ ] Create reusable Activity Progress Timeline
- [ ] Display progress updates chronologically
- [ ] Display timestamps
- [ ] Display descriptions
- [ ] Display associated evidence
- [ ] Preserve historical entries
- [ ] Display loading state
- [ ] Display empty state

Historical progress entries remain read-only.

---

## Activity History

- [ ] Create member Activity History page
- [ ] Use documented Activity History endpoint
- [ ] Display history chronologically
- [ ] Display activity category
- [ ] Display activity type
- [ ] Display title
- [ ] Display description
- [ ] Display timestamp
- [ ] Render documented metadata
- [ ] Support documented pagination
- [ ] Keep Activity History read-only

---

## Milestone 14 Verification

- [ ] Test My Activities
- [ ] Test Activity Details
- [ ] Test Activity start
- [ ] Test progress submission
- [ ] Test Progress Timeline
- [ ] Test Activity History
- [ ] Verify member can access only authorized assignments
- [ ] Run relevant tests
- [ ] Cross-check implementation with documentation

---

# Milestone 15 — Evidence & Offline Synchronization

## Evidence Capture

- [ ] Create evidence capture/upload interface
- [ ] Support approved photo capture
- [ ] Support approved video capture
- [ ] Display evidence requirements before capture
- [ ] Display selected evidence previews
- [ ] Allow removal before successful submission where permitted

---

## Evidence Limits

- [ ] Enforce basic client-side maximum of 10 photographs
- [ ] Enforce basic client-side maximum of 2 videos
- [ ] Validate maximum video duration of 60 seconds
- [ ] Display current photo count
- [ ] Display current video count
- [ ] Display backend validation failures

Backend validation remains authoritative.

---

## Evidence Upload

- [ ] Display upload progress
- [ ] Display individual upload state
- [ ] Distinguish pending evidence from synchronized evidence
- [ ] Handle upload failures
- [ ] Support retry where appropriate
- [ ] Never display failed evidence as synchronized
- [ ] Refresh backend evidence state after successful upload

---

## Evidence Gallery

- [ ] Create reusable Evidence Gallery
- [ ] Display photo previews
- [ ] Display video previews
- [ ] Display evidence type
- [ ] Display synchronization state where relevant
- [ ] Support accessible media controls
- [ ] Support responsive layouts

---

## Offline Pending State

- [ ] Detect connectivity loss
- [ ] Preserve supported pending progress updates locally
- [ ] Preserve supported pending evidence locally
- [ ] Clearly display pending synchronization state
- [ ] Avoid representing local data as server-confirmed data

---

## Synchronization

- [ ] Detect restored connectivity
- [ ] Automatically attempt synchronization
- [ ] Display synchronization progress
- [ ] Refresh backend state after successful synchronization
- [ ] Handle synchronization failures
- [ ] Allow retry where appropriate
- [ ] Prevent duplicate synchronization

---

## Lost Evidence

- [ ] Detect when locally referenced evidence is unavailable where possible
- [ ] Inform member that lost evidence must be captured again
- [ ] Do not create placeholder evidence
- [ ] Do not mark missing evidence as submitted

---

## Milestone 15 Verification

- [ ] Test photo evidence
- [ ] Test video evidence
- [ ] Test evidence limits
- [ ] Test upload progress
- [ ] Test failed uploads
- [ ] Test offline pending state
- [ ] Test reconnect synchronization
- [ ] Test lost evidence behavior
- [ ] Run relevant tests
- [ ] Cross-check implementation with documentation

---

# Milestone 16 — Submission, Review & Resubmission

## Submission Preparation

- [ ] Display submission summary
- [ ] Display progress history
- [ ] Display evidence summary
- [ ] Display required submission information
- [ ] Validate basic submission requirements
- [ ] Provide final submission confirmation

---

## Activity Submission

- [ ] Submit completed activity for review
- [ ] Display submission progress
- [ ] Prevent duplicate submissions
- [ ] Wait for backend confirmation
- [ ] Display successful submission confirmation
- [ ] Handle backend validation errors
- [ ] Refresh assignment state after submission

---

## Submitted State

- [ ] Make completed submission read-only
- [ ] Remove editing controls
- [ ] Remove evidence deletion controls
- [ ] Preserve evidence viewing
- [ ] Preserve progress viewing
- [ ] Display review status

---

## Administrator Review Queue

- [ ] Create administrator Review Queue
- [ ] Display submissions awaiting review
- [ ] Display member information
- [ ] Display activity information
- [ ] Display submission time
- [ ] Display review status
- [ ] Support documented filtering
- [ ] Support documented sorting
- [ ] Support pagination
- [ ] Display empty state when no reviews are pending

---

## Review Details

- [ ] Create Activity Review page
- [ ] Display submitted progress
- [ ] Display submitted evidence
- [ ] Display previous review history
- [ ] Display member information
- [ ] Display activity information
- [ ] Display review controls

---

## Needs Changes

- [ ] Implement `NEEDS_CHANGES` action
- [ ] Require administrator remarks
- [ ] Validate remarks before submission
- [ ] Display backend validation errors
- [ ] Refresh review state after success
- [ ] Display remarks prominently to member
- [ ] Allow member to continue the activity
- [ ] Allow corrected resubmission

There is no separate Activity Review `REJECTED` workflow.

---

## Resubmission

- [ ] Display previous administrator remarks
- [ ] Preserve previous submission history
- [ ] Allow member to make required corrections
- [ ] Allow new progress where applicable
- [ ] Allow required evidence updates where permitted
- [ ] Submit corrected activity
- [ ] Preserve previous review history
- [ ] Refresh review state after resubmission

---

## Verification

- [ ] Implement `VERIFIED` action
- [ ] Support administrator completion remark
- [ ] Display confirmation
- [ ] Prevent duplicate review actions
- [ ] Refresh review and assignment state after success
- [ ] Display Verified state to member

---

## Verified State

- [ ] Display Verified status
- [ ] Display administrator remark where available
- [ ] Keep final submission read-only
- [ ] Keep submitted evidence viewable
- [ ] Keep review history viewable
- [ ] Remove further submission actions

---

## Milestone 16 Verification

- [ ] Test initial submission
- [ ] Test submitted read-only state
- [ ] Test Review Queue
- [ ] Test `NEEDS_CHANGES`
- [ ] Test mandatory administrator remarks
- [ ] Test member correction workflow
- [ ] Test resubmission
- [ ] Test `VERIFIED`
- [ ] Verify no Activity Review `REJECTED` workflow exists
- [ ] Test permissions
- [ ] Run relevant tests
- [ ] Cross-check implementation with documentation

---

# Milestone 17 — Templates, Reports & Phase 3 Final Verification

## Activity Templates

- [ ] Create administrator Template Management page
- [ ] Display available templates
- [ ] Display template details
- [ ] Search templates
- [ ] Filter templates where supported
- [ ] Display empty state

---

## Template Creation

- [ ] Create template creation form
- [ ] Validate template information
- [ ] Submit template
- [ ] Display backend validation errors
- [ ] Prevent duplicate submissions
- [ ] Refresh template list after creation

---

## Template Editing

- [ ] Support editing templates where permitted
- [ ] Pre-populate template information
- [ ] Validate changes
- [ ] Submit updates
- [ ] Refresh template state after success

---

## Apply Template

- [ ] Create Apply Template workflow
- [ ] Display template preview
- [ ] Display generated activity information before confirmation where supported
- [ ] Submit template application request
- [ ] Display progress
- [ ] Refresh activity list after success
- [ ] Treat generated activities as independent records

---

## Activity Reports

- [ ] Display activity reporting options
- [ ] Request backend-generated Activity reports
- [ ] Display report generation state
- [ ] Display report availability
- [ ] Download completed reports
- [ ] Handle generation failures
- [ ] Handle download failures
- [ ] Do not calculate authoritative Activity statistics locally

---

## Performance

- [ ] Lazy load Activity Management where appropriate
- [ ] Lazy load Activity Templates
- [ ] Lazy load Review Queue
- [ ] Lazy load Activity Reports
- [ ] Lazy load large evidence previews where appropriate
- [ ] Optimize Activity list rendering
- [ ] Optimize Evidence Gallery rendering
- [ ] Avoid unnecessary Activity refetches
- [ ] Configure appropriate TanStack Query caching
- [ ] Verify performance with large Activity datasets

---

## Accessibility

- [ ] Verify keyboard navigation
- [ ] Verify Activity status is not communicated by color alone
- [ ] Verify priority is not communicated by color alone
- [ ] Verify evidence controls are accessible
- [ ] Verify video controls are keyboard accessible
- [ ] Verify upload progress is announced appropriately
- [ ] Verify review feedback is accessible
- [ ] Verify timeline reading order
- [ ] Verify focus management after dialogs and submissions

---

## Unit Tests

- [ ] Test Activity components
- [ ] Test Assignment components
- [ ] Test Progress components
- [ ] Test Evidence components
- [ ] Test Review components
- [ ] Test Template components
- [ ] Test Activity hooks
- [ ] Test Activity validation

---

## Integration Tests

- [ ] Test activity creation
- [ ] Test activity publishing
- [ ] Test individual assignment
- [ ] Test bulk assignment
- [ ] Test assignment conflict handling
- [ ] Test activity start
- [ ] Test progress submission
- [ ] Test evidence upload
- [ ] Test activity submission
- [ ] Test Needs Changes workflow
- [ ] Test corrected resubmission
- [ ] Test verification
- [ ] Test template application
- [ ] Test Activity reporting
- [ ] Test Activity History
- [ ] Test offline synchronization

---

## Permission Tests

- [ ] Verify members cannot create activities
- [ ] Verify members cannot assign volunteers
- [ ] Verify members cannot review submissions
- [ ] Verify members cannot manage templates
- [ ] Verify members cannot access another member's assignments
- [ ] Verify administrators can access approved management workflows
- [ ] Verify unauthorized responses are handled correctly

---

## Phase 3 Regression Testing

- [ ] Verify Phase 1 authentication remains functional
- [ ] Verify existing Event workflows remain functional
- [ ] Verify existing Attendance workflows remain functional
- [ ] Verify Phase 2 Presence Monitoring remains functional
- [ ] Verify existing Reporting remains functional
- [ ] Verify existing Notifications remain functional
- [ ] Verify Activity Layer changes do not break previous functionality

---

## Phase 3 Final Verification

Before Phase 3 frontend implementation is considered complete:

- [ ] Verify UI matches `frontend_ui_spec.md`
- [ ] Verify routing matches `frontend_routing.md`
- [ ] Verify state behavior matches `frontend_state_management.md`
- [ ] Verify components follow `frontend_component_guidelines.md`
- [ ] Verify visual behavior follows `frontend_design_system.md`
- [ ] Verify implementation follows `rules_frontend.md`
- [ ] Verify all requests match `API_contract.md`
- [ ] Verify backend remains authoritative
- [ ] Verify all Phase 3 frontend tests pass
- [ ] Verify existing functionality remains unaffected
- [ ] Cross-check implementation against Repository A documentation

