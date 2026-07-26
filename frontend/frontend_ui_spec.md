# frontend_ui_spec.md — UI Mockups & Screen Specifications

> High-level screen layouts for Phase 1. Detailed component breakdown in `component_guidelines.md`.

## 1. Screen Hierarchy

```
Login Screen
├── Admin Dashboard
│   ├── Sessions List
│   │   └── Session Detail + Check-ins
│   └── Create Session Form
├── Member Dashboard
│   ├── Available Sessions List
│   └── Check-in Flow (Geolocation + Camera)
└── Common
    ├── User Profile / Settings (if added)
    └── Logout
```

## 2. Login Screen

**Purpose:** Authenticate user (admin or member).

**Layout:**
```
┌─────────────────────────────────┐
│   InnoTech-Hub Attendance       │
│   & Activity Tracking           │
├─────────────────────────────────┤
│                                 │
│   Email / Username              │
│   [________________]            │
│                                 │
│   Password                      │
│   [________________]            │
│                                 │
│   [ Login ]                     │
│                                 │
│   Error message (if failed)     │
│   "Incorrect email or password" │
│                                 │
└─────────────────────────────────┘
```

**Form fields:**
- Email input (type="email")
- Password input (type="password")
- Login button (submit)
- Loading state (spinner on button)
- Error message (red text below form)

**Role determination:** Backend returns role in response; frontend routes accordingly.

---

## 3. Admin Dashboard

**Purpose:** Admin sees sessions they created, check-ins for each.

**Layout:**
```
┌─────────────────────────────────────────────────────────┐
│ Welcome, Admin! | Logout                               │
├─────────────────────────────────────────────────────────┤
│                                                         │
│ [ + Create Session ]                                   │
│                                                         │
│ ┌─────────────────────────────────────────────────────┐
│ │ Session 1: Workshop Attendance          [ View ]    │
│ │ Status: open | 2026-08-01 | 9:00–12:00              │
│ └─────────────────────────────────────────────────────┘
│                                                         │
│ ┌─────────────────────────────────────────────────────┐
│ │ Session 2: Volunteer Orientation        [ View ]    │
│ │ Status: scheduled | 2026-08-05 | 9:00–11:00         │
│ └─────────────────────────────────────────────────────┘
│                                                         │
└─────────────────────────────────────────────────────────┘
```

**Components:**
- Welcome header + logout button
- "Create Session" CTA button
- Session list (card per session)
  - Title, status badge, date/time
  - View button (navigate to detail)

---

## 4. Session Detail (Admin View)

**Purpose:** Admin sees session config + list of all check-ins.

**Layout:**
```
┌───────────────────────────────────────────────────────┐
│ < Back                                                │
├───────────────────────────────────────────────────────┤
│                                                       │
│ Workshop Attendance                                   │
│ Status: open | 2026-08-01 | 9:00–12:00               │
│ Venue: Latitude 12.9716, Longitude 77.5946, 50m rad  │
│                                                       │
│ [ Close Session ]   [ Export Excel ]                 │
│                                                       │
├─ Check-ins ─────────────────────────────────────────┤
│                                                       │
│ 42 Present | 6 Late | 3 Pending | 5 Rejected        │
│                                                       │
│ ┌─────────────────────────────────────────────────┐ │
│ │ R. Meera         | 08:55 | 42m away | Present  │ │
│ │ B. Sankar        | 09:05 | 38m away | Late     │ │
│ │ V. Kumar         | 10:30 | 185m away | Rejected│ │
│ └─────────────────────────────────────────────────┘ │
│                                                       │
└───────────────────────────────────────────────────────┘
```

**Components:**
- Back navigation
- Session header (title, status, date, venue)
- Action buttons (Close Session, Export Excel)
- Stats summary (counts per status)
- Check-in table (searchable, sortable)

---

## 5. Create Session Form

**Purpose:** Admin creates a new session.

**Layout:**
```
┌─────────────────────────────────────────┐
│ Create New Session                      │
├─────────────────────────────────────────┤
│                                         │
│ Title *                                 │
│ [____________________________]          │
│                                         │
│ Purpose (optional)                      │
│ [____________________________]          │
│                                         │
│ Date *                                  │
│ [____________________]                  │
│                                         │
│ Start Time *   End Time *               │
│ [_______]      [_______]                │
│                                         │
│ Grace Period (minutes) *                │
│ [_______]                               │
│                                         │
│ Venue Latitude * Longitude *            │
│ [_______]        [_______]              │
│ [ Use My Location ]                     │
│                                         │
│ Radius (meters) *                       │
│ [_______]                               │
│                                         │
│ [ Cancel ]  [ Create Session ]          │
│                                         │
│ Errors: "end_time must be after..."    │
│                                         │
└─────────────────────────────────────────┘
```

