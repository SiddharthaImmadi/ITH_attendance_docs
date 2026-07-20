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

**Purpose:** Member sees open sessions they can check in to.

**Layout:**
```
┌─────────────────────────────────────────┐
│ Welcome, R. Meera! | Logout             │
├─────────────────────────────────────────┤
│                                         │
│ Available Sessions                      │
│                                         │
│ ┌─────────────────────────────────────┐│
│ │ Workshop Attendance                 ││
│ │ 2026-08-01 | 9:00–12:00             ││
│ │ [ Check In ]                        ││
│ └─────────────────────────────────────┘│
│                                         │
│ My Check-in History                     │
│ ┌─────────────────────────────────────┐│
│ │ Volunteer Orientation  | Present    ││
│ │ 2026-08-05 09:02                    ││
│ └─────────────────────────────────────┘│
│                                         │
└─────────────────────────────────────────┘
```

**Components:**
- Welcome header + logout
- Available sessions list (open status only)
  - Title, date/time, "Check In" button per session
- My history section (read-only list)
  - Session title, final status, check-in time

---

## 7. Check-in Flow (Member)

**Step 1: Confirm Permissions**
```
┌──────────────────────────────────────┐
│ Ready to Check In?                   │
├──────────────────────────────────────┤
│                                      │
│ Workshop Attendance                  │
│ 2026-08-01 9:00–12:00 (Grace: 10m)  │
│                                      │
│ We need:                             │
│ • Your GPS location                  │
│ • Permission to use your camera      │
│                                      │
│ [ Cancel ]  [ Enable & Continue ]   │
│                                      │
└──────────────────────────────────────┘
```

**Step 2: Live Camera Capture**
```
┌──────────────────────────────────────┐
│ < Back to session                    │
├──────────────────────────────────────┤
│                                      │
│  [  Live Camera Stream  ]            │
│  [                     ]             │
│  [                     ]             │
│                                      │
│  Location: 12.9716, 77.5946 (12m)  │
│  Distance: 42m from venue ✓          │
│                                      │
│           [ Capture Photo ]          │
│                                      │
│  Camera permissions required         │
│                                      │
└──────────────────────────────────────┘
```

**Step 3: Confirm & Submit**
```
┌──────────────────────────────────────┐
│ Confirm Check-in                     │
├──────────────────────────────────────┤
│                                      │
│  [  Captured Photo ]                 │
│  [               ]                   │
│                                      │
│  Submitting...  [spinner]           │
│                                      │
│  [ Retake Photo ]  [ Submit ]        │
│                                      │
└──────────────────────────────────────┘
```

**Step 4: Result**
```
┌──────────────────────────────────────┐
│ ✓ Check-in Successful               │
├──────────────────────────────────────┤
│                                      │
│ Status: Present                      │
│ Time: 2026-08-01 08:55:00            │
│ Distance: 42m from venue             │
│                                      │
│ [ Back to Dashboard ]                │
│                                      │
└──────────────────────────────────────┘
```

Or error:
```
┌──────────────────────────────────────┐
│ ✗ Check-in Failed                   │
├──────────────────────────────────────┤
│                                      │
│ Error: Outside radius                │
│ You are 185m from the venue.          │
│ Allowed radius: 50m                  │
│                                      │
│ [ Try Again ]                        │
│                                      │
└──────────────────────────────────────┘
```

---

## 8. Premium Feel Guidelines

- Smooth loading spinners (not just text "Loading...")
- Color-coded status badges (green = present, yellow = late, red = rejected)
- Icons from lucide-react (checkmark, camera, map pin, etc.)
- Subtle shadows and spacing (Tailwind defaults)
- Clear error messages in red
- Success confirmations with a checkmark icon + green background
