# frontend_design_system.md — Design System & Styling

> Colors, typography, spacing, and component sizing for a consistent, premium look.

## 1. Color Palette

| Purpose | Color | Hex | Tailwind |
|---|---|---|---|
| Primary (CTA, links) | Blue | #3B82F6 | `blue-500` |
| Success (present status) | Green | #10B981 | `emerald-500` |
| Warning (late status) | Amber | #F59E0B | `amber-500` |
| Error (rejected, outside radius) | Red | #EF4444 | `red-500` |
| Info (pending verification) | Indigo | #6366F1 | `indigo-500` |
| Background | Light Gray | #F9FAFB | `gray-50` |
| Card Background | White | #FFFFFF | `white` |
| Border | Gray | #E5E7EB | `gray-200` |
| Text Primary | Dark Gray | #1F2937 | `gray-800` |
| Text Secondary | Medium Gray | #6B7280 | `gray-500` |
| Disabled | Light Gray | #D1D5DB | `gray-300` |

**Usage:**
- Buttons: `bg-blue-500 hover:bg-blue-600 text-white`
- Status badges: `bg-green-100 text-green-800` (present), `bg-red-100 text-red-800` (rejected)
- Inputs: `border border-gray-300 focus:border-blue-500 focus:ring-2 focus:ring-blue-200`

## 2. Typography

| Element | Font | Size | Weight | Line Height |
|---|---|---|---|---|
| Page Title | Inter | 32px | 700 (bold) | 1.2 |
| Section Header | Inter | 20px | 600 (semibold) | 1.3 |
| Card Title | Inter | 16px | 600 (semibold) | 1.4 |
| Body | Inter | 14px | 400 (normal) | 1.6 |
| Small/Label | Inter | 12px | 500 (medium) | 1.4 |
| Button Text | Inter | 14px | 500 (medium) | 1.4 |

**Font:** Use Tailwind's default system stack (or load Inter from Google Fonts).

```css
/* src/index.css */
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap');

body {
    font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
}
```

## 3. Spacing Scale

Use Tailwind's default spacing (4px base unit):

| Size | Pixels | Tailwind | Usage |
|---|---|---|---|
| xs | 4px | `p-1` | Tight spacing in tables, inline icons |
| sm | 8px | `p-2` | Padding in small buttons, gaps in lists |
| md | 16px | `p-4` | Standard padding in cards, inputs |
| lg | 24px | `p-6` | Padding in large cards, section spacing |
| xl | 32px | `p-8` | Gap between major sections |
| 2xl | 48px | `p-12` | Top-level page margins |

**Page margins:** `px-6 py-8` (24px horizontal, 32px vertical).
**Section gaps:** `gap-6` (24px between sections).

## 4. Component Sizing

| Component | Width | Height | Notes |
|---|---|---|---|
| Input field | 100% (in form) | 40px | `h-10` |
| Button | Auto (padding-based) | 40px | `h-10` |
| Card | Full width on mobile, max 600px desktop | Auto | Responsive |
| Form | 100% (max 500px on desktop) | Auto | Responsive |
| Modal/Dialog | 90vw (max 600px) | Auto | Centered |

## 5. Button Styles

### Primary Button (CTA)
```jsx
<button className="bg-blue-500 hover:bg-blue-600 text-white font-medium py-2 px-4 rounded-lg transition">
  Login
</button>
```

### Secondary Button
```jsx
<button className="bg-gray-100 hover:bg-gray-200 text-gray-800 font-medium py-2 px-4 rounded-lg transition">
  Cancel
</button>
```

### Danger Button
```jsx
<button className="bg-red-500 hover:bg-red-600 text-white font-medium py-2 px-4 rounded-lg transition">
  Delete
</button>
```

### Disabled Button
```jsx
<button disabled className="bg-gray-300 text-gray-500 font-medium py-2 px-4 rounded-lg cursor-not-allowed">
  Unavailable
</button>
```

## 6. Form Input Styles

