# frontend_state_management.md — State Management & Data Fetching

> How to manage state: local component state, context for auth, React Query for server state.

## 1. State Management Strategy

| State Type | Tool | Usage |
|---|---|---|
| Component local state | React `useState` | Form inputs, UI toggles, modals |
| Auth context | React Context + custom hook | Current user, login/logout |
| Server state (API data) | React Query (TanStack Query) | Sessions, check-ins, reports |
| App settings | localStorage + context | Theme (future), UI preferences |

## 2. Auth Context Setup

```typescript
// src/lib/auth.ts
import { createContext, useContext, useState, useCallback, ReactNode } from 'react';
import * as api from './api';

interface User {
  id: string;
  full_name: string;
  email: string;
  role: 'admin' | 'member';
}

interface AuthContextType {
  user: User | null;
  isAuthenticated: boolean;
  login: (email: string, password: string) => Promise<void>;
  logout: () => void;
}

const AuthContext = createContext<AuthContextType | undefined>(undefined);

export function AuthProvider({ children }: { children: ReactNode }) {
  const [user, setUser] = useState<User | null>(() => {
    // Restore from localStorage if available
    const stored = localStorage.getItem('user');
    return stored ? JSON.parse(stored) : null;
  });

  const login = useCallback(async (email: string, password: string) => {
    const response = await api.login(email, password);
    localStorage.setItem('access_token', response.access_token);
    localStorage.setItem('user', JSON.stringify(response.user));
    setUser(response.user);
  }, []);

  const logout = useCallback(() => {
    localStorage.removeItem('access_token');
    localStorage.removeItem('user');
    setUser(null);
  }, []);

  return (
    <AuthContext.Provider value={{ user, isAuthenticated: !!user, login, logout }}>
      {children}
    </AuthContext.Provider>
  );
}

export function useAuth() {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error('useAuth must be used within AuthProvider');
  }
  return context;
}
```

**Wrap app with provider:**
```typescript
// src/main.tsx
import { AuthProvider } from '@/lib/auth';

ReactDOM.createRoot(document.getElementById('root')!).render(
  <React.StrictMode>
    <AuthProvider>
      <Router>
        <App />
      </Router>
    </AuthProvider>
  </React.StrictMode>,
);
```

## 3. React Query Setup

```typescript
// src/lib/queryClient.ts
import { QueryClient } from '@tanstack/react-query';

export const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 1000 * 60 * 5,  // 5 minutes
      gcTime: 1000 * 60 * 10,    // 10 minutes
      retry: 1,
      refetchOnWindowFocus: false,
    },
  },
});

// src/main.tsx
import { QueryClientProvider } from '@tanstack/react-query';
import { queryClient } from '@/lib/queryClient';

ReactDOM.createRoot(document.getElementById('root')!).render(
  <React.StrictMode>
    <QueryClientProvider client={queryClient}>
      <AuthProvider>
        <Router>
          <App />
        </Router>
      </AuthProvider>
    </QueryClientProvider>
  </React.StrictMode>,
);
```

## 4. Fetching Data with React Query

```typescript
// src/lib/hooks.ts
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import * as api from './api';

// Query: Fetch sessions
export function useSessions() {
  return useQuery({
    queryKey: ['sessions'],
    queryFn: api.fetchSessions,
  });
}

// Query: Fetch session detail
export function useSession(id: string) {
  return useQuery({
    queryKey: ['sessions', id],
    queryFn: () => api.fetchSession(id),
  });
}

// Query: Fetch attendance history
export function useAttendanceHistory() {
  return useQuery({
    queryKey: ['attendance', 'history'],
    queryFn: api.fetchAttendanceHistory,
  });
}

// Mutation: Create session
export function useCreateSession() {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: api.createSession,
    onSuccess: () => {
      // Invalidate sessions list so it refetches
      queryClient.invalidateQueries({ queryKey: ['sessions'] });
    },
  });
}

// Mutation: Submit check-in
export function useCheckIn() {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: api.checkIn,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['attendance', 'history'] });
    },
  });
}

// Mutation: Close session
export function useCloseSession() {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: (sessionId: string) => api.closeSession(sessionId),
    onSuccess: (_, sessionId) => {
      // Invalidate this session's detail
      queryClient.invalidateQueries({ queryKey: ['sessions', sessionId] });
      // Invalidate sessions list
      queryClient.invalidateQueries({ queryKey: ['sessions'] });
    },
  });
}
```

## 5. Using Queries in Components

```typescript
// src/components/features/session/SessionList.tsx
import { useSessions } from '@/lib/hooks';
import { Spinner } from '@/components/common/Spinner';

export function SessionList() {
  const { data: sessions, isLoading, error } = useSessions();

  if (isLoading) return <Spinner />;
  if (error) return <div className="text-red-500">Error loading sessions</div>;
  if (!sessions?.length) return <p>No sessions available</p>;

  return (
    <div className="grid gap-4">
      {sessions.map(session => (
        <SessionCard key={session.id} session={session} />
      ))}
    </div>
  );
}
```

## 6. Using Mutations in Components