**Form fields:**
- Title (text input, required)
- Purpose (textarea, optional)
- Date (date picker, required)
- Start/End time (time pickers, required)
- Grace period (number input, required)
- Venue lat/lng (number inputs, required)
  - "Use My Location" button (geolocation)
- Radius (number input, required)
- Error display (red text)
- Loading state on submit button

---

## 6. Member Dashboard

**Purpose:**  
The Member Dashboard is the primary landing page after a member logs in. It provides a clear overview of the member's current attendance, available sessions, and previous attendance history while allowing quick access to ongoing activities.

---

### Layout (Desktop)

```
┌──────────────────────────────────────────────────────────────────────────────────────────────┐
│ InnoTech-Hub                                                          R. Meera              │
├──────────────┬───────────────────────────────────────────────────────────────────────────────┤
│ Dashboard    │ Welcome Back, R. Meera                                                     │
│ Sessions     │                                                                            │
│ History      │ ┌────────────────────────────────────────────────────────────────────────┐ │
│ Profile      │ │ Active Session                                                         │ │
│ Logout       │ │ Workshop Attendance                                                    │ │
│              │ │ Status : Present                                                       │ │
│              │ │ Time   : 09:00 – 12:00                                                │ │
│              │ │ Progress : ██████████░░░░░░░                                           │ │
│              │ │                                                                        │ │
│              │ │                     [ Open Session ]                                   │ │
│              │ └────────────────────────────────────────────────────────────────────────┘ │
│              │                                                                            │
│              │ Available Sessions                                                        │
│              │                                                                            │
│              │ ┌─────────────────────┐ ┌─────────────────────┐ ┌─────────────────────┐ │
│              │ │ Workshop A          │ │ Volunteer Event     │ │ AI Seminar          │ │
│              │ │ Today               │ │ Tomorrow            │ │ Next Week           │ │
│              │ │ 📍 Main Hall        │ │ 📍 Lab 2            │ │ 📍 Hall A           │ │
│              │ │ [ Check In ]        │ │ [ View ]            │ │ [ View ]            │ │
│              │ └─────────────────────┘ └─────────────────────┘ └─────────────────────┘ │
│              │                                                                            │
│              │ Recent Attendance                                                         │
│              │                                                                            │
│              │ Workshop Attendance        Present        Yesterday                        │
│              │ Volunteer Orientation     Late           10 Jul                           │
│              │ AI Bootcamp               Present        02 Jul                           │
└──────────────┴──────────────────────────────────────────────────────────────────────────────┘
```

---

### Sidebar Navigation

The member interface uses a sidebar for primary navigation.

Navigation Items:

- Dashboard
- Sessions
- History
- Profile
- Logout

Responsive behavior:

- Desktop → Fixed sidebar
- Tablet → Collapsible sidebar
- Mobile → Sidebar opens from a hamburger menu

Navigation remains consistent throughout all member pages.

---

## Dashboard Structure

The dashboard is divided into three major sections.

### 1. Active Session

Displayed only when the member currently has an active attendance session.

Displays:

- Session title
- Session time
- Current attendance status
- Session progress
- Open Session button

The Active Session section always appears at the top because it is the member's highest priority task.

If no active session exists, this section is omitted.

---

### 2. Available Sessions

Displays sessions available for participation.

Each card contains:

- Session title
- Date
- Time
- Venue
- Availability
- Primary action

Possible actions include:

- Check In
- View Details

Cards should maintain a consistent visual appearance regardless of session status.

---

### 3. Recent Attendance

Displays recently completed attendance records.

Each record shows:

- Session title
- Final attendance status
- Attendance date

Selecting a record opens the Attendance Summary screen.

---

## Card Design

All cards throughout the dashboard follow a consistent design language.

Each card includes:

- Title
- Supporting information
- Status badge (where applicable)
- Primary action button

Visual characteristics:

- Rounded corners
- Soft elevation
- Consistent spacing
- Hover feedback
- Keyboard accessibility

---

## Empty States

### No Active Session

Display an informational card indicating that no attendance session is currently active.

---

### No Available Sessions

Display:

> No sessions are currently available.

with an appropriate illustration or icon.

---

### No Attendance History

Display:

> Your attendance history will appear here after completing your first session.

---

## Loading States

While dashboard information is loading:

