# Frontend Development To-Do List — Phase 1

## Milestone 0: Environment & Scaffolding

- [ ] Create Vite project with React + TypeScript template
- [ ] Install and configure Tailwind CSS
- [ ] Install and configure shadcn/ui
- [ ] Set up React Router v6
- [ ] Install dependencies: react-query, react-hook-form, zod, axios, framer-motion, lucide-react
- [ ] Create project folder structure (src/pages, src/components, src/lib, etc.)
- [ ] Set up environment variables file (.env.local with VITE_API_BASE_URL)
- [ ] Create API client (src/lib/api.ts) with mock mode toggle
- [ ] Verify frontend runs locally on localhost:5173
- [ ] Verify backend health endpoint can be reached (test CORS)

## Milestone 1: Authentication (Login & Session Restore)

### Login Page
- [ ] Create LoginPage component (src/pages/LoginPage.tsx)
- [ ] Build login form with email and password fields
- [ ] Add form validation with Zod schema
- [ ] Create error message display
- [ ] Add loading state on submit button
- [ ] Implement form submission with mock API (shows loading, then success/error)
- [ ] Style with Tailwind and shadcn/ui button, input components
- [ ] Ensure responsive layout (mobile-first)

### Authentication Logic
- [ ] Create AuthContext (src/lib/auth.ts) with login/logout functions
- [ ] Wrap app with AuthProvider in main.tsx
- [ ] Implement JWT storage in localStorage
- [ ] Create useAuth() custom hook for accessing auth state
- [ ] Add session persistence (restore from localStorage on app load)
- [ ] Create ProtectedRoute component to guard admin/member routes

### Get Current User Endpoint
- [ ] Create useGetMe hook that calls GET /me
- [ ] Implement session restore flow (on app mount, fetch /me to refresh user)
- [ ] Store user profile in AuthContext
- [ ] Handle 401 errors (token expired, redirect to login)

### Role-Based Routing
- [ ] Create AdminDashboard placeholder page
- [ ] Create MemberDashboard placeholder page
- [ ] Implement routing logic: if admin, go to /admin/dashboard; if member, go to /member/dashboard
- [ ] Test login flow end-to-end with mocks
- [ ] Create logout button in header
- [ ] Test logout clears token and redirects to login

## Milestone 2: Session Management (Admin Side)

### Admin Dashboard
- [ ] Create AdminDashboard page (src/pages/AdminDashboard.tsx)
- [ ] Add welcome header with user name
- [ ] Add logout button
- [ ] Create "Create Session" CTA button
- [ ] Build sessions list component
- [ ] Implement useSessionsList hook (mock data initially)
- [ ] Display sessions as cards (title, status, date/time)
- [ ] Add "View" button on each session card

### Create Session Form
- [ ] Create CreateSessionPage (src/pages/CreateSessionPage.tsx)
- [ ] Build session creation form with all required fields
- [ ] Add form validation (title, date, start/end time, venue, radius)
- [ ] Implement "Use My Location" button (use navigator.geolocation)
- [ ] Add date/time pickers
- [ ] Implement form submission with useCreateSession hook
- [ ] Add error display for validation failures
- [ ] Add loading state on submit button
- [ ] Redirect to dashboard on success
- [ ] Style form for premium feel

### Session Detail & Check-ins
- [ ] Create SessionDetail page (src/pages/SessionDetail.tsx)
- [ ] Display session information (title, date, time, venue, radius)
- [ ] Show check-in summary (Present, Late, Pending, Rejected counts)
- [ ] Build check-ins table with member names, check-in times, distances, statuses
- [ ] Implement status badges (green for present, yellow for late, red for rejected)
- [ ] Add "Close Session" button
- [ ] Add "Export Excel" button (disabled for now, placeholder)
- [ ] Implement useSession hook to fetch session detail
- [ ] Add back navigation

### Session List Filtering (Optional for Phase 1)
- [ ] Add status filter (All, Scheduled, Open, Closed)
- [ ] Add search by session title
- [ ] Add sorting (by date, by status)

## Milestone 3: Member Check-in Flow

### Member Dashboard
- [ ] Create MemberDashboard page
- [ ] Display "Available Sessions" (open sessions only)
- [ ] Show session cards with title, date/time, "Check In" button
- [ ] Display "My Check-in History" section below
- [ ] Implement useSessionsList hook (filtered for members)
- [ ] Add logout button

### Check-in Flow (Multi-Step)

#### Step 1: Permission Request Screen
- [ ] Create CheckinFlow page (src/pages/CheckinFlow.tsx)
- [ ] Display session information
- [ ] Show "Ready to check in?" message
- [ ] List what's needed (GPS location, camera access)
- [ ] Add "Enable & Continue" button
- [ ] Add "Cancel" button