```typescript
// src/pages/CreateSessionPage.tsx
import { useCreateSession } from '@/lib/hooks';
import { useNavigate } from 'react-router-dom';
import { SessionForm } from '@/components/forms/SessionForm';

export default function CreateSessionPage() {
  const navigate = useNavigate();
  const { mutate: createSession, isPending } = useCreateSession();

  const handleSubmit = (data: SessionCreateData) => {
    createSession(data, {
      onSuccess: () => {
        navigate('/admin/dashboard');
      },
      onError: (error) => {
        console.error('Failed to create session:', error);
      },
    });
  };

  return (
    <SessionForm
      onSubmit={handleSubmit}
      isLoading={isPending}
    />
  );
}
```

## 7. Form State Management

```typescript
// src/components/forms/SessionForm.tsx
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

const sessionSchema = z.object({
  title: z.string().min(1, 'Title is required'),
  purpose: z.string().optional(),
  date: z.string().refine(d => new Date(d) > new Date()),
  start_time: z.string(),
  end_time: z.string(),
  grace_period_minutes: z.number().min(0),
  venue_lat: z.number(),
  venue_lng: z.number(),
  radius_meters: z.number().min(1),
});

type SessionFormData = z.infer<typeof sessionSchema>;

interface SessionFormProps {
  onSubmit: (data: SessionFormData) => void;
  isLoading: boolean;
}

export function SessionForm({ onSubmit, isLoading }: SessionFormProps) {
  const {
    register,
    handleSubmit,
    formState: { errors },
    watch,
  } = useForm<SessionFormData>({
    resolver: zodResolver(sessionSchema),
  });

  const startTime = watch('start_time');

  return (
    <form onSubmit={handleSubmit(onSubmit)} className="space-y-4">
      {/* Form fields */}
      <input {...register('title')} type="text" placeholder="Session title" />
      {errors.title && <span className="text-red-500">{errors.title.message}</span>}

      <input {...register('date')} type="date" />
      {errors.date && <span className="text-red-500">{errors.date.message}</span>}

      <input {...register('start_time')} type="time" />
      <input {...register('end_time')} type="time" min={startTime} />

      <button type="submit" disabled={isLoading}>
        {isLoading ? 'Creating...' : 'Create Session'}
      </button>
    </form>
  );
}
```

## 8. Optimistic Updates (Advanced)

```typescript
// In a mutation, update UI before server responds
export function useCheckIn() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: api.checkIn,
    onMutate: async (checkInData) => {
      // Cancel in-flight queries
      await queryClient.cancelQueries({ queryKey: ['attendance', 'history'] });

      // Get old data
      const oldData = queryClient.getQueryData(['attendance', 'history']);

      // Optimistically update
      queryClient.setQueryData(['attendance', 'history'], (old: any) => [
        ...old,
        {
          ...checkInData,
          final_status: 'present', // Optimistic assumption
          check_in_time: new Date().toISOString(),
        },
      ]);

      return { oldData };
    },
    onError: (err, _, context) => {
      // Revert on error
      if (context?.oldData) {
        queryClient.setQueryData(['attendance', 'history'], context.oldData);
      }
    },
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['attendance', 'history'] });
    },
  });
}
```

## 9. Persisting Auth Across Sessions

```typescript
// src/lib/auth.ts
// On app load, restore auth from localStorage

export function useAuthPersist() {
  const { user, setUser } = useAuth();

  useEffect(() => {
    const token = localStorage.getItem('access_token');
    const userData = localStorage.getItem('user');

    if (token && userData) {
      setUser(JSON.parse(userData));
    }
  }, []);
}

// In App.tsx, call once on mount
import { useAuthPersist } from '@/lib/auth';

function App() {
  useAuthPersist();
  return <Routes>{/* ... */}</Routes>;
}
```

## 10. Error Handling & Retry

```typescript
// API client with retry logic
import axios from 'axios';

const apiClient = axios.create({
  baseURL: import.meta.env.VITE_API_BASE_URL,
});

// Add auth token to requests
apiClient.interceptors.request.use(config => {
  const token = localStorage.getItem('access_token');
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});

// Handle 401 — logout user
apiClient.interceptors.response.use(
  response => response,
  error => {
    if (error.response?.status === 401) {
      // Clear auth and redirect to login
      localStorage.removeItem('access_token');
      window.location.href = '/login';
    }
    return Promise.reject(error);
  }
);

export default apiClient;
```

## 11. Query Invalidation Patterns

```typescript
// Manual invalidation when needed
const queryClient = useQueryClient();

// Invalidate specific query
queryClient.invalidateQueries({ queryKey: ['sessions', sessionId] });

// Invalidate all queries with a prefix
queryClient.invalidateQueries({ queryKey: ['sessions'] });

// Invalidate with a predicate
queryClient.invalidateQueries({
  predicate: query => query.queryKey[0] === 'sessions',
});
```

## 12. State Management Summary

| Use Case | Tool | Example |
|---|---|---|
| Form input state | `useState` | Email input in LoginForm |
| Auth context | Context API + localStorage | Current user, login/logout |
| API data | React Query | Sessions, check-ins |
| UI toggles | `useState` | Modal open/close |
| Complex local state | `useReducer` (Phase 2) | Multi-step check-in flow |
