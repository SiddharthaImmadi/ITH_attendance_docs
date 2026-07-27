# frontend_design_system.md — Design System & Styling

> This document defines the visual language, design tokens, layout standards, and user interface principles for the InnoTech Hub Attendance System. It provides a consistent foundation for every frontend interface while remaining independent of implementation details and frontend frameworks.

---

# 1. Color Palette

A consistent color system improves usability, accessibility, and recognition across the application.

Colors should communicate interface hierarchy before they communicate status.

Status colors should always be accompanied by text or icons and must never be the sole indicator of meaning.

---

## Primary Colors

| Purpose | Color | Hex | Tailwind |
|----------|-------|-----|-----------|
| Primary | Blue | #3B82F6 | `blue-500` |
| Primary Hover | Blue | #2563EB | `blue-600` |
| Secondary | Gray | #6B7280 | `gray-500` |
| Background | Light Gray | #F9FAFB | `gray-50` |
| Surface | White | #FFFFFF | `white` |
| Border | Gray | #E5E7EB | `gray-200` |
| Divider | Gray | #D1D5DB | `gray-300` |

---

## Semantic Colors

| Purpose | Color | Tailwind |
|----------|-------|-----------|
| Success | Emerald | `emerald-500` |
| Warning | Amber | `amber-500` |
| Error | Red | `red-500` |
| Information | Indigo | `indigo-500` |
| Neutral | Gray | `gray-500` |

---

## Text Colors

| Usage | Tailwind |
|--------|----------|
| Primary Text | `gray-800` |
| Secondary Text | `gray-600` |
| Muted Text | `gray-500` |
| Disabled Text | `gray-400` |
| Inverse Text | `white` |

---

## Attendance & Presence Status Colors

The application contains multiple attendance and monitoring workflows. Status colors should remain consistent throughout every screen.

| Status | Recommended Color |
|----------|-------------------|
| Event Active | Blue |
| Attendance Confirmed | Emerald |
| Presence OK | Emerald |
| Returned | Emerald |
| Late | Amber |
| Pending Review | Amber |
| Emergency Pending | Amber |
| Validation Failed | Red |
| Left Venue | Red |
| Emergency Rejected | Red |
| Volunteer Blocked | Red |
| Completed | Gray |

Status styling should remain visually consistent across:

- Cards
- Tables
- Timelines
- Badges
- Notifications
- Reports

---

## Usage Guidelines

Primary color should be used for:

- Primary buttons
- Navigation highlights
- Active navigation
- Links
- Focus indicators

Success colors indicate successful operations such as:

- Attendance confirmed
- Presence confirmed
- Successful actions
- Completed workflows

Warning colors indicate situations requiring user attention, including:

- Pending administrator review
- Late attendance
- Pending emergency requests

Error colors indicate failed operations or important alerts including:

- Validation failures
- Leaving the approved event boundary
- Administrative restrictions

Neutral colors should be used for informational content, secondary actions, and completed workflows.

---

# 2. Typography

Typography establishes visual hierarchy and improves readability across the application.

The design system uses a clean sans-serif typeface suitable for dashboards, forms, tables, and reports.

---

## Font Family

Primary Font:

**Inter**

Fallback:

- System UI
- Segoe UI
- Roboto
- Helvetica
- Arial
- sans-serif

---

## Typography Scale

| Element | Size | Weight | Line Height |
|----------|------|---------|-------------|
| Display | 36px | 700 | 1.2 |
| Page Title | 32px | 700 | 1.2 |
| Section Title | 24px | 600 | 1.3 |
| Card Title | 18px | 600 | 1.4 |
| Body | 14px | 400 | 1.6 |
| Caption | 12px | 500 | 1.4 |
| Button Text | 14px | 500 | 1.4 |

---

## Typography Principles

Typography should:

- establish clear hierarchy;
- remain consistent across pages;
- avoid excessive font weights;
- maintain sufficient spacing between headings and content;
- support accessibility requirements.

Avoid using more than three font weights on a single screen.

---

# 3. Spacing Scale

Consistent spacing improves readability and creates predictable layouts.

The application follows an 8-point spacing system built on Tailwind's spacing utilities.

---

## Spacing Tokens

| Token | Pixels | Typical Usage |
|--------|---------|---------------|
| xs | 4px | Icons, tight spacing |
| sm | 8px | Compact controls |
| md | 16px | Standard padding |
| lg | 24px | Card spacing |
| xl | 32px | Section spacing |
| 2xl | 48px | Page spacing |
| 3xl | 64px | Large layout separation |