#### Step 2: Live Camera Capture
- [ ] Create CameraCapture component (src/components/features/attendance/CameraCapture.tsx)
- [ ] Request camera permission (navigator.mediaDevices.getUserMedia)
- [ ] Display live video stream from camera
- [ ] Request GPS location (navigator.geolocation.getCurrentPosition)
- [ ] Display current location and distance from venue
- [ ] Add "Capture Photo" button
- [ ] Show GPS accuracy and distance dynamically

#### Step 3: Photo Confirmation
- [ ] Show captured photo preview
- [ ] Display "Retake Photo" and "Submit" buttons
- [ ] Show GPS location and distance to venue
- [ ] Add loading state on submit

#### Step 4: Success/Error Response
- [ ] Display success screen with status (Present/Late/Pending)
- [ ] Show check-in time and distance
- [ ] Add "Back to Dashboard" button for success
- [ ] Display error screen if check-in fails (outside radius, duplicate, etc.)
- [ ] Show clear error message with reason
- [ ] Add "Try Again" button for errors

### Check-in API Integration
- [ ] Create useCheckIn hook
- [ ] Implement multipart form data submission (lat, lng, accuracy, photo)
- [ ] Handle success responses (final_status, distance, check_in_time)
- [ ] Handle error responses (OUTSIDE_RADIUS, DUPLICATE_CHECK_IN, SESSION_NOT_OPEN, etc.)
- [ ] Test with real backend once available

### Attendance History
- [ ] Create AttendanceHistory page (src/pages/AttendanceHistory.tsx)
- [ ] Implement useAttendanceHistory hook
- [ ] Display list of past check-ins (session name, check-in time, status)
- [ ] Show status badges with appropriate colors
- [ ] Sort by most recent first
- [ ] Add back navigation

## Milestone 4: Reporting (Admin Only)

### Excel Export
- [ ] Add "Export Excel" button to SessionDetail page
- [ ] Implement useExportReport hook
- [ ] Trigger download of .xlsx file
- [ ] Handle export errors
- [ ] Show loading state during export
- [ ] Test exported file opens correctly in Excel/Sheets

## Milestone 5: Polish & Premium Feel

### UI Polish
- [ ] Add loading spinners on all async operations
- [ ] Implement smooth page transitions (Framer Motion)
- [ ] Add success/error toast notifications
- [ ] Ensure all status badges have correct colors (green/yellow/red/indigo)
- [ ] Add icons to buttons (from lucide-react)
- [ ] Review spacing and alignment (use design_system.md spacing scale)
- [ ] Add hover effects on clickable elements
- [ ] Test responsive layout on mobile, tablet, desktop

### Error Handling
- [ ] Display user-friendly error messages for all API failures
- [ ] Handle network errors (offline)
- [ ] Handle 401 unauthorized (redirect to login)
- [ ] Handle 403 forbidden (show permission error)
- [ ] Handle 404 not found (show resource error)
- [ ] Handle 422 validation errors (show specific field errors)

### Form Improvements
- [ ] Clear forms after successful submission
- [ ] Disable submit buttons while loading
- [ ] Show validation errors on blur (not just submit)
- [ ] Add success message after form submission

### Browser Testing
- [ ] Test on Chrome (latest)
- [ ] Test on Firefox (latest)
- [ ] Test on Safari (latest)
- [ ] Test on mobile browsers (iOS Safari, Android Chrome)
- [ ] Verify geolocation works on all browsers
- [ ] Verify camera access works on all browsers

### Performance
- [ ] Lazy load pages (code splitting)
- [ ] Optimize images (if added)
- [ ] Check bundle size (should be <200KB gzipped)
- [ ] Test Core Web Vitals

### Accessibility
- [ ] Ensure all form inputs have labels
- [ ] Verify focus indicators visible on all elements
- [ ] Test with keyboard navigation (Tab through all pages)
- [ ] Add aria-labels to icon-only buttons
- [ ] Test with screen reader (if possible)

## Integration Testing (Before Merge)

- [ ] Test full login flow (with real backend)
- [ ] Test session creation flow (create, see in list)
- [ ] Test session detail view (shows check-ins)
- [ ] Test check-in flow end-to-end (permission → camera → submit)
- [ ] Test duplicate check-in error
- [ ] Test outside radius rejection
- [ ] Test attendance history display
- [ ] Test Excel export
- [ ] Test logout
- [ ] Test mobile responsiveness

## Documentation

- [ ] Verify all components follow component_guidelines.md
- [ ] Verify all styling follows design_system.md
- [ ] Verify all routing follows routing.md
- [ ] Verify all state management follows state_management.md
- [ ] Update progress.md with completion status
- [ ] Add entry to changelog.md for Phase 1 completion
