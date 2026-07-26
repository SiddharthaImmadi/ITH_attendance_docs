# frontend_routing.md — Route Structure & Navigation

> Page routes, route protection, and navigation patterns using React Router v6.

## 1. Route Hierarchy

```
/
├── /login                    (LoginPage) — public
├── /admin
│   ├── /dashboard           (AdminDashboard) — admin only
│   ├── /sessions            (SessionsList) — admin only
│   ├── /sessions/:id        (SessionDetail) — admin only
│   └── /sessions/create     (CreateSessionPage) — admin only
└── /member
    ├── /dashboard           (MemberDashboard) — member only
    ├── /sessions/:id/checkin (CheckinFlow) — member only
    └── /history             (AttendanceHistory) — member only
```

## 2. Router Setup

```typescript
// src/main.tsx
import React from 'react';
import ReactDOM from 'react-dom/client';
import { BrowserRouter as Router, Routes, Route } from 'react-router-dom';
import App from './App';
import './index.css';

ReactDOM.createRoot(document.getElementById('root')!).render(
  <React.StrictMode>
    <Router>
      <App />
    </Router>
  </React.StrictMode>,
);

// src/App.tsx
import { Routes, Route, Navigate } from 'react-router-dom';
import { useAuth } from '@/lib/hooks';
import LoginPage from '@/pages/LoginPage';
import AdminDashboard from '@/pages/AdminDashboard';
import MemberDashboard from '@/pages/MemberDashboard';
import NotFound from '@/pages/NotFound';
import ProtectedRoute from '@/components/layout/ProtectedRoute';

function App() {
  return (
    <Routes>
      <Route path="/login" element={<LoginPage />} />
      <Route
        path="/admin/*"
        element={
          <ProtectedRoute requiredRole="admin">
            <AdminRoutes />
          </ProtectedRoute>
        }
      />
      <Route
        path="/member/*"
        element={
          <ProtectedRoute requiredRole="member">
            <MemberRoutes />
          </ProtectedRoute>
        }
      />
      <Route path="/" element={<Navigate to="/login" replace />} />
      <Route path="*" element={<NotFound />} />
    </Routes>
  );
}

function AdminRoutes() {
  return (
    <Routes>
      <Route path="/dashboard" element={<AdminDashboard />} />
      <Route path="/sessions/:id" element={<SessionDetail />} />
      <Route path="/sessions/create" element={<CreateSessionPage />} />
    </Routes>
  );
}

function MemberRoutes() {
  return (
    <Routes>
      <Route path="/dashboard" element={<MemberDashboard />} />
      <Route path="/sessions/:id/checkin" element={<CheckinFlow />} />
      <Route path="/history" element={<AttendanceHistory />} />
    </Routes>
  );
}

export default App;
```

## 3. Protected Route Component

```typescript
// src/components/layout/ProtectedRoute.tsx
import { Navigate } from 'react-router-dom';
import { useAuth } from '@/lib/hooks';

interface ProtectedRouteProps {
  requiredRole?: 'admin' | 'member';
  children: React.ReactNode;
}

export default function ProtectedRoute({ requiredRole, children }: ProtectedRouteProps) {
  const { user, isAuthenticated } = useAuth();

  if (!isAuthenticated) {
    return <Navigate to="/login" replace />;
  }

  if (requiredRole && user?.role !== requiredRole) {
    return <Navigate to="/login" replace />;
  }

  return <>{children}</>;
}
```

## 4. Navigation Patterns

### Redirect on Login
```typescript
// src/pages/LoginPage.tsx
import { useNavigate } from 'react-router-dom';
import { useAuth } from '@/lib/hooks';

export default function LoginPage() {
  const navigate = useNavigate();
  const { login } = useAuth();

  const handleLogin = async (email: string, password: string) => {
    await login(email, password);
    const user = useAuth().user;
    if (user?.role === 'admin') {
      navigate('/admin/dashboard');
    } else {
      navigate('/member/dashboard');
    }
  };

  return (
    // Login form...
  );
}
```

### Programmatic Navigation
```typescript
// From any component
import { useNavigate } from 'react-router-dom';

export function SessionCard({ session }) {
  const navigate = useNavigate();

  return (
    <div onClick={() => navigate(`/admin/sessions/${session.id}`)}>
      {session.title}
    </div>
  );
}
```

### Links
```typescript
import { Link } from 'react-router-dom';

export function Header() {
  return (
    <nav>
      <Link to="/admin/dashboard">Dashboard</Link>
      <Link to="/admin/sessions/create">Create Session</Link>
    </nav>
  );
}
```

## 5. Route State & Query Parameters

### Using URL Parameters
```typescript
import { useParams } from 'react-router-dom';

export function SessionDetail() {
  const { id } = useParams<{ id: string }>();
  // Fetch session with id
}
```

### Using Query Strings (Optional)
```typescript
import { useSearchParams } from 'react-router-dom';

export function SessionsList() {
  const [searchParams, setSearchParams] = useSearchParams();
  const status = searchParams.get('status') || 'open';

  return (
    <select value={status} onChange={e => setSearchParams({ status: e.target.value })}>
      <option value="open">Open</option>
      <option value="closed">Closed</option>
    </select>
  );
}
```

## 6. Logout Navigation

```typescript
// In Header or Menu
import { useNavigate } from 'react-router-dom';
import { useAuth } from '@/lib/hooks';

export function Header() {
  const navigate = useNavigate();
  const { logout } = useAuth();

  const handleLogout = () => {
    logout();
    navigate('/login', { replace: true });  // replace: true prevents back button to protected route
  };

  return (
    <button onClick={handleLogout}>Logout</button>
  );
}
```