---

## Layout Guidelines

Standard page padding:

- Horizontal: 24px
- Vertical: 32px

Card padding:

- 24px

Modal padding:

- 24px

Table cell padding:

- 16px

Sidebar spacing:

- 16–24px

Header spacing:

- 24px

---

## Component Spacing

Use consistent spacing between:

- labels and inputs;
- cards within grids;
- page sections;
- toolbar actions;
- navigation items;
- dashboard widgets.

Avoid introducing arbitrary spacing values that are inconsistent with the design system.

---

# 4. Layout & Component Sizing

The application follows a responsive dashboard layout optimized primarily for desktop usage while remaining usable on tablets and mobile devices.

---

## Desktop Layout

Desktop layout consists of four primary regions.

- Sidebar
- Header
- Main Content
- Optional Right Panel (future expansion)

```
+-------------------------------------------------------------+
| Sidebar | Header                                            |
|         |---------------------------------------------------|
|         |                                                   |
|         |               Main Content                        |
|         |                                                   |
|         |                                                   |
+-------------------------------------------------------------+
```

---

## Responsive Behavior

### Desktop

- Fixed sidebar
- Multi-column layouts
- Persistent navigation

### Tablet

- Collapsible sidebar
- Two-column layouts where appropriate

### Mobile

- Drawer navigation
- Single-column layouts
- Full-width content

---

## Component Sizes

| Component | Recommended Size |
|-----------|------------------|
| Input | Height: 40px |
| Primary Button | Height: 40px |
| Secondary Button | Height: 40px |
| Search Bar | Height: 40px |
| Status Badge | Auto |
| Card | Responsive |
| Modal | Maximum width: 600px |
| Drawer | Full height |
| Sidebar | 260px expanded / 72px collapsed |

---

## Card Principles

Cards are the primary information container throughout the application.

Every card should present information in the following order:

1. Title
2. Status
3. Primary Information
4. Secondary Information
5. Actions

Cards should maintain consistent spacing, border radius, and elevation across all modules.

Typical examples include:

- Event Summary
- Attendance Summary
- Presence Overview
- Emergency Ticket
- Volunteer Information
- Report Summary
- Notification Summary

---

## Grid Layout

Dashboard content should use responsive grids.

Typical layouts:

- 1 column (mobile)
- 2 columns (tablet)
- 3–4 columns (desktop)

Cards within the same row should maintain equal heights whenever practical.

---

## Design Principles

Layout should prioritize:

- readability;
- consistency;
- predictable navigation;
- minimal visual clutter;
- efficient information density;
- responsive behavior.

Every screen should maintain visual consistency regardless of feature area.

---

# 5. Button Design

Buttons represent the primary interaction mechanism throughout the application. Their appearance should clearly communicate importance and expected user actions.

---

## Button Hierarchy

Buttons are categorized into four levels.

| Type | Purpose |
|------|---------|
| Primary | Main page action |
| Secondary | Supporting actions |
| Destructive | Irreversible actions |
| Disabled | Unavailable actions |

Only one Primary button should normally exist within a logical action group.

---

## Primary Button

Used for the most important action on a page.

Examples:

- Check In
- Submit Attendance
- Save Event
- Create Event
- Approve Request
- Submit Emergency Ticket
- Generate Report

Recommended styling:

- Blue background
- White text
- Medium font weight
- Rounded corners
- Visible hover state
- Clear keyboard focus indicator

---

## Secondary Button

Used for supporting actions.

Examples:

- Cancel
- Back
- Close
- Edit
- View Details
- Export
- Filter

Secondary buttons should never visually compete with Primary buttons.

---

## Destructive Button

Reserved for irreversible operations.

Examples:

- Delete Event
- Remove Volunteer
- Reject Request
- Remove Attendance Record

Destructive actions should always require user confirmation.

---

## Disabled Button

Disabled buttons communicate that an action is currently unavailable.

Examples:

- Missing required information
- Insufficient permissions
- Event not yet started
- Processing in progress

Disabled buttons should remain readable while clearly indicating they cannot be interacted with.

---

## Button States

Every button should define the following interaction states.

- Default
- Hover
- Focus
- Active
- Loading
- Disabled

Visual feedback should remain consistent across the application.