```jsx
<input
  type="email"
  className="w-full h-10 px-3 border border-gray-300 rounded-lg focus:outline-none focus:border-blue-500 focus:ring-2 focus:ring-blue-200 transition"
  placeholder="Enter email"
/>
```

**States:**
- Default: `border-gray-300`
- Focus: `border-blue-500 ring-2 ring-blue-200`
- Error: `border-red-500 ring-2 ring-red-200`
- Disabled: `bg-gray-100 cursor-not-allowed`

## 7. Card Styling

```jsx
<div className="bg-white border border-gray-200 rounded-lg p-4 shadow-sm hover:shadow-md transition">
  <h3 className="text-gray-800 font-semibold mb-2">Card Title</h3>
  <p className="text-gray-600 text-sm">Card content goes here</p>
</div>
```

**Shadows:**
- Subtle: `shadow-sm`
- Hover: `hover:shadow-md`
- Modal/dropdown: `shadow-lg`

## 8. Status Badge Styling

```jsx
// Present (success)
<span className="inline-block bg-emerald-100 text-emerald-800 text-sm font-medium px-3 py-1 rounded-full">
  Present
</span>

// Late (warning)
<span className="inline-block bg-amber-100 text-amber-800 text-sm font-medium px-3 py-1 rounded-full">
  Late
</span>

// Rejected (error)
<span className="inline-block bg-red-100 text-red-800 text-sm font-medium px-3 py-1 rounded-full">
  Rejected
</span>

// Pending Verification (info)
<span className="inline-block bg-indigo-100 text-indigo-800 text-sm font-medium px-3 py-1 rounded-full">
  Pending
</span>
```

## 9. Responsive Breakpoints

Use Tailwind breakpoints (mobile-first):

| Breakpoint | Width | Tailwind | Usage |
|---|---|---|---|
| Mobile | <640px | (default) | Full-width layouts |
| Tablet | ≥640px | `sm:` | 2-column layouts |
| Desktop | ≥1024px | `lg:` | 3-column, wider content |

**Example:**
```jsx
<div className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-4">
  {/* Cards stack vertically on mobile, 2 cols on tablet, 3 on desktop */}
</div>
```

## 10. Elevation & Depth

```
Level 1 (default):    no shadow
Level 2 (cards):      shadow-sm (subtle)
Level 3 (hover):      shadow-md (lifted)
Level 4 (modals):     shadow-lg (prominent)
```

## 11. Transitions & Animations

```jsx
// Fade in
className="opacity-0 animate-fade-in"

// Slide in from left
className="translate-x-full transition-transform duration-300"

// Button hover
className="bg-blue-500 hover:bg-blue-600 transition-colors duration-200"

// Loading spinner
className="animate-spin h-5 w-5"
```

Use Framer Motion for complex animations (see `component_guidelines.md`).

## 12. Dark Mode (Future)

Not included in Phase 1. Add `prefers-color-scheme` support in Phase 2.

## 13. Accessibility

- Minimum contrast ratio 4.5:1 for text (WCAG AA)
- Focus indicators on all interactive elements
- Icon buttons have aria-label
- Color not the only indicator of status (use text + icon + color)


## 14. Phase 2 Design System Extensions

Phase 2 expands the design system to support the enhanced desktop-first dashboard, live attendance monitoring, and administrator workflows.

---

## 14.1 Layout Principles

The application follows a **desktop-first** layout.

Desktop Layout:

```
+--------------------------------------------------------------+
| Sidebar |                Main Content                        |
|         |----------------------------------------------------|
|         | Page Header                                        |
|         |----------------------------------------------------|
|         | Cards / Tables / Timeline / Forms                  |
|         |                                                    |
+--------------------------------------------------------------+
```

Responsive behavior:

- Desktop: Fixed sidebar
- Tablet: Collapsible sidebar
- Mobile: Hamburger navigation

---

## 14.2 Sidebar

Sidebar Width:

- Expanded: 260px
- Collapsed: 72px

Sidebar Items:

