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