---

## Loading Buttons

When an operation is in progress:

- disable repeated clicks;
- display a loading indicator;
- preserve the button size;
- keep the action label visible whenever practical.

Examples include:

- Creating an event
- Saving attendance
- Uploading data
- Exporting reports

---

# 6. Form Input Design

Forms are used throughout the application for authentication, event management, attendance workflows, emergency requests, and administrative operations.

Input components should prioritize clarity, accessibility, and consistency.

---

## Standard Input Fields

Common input types include:

- Text
- Email
- Password
- Number
- Date
- Time
- Search
- Text Area
- Select
- Multi-select

Each input should include:

- Label
- Placeholder (optional)
- Validation message
- Help text (when needed)

---

## Input States

Every input should support the following states.

### Default

Neutral border with readable text.

### Focus

Strong focus indicator using the primary color.

### Error

Clear validation styling using the error color.

Validation messages should explain how the user can resolve the issue.

### Disabled

Displayed when editing is not permitted.

---

## Validation Principles

Validation should be:

- immediate where appropriate;
- clear;
- actionable;
- consistent.

Avoid technical error messages.

Instead of:

> Invalid input.

Prefer:

> Event name cannot be empty.

---

## Search Inputs

Search fields are commonly used for:

- Events
- Users
- Attendance
- Reports
- Notifications

Search should remain responsive even for large datasets.

---

## Date & Time Selection

Date and time controls should:

- use consistent formatting;
- respect user locale when appropriate;
- clearly distinguish date from time;
- prevent invalid selections whenever possible.

---

## Dropdowns

Dropdowns are appropriate for selecting predefined values such as:

- Event
- Role
- Status
- Attendance Type
- Notification Category

Avoid excessively long dropdown lists.

Searchable dropdowns are preferred when many options exist.

---

## Form Layout

Recommended layout:

- Single-column on mobile
- Multi-column on larger screens where readability is maintained

Related inputs should be grouped together using consistent spacing.

---

# 7. Cards & Surface Components

Cards are the primary information containers throughout the application.

They should present information clearly while maintaining a clean and consistent appearance.

---

## Card Structure

Recommended hierarchy:

1. Header
2. Status
3. Primary Information
4. Supporting Information
5. Actions

This hierarchy should remain consistent across all card types.

---

## Common Card Types

Examples include:

- Event Summary
- Attendance Summary
- Presence Overview
- Emergency Ticket
- Volunteer Profile
- Notification
- Report Summary
- Activity Timeline

Although the displayed information differs, all cards should follow the same visual structure.

---

## Card Styling

Cards should include:

- White background
- Rounded corners
- Subtle border
- Light elevation
- Consistent internal spacing

Hover elevation may be used for interactive cards.

---

## Information Hierarchy

Important information should appear first.

For example:

- Event Name
- Current Status
- Date and Time
- Location
- Assigned Volunteers
- Available Actions

Supporting details should never compete visually with primary information.

---

## Card Actions

Actions should be placed consistently.

Recommended placement:

- Header actions
- Bottom action row
- Overflow menu for secondary operations

Avoid scattering actions throughout the card.

---

## Empty Cards

When no information exists, cards should communicate the situation clearly.

Example messages:

- No upcoming events.
- No attendance records available.
- No notifications found.
- No emergency requests pending.

Whenever possible, provide a clear next action.

---

# 8. Status Indicators & Feedback Components

Status indicators provide immediate visual understanding of system state.

They should always combine:

- Color
- Text
- Optional icon

Color alone must never communicate status.

---

## Attendance Status

Typical attendance states include:

| Status | Color |
|---------|-------|
| Checked In | Emerald |
| Active | Blue |
| Completed | Gray |
| Late | Amber |

---

## Presence Monitoring

Presence-related indicators include:

| Status | Color |
|---------|-------|
| Presence OK | Emerald |
| Left Venue | Red |
| Returned | Emerald |
| Monitoring Active | Blue |

These indicators should remain consistent across dashboards, timelines, and reports.

---

## Emergency Ticket Status

Emergency workflows use dedicated status indicators.

| Status | Color |
|---------|-------|
| Pending Review | Amber |
| Approved | Emerald |
| Rejected | Red |
| Closed | Gray |

---

## Volunteer Status

Administrative workflows may display volunteer availability.

Examples include:

- Available
- Assigned
- Restricted
- Blocked