- Replace cards with skeleton placeholders.
- Disable actions until required data is available.
- Prevent layout shifts during loading.

---

## Error States

If dashboard data cannot be retrieved:

Display:

- Error message
- Retry button

Previously loaded information should remain visible whenever possible.

---

## Accessibility

The dashboard must support:

- Keyboard navigation
- Screen reader compatibility
- Visible keyboard focus indicators
- Responsive layouts
- High-contrast status indicators

---

## Design Principles

The Member Dashboard should emphasize:

- Immediate awareness of active attendance
- Minimal effort to continue an active session
- Clear information hierarchy
- Modern SaaS-inspired design
- Consistent card-based layouts
- Responsive behavior across desktop, tablet, and mobile devices

## 7. Attendance Lifecycle (Member)

**Purpose:**  
Guide the member through the complete attendance lifecycle, from check-in to final attendance completion, while providing continuous visibility into attendance status throughout the session.

---

### Step 1 – Select Session

From the Member Dashboard, the member selects an available session.

The session card displays:

- Session title
- Date
- Time
- Venue
- Primary action: **Check In**

---

### Step 2 – Permission Notice

Before requesting device permissions, explain why they are required.

```
┌──────────────────────────────────────────────┐
│ Ready to Check In?                           │
├──────────────────────────────────────────────┤
│                                              │
│ We require:                                  │
│                                              │
│ • Your live GPS location                     │
│ • Access to your device camera               │
│                                              │
│ This information is used only to verify      │
│ your attendance for this session.            │
│                                              │
│ [ Cancel ]     [ Continue ]                  │
└──────────────────────────────────────────────┘
```

After confirmation:

- Request GPS permission.
- Request Camera permission.

---

### Step 3 – Live Camera Capture

Display the live camera preview.

Show:

- Camera preview
- Current GPS location
- Distance from venue
- Camera status

Primary action:

- Capture Photo

---

### Step 4 – Confirm Check-in

Display the captured image for confirmation.

Options:

- Retake Photo
- Submit Check-in

During submission:

- Disable all actions.
- Display loading indicator.

---

### Step 5 – Check-in Result

If successful:

Display:

- Check-in successful
- Current attendance status
- Check-in time

Primary action:

- Open Active Session

If unsuccessful:

Display the backend error message with an option to retry when appropriate.

---

### Step 6 – Active Session

The Active Session page remains available until attendance is completed.

Display:

- Session information
- Current attendance status
- Presence status (Inside / Outside Venue)
- Live attendance timeline
- Session progress
- Check Out button

Attendance information updates automatically as new events are received.

Example timeline:

```
✓ Checked In        09:02 AM
↗ Left Venue        09:47 AM
↘ Returned          09:55 AM
```

---

### Step 7 – Request Early Check-out

When the member selects **Check Out** before the session has officially ended, an approval request is required.

Display:

```
┌──────────────────────────────────────────────┐
│ Request Early Check-out                      │
├──────────────────────────────────────────────┤
│                                              │
│ Reason                                       │
│                                              │
│ [______________________________]             │
│                                              │
│ [ Cancel ]   [ Submit Request ]              │
└──────────────────────────────────────────────┘
```

The reason is sent to the administrator for review.

---

### Step 8 – Pending Administrator Approval

After submission, display:

```
Request Submitted

Your early check-out request is awaiting
administrator approval.

You may continue participating in the session
while waiting for a decision.
```

The Active Session page remains accessible.

---

### Step 9 – Administrator Decision

#### If Approved

Notify the member.

Enable the **Complete Check-out** action.

After confirmation, complete attendance.

---

#### If Rejected

Notify the member that the request was rejected.

Display the administrator's response when provided.

The member continues participating in the active session.

---

### Step 10 – Attendance Summary

After successful check-out, display a dedicated attendance summary.

Include:

- Session title
- Check-in time
- Check-out time
- Total attendance duration
- Final attendance status
- Attendance timeline

Primary action:

- Return to Dashboard

---

### Design Principles

The attendance flow should:

- Minimize unnecessary user interactions.
- Clearly communicate each attendance stage.
- Provide continuous visibility into attendance status.
- Present meaningful feedback during all operations.
- Treat the backend as the authoritative source for attendance decisions.---

## 8. Premium Feel Guidelines

- Smooth loading spinners (not just text "Loading...")
- Color-coded status badges (green = present, yellow = late, red = rejected)
- Icons from lucide-react (checkmark, camera, map pin, etc.)
- Subtle shadows and spacing (Tailwind defaults)
- Clear error messages in red
- Success confirmations with a checkmark icon + green background