- Dashboard
- Sessions
- History
- Profile
- Logout

Active item style:

- Primary background
- White icon
- White text
- Rounded corners

Inactive items:

- Transparent background
- Gray text
- Blue hover background

---

## 14.3 Dashboard Cards

Dashboard cards should use consistent spacing and hierarchy.

Card Structure:

```
Title

Status Badge

Primary Information

Secondary Information

Primary Action
```

Card spacing:

- Padding: p-6
- Gap: gap-6
- Border Radius: rounded-xl
- Shadow: shadow-sm
- Hover: shadow-md

---

## 14.4 Active Session Card

The Active Session card is the highest-priority component.

Display:

- Session title
- Current attendance status
- Venue
- Time remaining
- Session progress
- Open Session button

The card should always appear above Available Sessions.

---

## 14.5 Session Cards

Available Session cards display:

- Session title
- Date
- Time
- Venue
- Attendance status
- Check In button

Cards should maintain equal height within the grid.

---

## 14.6 Timeline Component

Timeline events are displayed vertically.

```
● Checked In
│
● Entered Venue
│
● Left Venue
│
● Returned
│
● Checked Out
```

Each event contains:

- Status icon
- Event title
- Timestamp
- Optional backend message

---

## 14.7 Status Colors

Additional Phase 2 status colors:

| Status | Tailwind |
|---------|----------|
| Active Session | blue-500 |
| Inside Venue | emerald-500 |
| Outside Venue | red-500 |
| Pending Approval | amber-500 |
| Approved | emerald-500 |
| Rejected | red-500 |
| Completed | gray-600 |

Status indicators should always include both color and text.

---

## 14.8 Progress Indicators

Session progress should use a horizontal progress bar.

Examples:

```
████████░░ 80%

███░░░░░░░ 30%
```

Use:

- Rounded edges
- Smooth animation
- Percentage label

---

## 14.9 Tables

Administrator tables should include:

- Sticky header
- Hover highlight
- Alternating row backgrounds (optional)
- Responsive horizontal scrolling

Table actions should appear in the final column.

---

## 14.10 Dialogs

Dialogs are used for:

- Early Check-out Request
- Confirmation Actions
- Delete Confirmation
- Session Close Confirmation

Dialog Size:

- Max Width: 600px
- Rounded corners
- Shadow-lg

Buttons:

Primary action on the right.

Cancel action on the left.

---

## 14.11 Empty States

Every major page should define an empty state.

Example:

"No active session."

"Join an available session to begin attendance."

Provide a primary action whenever possible.

---

## 14.12 Loading States

Use skeleton loaders instead of blank pages.

Examples:

- Dashboard cards
- Session Detail
- Timeline
- Attendance Summary
- Tables

Loading indicators should preserve page layout.

---

## 14.13 Notification Design

Use toast notifications for:

- Check-in successful
- Check-out successful
- Approval received
- Request rejected
- Session created
- Session closed
- API failures

Toast Position:

Top-right on desktop.

Top-center on mobile.

---

## 14.14 Icons

Use Lucide React icons consistently.

Recommended icons:

| Action | Icon |
|---------|------|
| Dashboard | LayoutDashboard |
| Session | Calendar |
| Attendance | CheckCircle |
| Timeline | Clock |
| Active Session | Activity |
| Check In | LogIn |
| Check Out | LogOut |
| History | History |
| Profile | User |
| Logout | DoorOpen |

Icons should be paired with text whenever possible.

---

## 14.15 Animation Guidelines

Animations should enhance usability without slowing interaction.

Recommended:

- Fade between pages
- Card hover elevation
- Sidebar collapse animation
- Timeline item appearance
- Toast slide animation
- Progress bar animation

Avoid excessive motion that distracts users.

---

## 14.16 Design Principles

All Phase 2 interfaces should follow these principles:

- Desktop-first layout
- Clean Modern SaaS appearance
- Consistent spacing
- Minimal visual clutter
- Strong information hierarchy
- High accessibility
- Responsive behavior across all supported devices