Each state should use a consistent visual treatment across the application.

---

## Progress Indicators

Progress components communicate long-running operations.

Examples include:

- Event progress
- Attendance completion
- File uploads
- Report generation

Progress indicators should:

- animate smoothly;
- display percentages when meaningful;
- avoid abrupt visual changes.

---

## Timeline Indicators

Timeline entries should visually distinguish different event types while preserving chronological order.

Each timeline item should display:

- Status indicator
- Title
- Timestamp
- Optional description

Examples:

- Checked In
- Presence Confirmed
- Left Venue
- Returned
- Emergency Ticket Submitted
- Emergency Ticket Approved
- Attendance Completed

---

## Toast Notifications

Toast notifications communicate short-lived system feedback.

Typical examples:

- Attendance recorded successfully.
- Event created successfully.
- Report exported successfully.
- Emergency request submitted.
- Request approved.
- Request rejected.
- Changes saved.

Notifications should disappear automatically after a reasonable duration while remaining accessible to screen readers.

---

# 9. Responsive Design

The application is designed using a responsive, desktop-first approach while ensuring a consistent experience across tablets and mobile devices.

Responsive layouts should preserve functionality, readability, and navigation regardless of screen size.

---

## Breakpoints

The application follows Tailwind CSS default breakpoints.

| Device | Width | Typical Usage |
|----------|--------|----------------|
| Mobile | < 640px | Single-column layout |
| Small Tablet | ≥ 640px | Two-column layout where appropriate |
| Large Tablet | ≥ 768px | Expanded content areas |
| Desktop | ≥ 1024px | Full dashboard layout |
| Large Desktop | ≥ 1280px | Wider content grids |

---

## Layout Behavior

### Mobile

The mobile interface should prioritize simplicity.

Characteristics include:

- Single-column layouts
- Drawer navigation
- Full-width cards
- Large touch targets
- Vertical scrolling
- Simplified action groups

---

### Tablet

Tablet layouts should balance readability and information density.

Typical characteristics include:

- Collapsible sidebar
- Two-column grids
- Responsive tables
- Flexible dashboards

---

### Desktop

Desktop is the primary operating environment for administrators.

Desktop layouts should support:

- Persistent sidebar navigation
- Multi-column dashboards
- Large data tables
- Simultaneous information panels
- Efficient workflows

---

## Responsive Components

Every major component should adapt gracefully.

Examples include:

- Dashboard cards
- Data tables
- Forms
- Charts
- Reports
- Timelines
- Notifications
- Dialogs

No component should require horizontal scrolling except large administrative tables.

---

## Navigation

Navigation adapts according to screen size.

Desktop:

- Persistent sidebar
- Fixed header
- Breadcrumb support where appropriate

Tablet:

- Collapsible sidebar

Mobile:

- Drawer navigation
- Compact header
- Simplified action menu

Navigation behavior should remain predictable throughout the application.

---

## Tables

Administrative tables may contain large datasets.

Responsive tables should support:

- Horizontal scrolling when necessary
- Sticky headers
- Consistent column alignment
- Clear row separation
- Accessible keyboard navigation

Critical actions should remain visible whenever practical.

---

## Forms

Responsive forms should:

- Stack fields vertically on smaller devices
- Expand into multiple columns on larger displays
- Preserve logical field grouping
- Maintain comfortable touch spacing

---

## Dashboard Layout

Dashboard widgets should reorganize automatically according to available space.

Recommended progression:

Desktop

```
4 Cards
```

Tablet

```
2 Cards
2 Cards
```

Mobile

```
Card

Card

Card

Card
```

Widgets should never become unreadable due to aggressive resizing.

---

# 10. Elevation & Visual Hierarchy

Visual hierarchy helps users understand relationships between interface elements.

Elevation should be subtle and purposeful.

---

## Elevation Levels

| Level | Usage |
|--------|--------|
| Level 0 | Page background |
| Level 1 | Standard surfaces |
| Level 2 | Cards |
| Level 3 | Hovered interactive components |
| Level 4 | Dialogs |
| Level 5 | Critical overlays |

Avoid excessive shadows that distract from content.

---

## Borders

Borders provide structure without creating visual clutter.

Recommended usage:

- Card outlines
- Input fields
- Tables
- Dividers
- Panels

Borders should use neutral colors and consistent thickness.

---

## Rounded Corners

Corner radius should remain consistent across the application.