## 7. Lazy Loading Routes (Future Optimization)

```typescript
import { lazy, Suspense } from 'react';
import { Spinner } from '@/components/common/Spinner';

const AdminDashboard = lazy(() => import('@/pages/AdminDashboard'));
const MemberDashboard = lazy(() => import('@/pages/MemberDashboard'));

function App() {
  return (
    <Routes>
      <Route
        path="/admin/dashboard"
        element={
          <Suspense fallback={<Spinner />}>
            <AdminDashboard />
          </Suspense>
        }
      />
    </Routes>
  );
}
```

## 8. 404 & Error Pages

```typescript
// src/pages/NotFound.tsx
import { useNavigate } from 'react-router-dom';
import { Button } from '@/components/common/Button';

export default function NotFound() {
  const navigate = useNavigate();

  return (
    <div className="flex flex-col items-center justify-center h-screen">
      <h1 className="text-4xl font-bold mb-4">404 Not Found</h1>
      <p className="text-gray-600 mb-6">The page you're looking for doesn't exist.</p>
      <Button onClick={() => navigate('/')}>Go to Home</Button>
    </div>
  );
}
```

## 9. Route Configuration Summary

| Route | Component | Auth Required | Role |
|---|---|---|---|
| `/login` | LoginPage | No | Any |
| `/admin/dashboard` | AdminDashboard | Yes | admin |
| `/admin/sessions/:id` | SessionDetail | Yes | admin |
| `/admin/sessions/create` | CreateSessionPage | Yes | admin |
| `/member/dashboard` | MemberDashboard | Yes | member |
| `/member/sessions/:id/checkin` | CheckinFlow | Yes | member |
| `/member/history` | AttendanceHistory | Yes | member |
| `/` | Redirect | No | Any |
| `*` | NotFound | No | Any |

## 10. Route Transitions (Animation)

```typescript
// src/App.tsx
import { AnimatePresence, motion } from 'framer-motion';
import { useLocation } from 'react-router-dom';

function App() {
  const location = useLocation();

  return (
    <AnimatePresence mode="wait">
      <motion.div
        key={location.pathname}
        initial={{ opacity: 0, y: 10 }}
        animate={{ opacity: 1, y: 0 }}
        exit={{ opacity: 0, y: -10 }}
        transition={{ duration: 0.2 }}
      >
        <Routes location={location}>
          {/* Routes here */}
        </Routes>
      </motion.div>
    </AnimatePresence>
  );
}
```
## 11. Phase 2 Route Extensions

Phase 2 extends the existing routing structure to support continuous attendance monitoring, administrator review workflows, and attendance summaries.

---

## 11.1 Updated Route Hierarchy

```
/
├── /login
├── /admin
│   ├── /dashboard
│   ├── /sessions
│   ├── /sessions/create
│   └── /sessions/:id
│       ├── Overview
│       ├── Live Monitoring
│       ├── Early Check-out Requests
│       └── Reports
└── /member
    ├── /dashboard
    ├── /sessions/:id/checkin
    ├── /sessions/:id/active
    ├── /sessions/:id/summary
    ├── /history
    └── /profile
```

---

## 11.2 New Member Routes

| Route | Purpose |
|--------|---------|
| `/member/sessions/:id/active` | Active attendance session |
| `/member/sessions/:id/summary` | Attendance summary after successful check-out |
| `/member/profile` | Member profile and preferences (future-ready) |

---

## 11.3 Active Session Navigation

Member navigation flow:

```
Dashboard

↓

Check In

↓

Active Session

↓

Attendance Summary

↓

Dashboard
```

If an Active Session exists, selecting **Open Session** from the dashboard always returns the member to the active attendance page.

---

## 11.4 Early Check-out Navigation

When a member requests early check-out:

```
Active Session

↓

Request Early Check-out

↓

Pending Approval

↓

Administrator Decision

├── Approved
│     ↓
│ Complete Check-out
│     ↓
│ Attendance Summary
│
└── Rejected
      ↓
Return to Active Session
```

---

## 11.5 Administrator Navigation

Session Detail becomes the central workspace for administrators.

Within a session, administrators can access:

- Session Overview
- Live Attendance Monitoring
- Early Check-out Requests
- Attendance Reports

Navigation between these sections should not require leaving the Session Detail page.

---

## 11.6 Route Protection

All new Phase 2 routes inherit the existing authentication rules.

Member-only routes:

- Active Session
- Attendance Summary
- Attendance History
- Profile

Administrator-only functionality:

- Live Monitoring
- Early Check-out Review
- Reports

---

## 11.7 Navigation Principles

- Users should never manually type URLs to continue attendance.
- Active sessions should always be reachable from the dashboard.
- Attendance Summary is displayed only after successful attendance completion.
- After attendance completion, navigation returns naturally to the dashboard.
- Protected routes must remain inaccessible after logout.

---

## 11.8 Route Summary

| Route | Component | Role |
|--------|-----------|------|
| `/member/dashboard` | MemberDashboard | Member |
| `/member/sessions/:id/checkin` | CheckInFlow | Member |
| `/member/sessions/:id/active` | ActiveSessionPage | Member |
| `/member/sessions/:id/summary` | AttendanceSummaryPage | Member |
| `/member/history` | AttendanceHistory | Member |
| `/member/profile` | MemberProfile | Member |
| `/admin/dashboard` | AdminDashboard | Admin |
| `/admin/sessions/:id` | SessionDetail | Admin |