Suggested usage:

| Component | Radius |
|------------|---------|
| Buttons | Medium |
| Inputs | Medium |
| Cards | Large |
| Dialogs | Large |
| Badges | Pill |
| Avatars | Circular |

---

## Information Hierarchy

Visual emphasis should follow this order:

1. Page Title
2. Section Heading
3. Primary Content
4. Supporting Information
5. Metadata

Actions should remain visually distinct without overpowering content.

---

## Density

The interface should maintain a balance between information density and readability.

Avoid:

- excessive whitespace;
- overcrowded layouts;
- inconsistent spacing;
- unnecessary decoration.

The design should emphasize clarity over visual complexity.

---

# 11. Motion & Interaction Guidelines

Animations should improve usability by providing meaningful feedback and reinforcing user actions.

Motion should never delay task completion or distract users.

---

## Motion Principles

Animations should be:

- Fast
- Consistent
- Purposeful
- Predictable
- Accessible

Recommended duration:

150–300 milliseconds for most interactions.

---

## Appropriate Uses

Motion may be used for:

- Page transitions
- Card hover effects
- Sidebar expansion and collapse
- Dialog appearance
- Notification appearance
- Loading indicators
- Progress updates

Animations should communicate changes in interface state rather than serve decorative purposes.

---

## Hover Feedback

Interactive components should provide immediate hover feedback.

Examples include:

- Buttons
- Cards
- Navigation items
- Table rows

Hover effects should remain subtle and consistent.

---

## Loading States

Users should always receive feedback while waiting for asynchronous operations.

Preferred loading indicators include:

- Skeleton loaders
- Progress bars
- Inline loading indicators
- Button loading states

Avoid blank pages during loading whenever possible.

---

## Page Transitions

Transitions between pages should be smooth without creating unnecessary delays.

Navigation should always feel immediate.

---

## Progress Feedback

Progress indicators should be used for operations such as:

- Report generation
- File uploads
- Data synchronization
- Long-running administrative tasks

Progress components should accurately represent operation status whenever possible.

---

## Accessibility

Users with reduced motion preferences should experience minimal animation.

Motion should respect operating system accessibility settings whenever possible.

Animations must never interfere with keyboard navigation or screen reader functionality.

---

# 12. Future Enhancements

The current design system establishes a stable foundation while allowing future visual enhancements without requiring major redesigns.

Future additions should extend existing design principles rather than replace them.

---

## Theme Support

Future releases may introduce additional visual themes, including:

- Dark Mode
- High Contrast Mode
- Organization-specific branding

Theme support should rely on reusable design tokens instead of component-specific styling.

---

## Expanded Design Tokens

Future design tokens may include:

- Additional semantic colors
- Extended typography scales
- Additional spacing values
- Motion presets
- Shadow presets
- Border styles

New tokens should remain backward compatible with existing components.

---

## Advanced Components

Future interface improvements may include:

- Rich data visualizations
- Interactive dashboards
- Advanced filtering interfaces
- Custom analytics widgets
- Calendar-based planning views
- Configurable workspace layouts

These enhancements should continue following the principles defined in this document.

---

## Design Principles

Future enhancements should always preserve the following goals:

- Consistency
- Accessibility
- Responsiveness
- Simplicity
- Predictability
- Scalability

The design system should evolve gradually while maintaining a familiar user experience for both administrators and members.

---

# 13. Accessibility

Accessibility is a core requirement of the design system. Every interface should be usable by all supported users regardless of their abilities or assistive technologies.

Accessibility should be considered during design rather than added after implementation.

---

## Accessibility Principles

Every interface should be:

- Perceivable
- Operable
- Understandable
- Robust

Design decisions should align with WCAG 2.1 AA guidelines whenever applicable.

---

## Color & Contrast

Color should improve understanding but should never be the only way information is communicated.

Status indicators should always combine:

- Color
- Text
- Optional icon

Examples:

✓ Attendance Confirmed

⚠ Pending Review

✕ Validation Failed

Recommended minimum contrast ratio:

- Normal text: **4.5 : 1**
- Large text: **3 : 1**

---

## Keyboard Navigation

All interactive components should be fully usable using only a keyboard.

Users should be able to:

- Navigate through pages
- Open menus
- Submit forms
- Close dialogs
- Move between table rows
- Activate buttons

without requiring a pointing device.

---

## Focus Indicators

Every interactive component must display a clearly visible focus state.

Examples include:

- Buttons
- Links
- Form inputs
- Navigation items
- Table actions
- Dropdowns

Focus indicators should remain visually consistent throughout the application.

---

## Forms

Accessible forms should include:

- Visible labels
- Required field indicators
- Clear validation messages
- Helpful error descriptions
- Logical tab order

Placeholder text should never replace field labels.

---

## Icons

Icons should support content rather than replace it.

Whenever possible:

- Pair icons with text.
- Provide accessible labels for icon-only actions.
- Maintain consistent icon usage across modules.

Decorative icons should be ignored by assistive technologies.

---

## Tables

Administrative tables should remain accessible even when displaying large datasets.

Tables should provide:

- Clear column headers
- Consistent row structure
- Keyboard accessibility
- Responsive scrolling
- Readable spacing

Actions associated with each row should remain discoverable.

---

## Dialogs

Dialogs should:

- Move keyboard focus into the dialog when opened.
- Trap keyboard focus until closed.
- Restore focus to the triggering element when dismissed.
- Provide descriptive titles.
- Clearly identify primary and secondary actions.

Dialogs should never interrupt users unnecessarily.

---

## Notifications

Notifications should communicate important system events without disrupting user workflows.

Notifications should:

- Remain readable.
- Be dismissible when appropriate.
- Avoid covering critical interface elements.
- Use consistent placement throughout the application.

Critical notifications should require explicit acknowledgement when necessary.

---

## Responsive Accessibility

Accessibility requirements apply equally across:

- Desktop
- Tablet
- Mobile

Touch targets should remain comfortably usable on touch devices.

Text should never become unreadable because of responsive layout changes.

---

## Reduced Motion

Users who prefer reduced motion should experience minimal interface animation.

Animations should never:

- delay interaction;
- hide important information;
- interfere with accessibility technologies.

---

## Readability

Interfaces should prioritize readability through:

- Consistent typography
- Predictable layouts
- Appropriate spacing
- Clear information hierarchy
- Concise language

Content should be easy to scan without sacrificing important detail.

---

# 14. Design Principles

The design system provides a unified visual language for every frontend module of the InnoTech Hub Attendance System.

Every new screen, component, and workflow should follow these principles.

---

## Consistency

Components should behave consistently regardless of feature area.

Buttons, forms, cards, tables, dialogs, navigation, and notifications should maintain the same visual language throughout the application.

---

## Clarity

Interfaces should communicate information clearly and avoid unnecessary complexity.

Users should immediately understand:

- what they are viewing;
- what actions are available;
- what the current system state is.

---

## Simplicity

Interfaces should present only the information required for the current task.

Avoid:

- unnecessary decoration;
- duplicated information;
- excessive visual hierarchy;
- overcrowded layouts.

---

## Scalability

The design system should support future expansion without requiring redesign of existing components.

New modules should reuse existing:

- Colors
- Typography
- Spacing
- Cards
- Tables
- Dialogs
- Navigation
- Status indicators

before introducing new patterns.

---

## Accessibility

Accessibility is a shared responsibility across every screen and component.

New features should inherit the accessibility standards defined in this document instead of introducing alternative behaviors.

---

## Responsiveness

All interfaces should adapt gracefully across supported screen sizes while maintaining consistent behavior and visual hierarchy.

Layouts should optimize available space without sacrificing readability.

---

## Information Hierarchy

Users should always be able to identify:

1. Where they are.
2. What they are viewing.
3. What requires attention.
4. What actions are available.
5. What has changed.

Visual hierarchy should reinforce this order throughout the application.

---

## Maintainability

Design decisions should encourage reuse rather than duplication.

Whenever a new component is introduced, designers and developers should first determine whether an existing pattern can be extended before creating a new one.

This approach promotes consistency, reduces maintenance effort, and improves the overall user experience.

---

# Conclusion

This document defines the visual foundation for the frontend of the InnoTech Hub Attendance System.

It establishes consistent standards for colors, typography, spacing, layouts, components, responsiveness, motion, and accessibility while remaining independent of specific frontend frameworks or implementation details.

All future frontend modules should reference this design system to ensure a cohesive, accessible, and maintainable user experience across administrator and member interfaces.

As the application evolves, this design system should be extended through reusable design tokens and shared component patterns rather than introducing isolated visual styles or feature-specific conventions.
