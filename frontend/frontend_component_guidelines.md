# frontend_component_guidelines.md — Component Architecture & Patterns

> How to structure React components: file organization, reusable patterns, and shadcn/ui integration.

## 1. Component Folder Structure

```
src/
├── components/
│   ├── common/              # Reusable across app
│   │   ├── Button.tsx
│   │   ├── Card.tsx
│   │   ├── Modal.tsx
│   │   ├── Spinner.tsx
│   │   └── StatusBadge.tsx
│   ├── forms/               # Form-specific components
│   │   ├── LoginForm.tsx
│   │   ├── SessionForm.tsx
│   │   └── CheckinForm.tsx
│   ├── layout/              # App structure
│   │   ├── Header.tsx
│   │   ├── Sidebar.tsx
│   │   └── MainLayout.tsx
│   └── features/            # Feature-specific components
│       ├── session/
│       │   ├── SessionList.tsx
│       │   ├── SessionDetail.tsx
│       │   └── SessionCard.tsx
│       └── attendance/
│           ├── CheckinFlow.tsx
│           ├── CameraCapture.tsx
│           └── AttendanceHistory.tsx
├── pages/
│   ├── LoginPage.tsx
│   ├── AdminDashboard.tsx
│   ├── MemberDashboard.tsx
│   └── NotFound.tsx
├── lib/
│   ├── api.ts               # API client
│   ├── auth.ts              # Auth helpers
│   └── hooks.ts             # Custom hooks
└── main.tsx
```

## 2. Component File Template

```typescript
// src/components/common/Button.tsx
import React from 'react';
import { cn } from '@/lib/utils';  // shadcn utility

interface ButtonProps extends React.ButtonHTMLAttributes<HTMLButtonElement> {
  variant?: 'primary' | 'secondary' | 'danger';
  size?: 'sm' | 'md' | 'lg';
  loading?: boolean;
}

export const Button: React.FC<ButtonProps> = ({
  variant = 'primary',
  size = 'md',
  loading = false,
  children,
  disabled,
  className,
  ...props
}) => {
  const baseStyles = 'font-medium rounded-lg transition-colors focus:outline-none focus:ring-2';

  const variantStyles = {
    primary: 'bg-blue-500 hover:bg-blue-600 text-white focus:ring-blue-200',
    secondary: 'bg-gray-100 hover:bg-gray-200 text-gray-800 focus:ring-gray-200',
    danger: 'bg-red-500 hover:bg-red-600 text-white focus:ring-red-200',
  };

  const sizeStyles = {
    sm: 'px-3 py-1 text-sm',
    md: 'px-4 py-2 text-base',
    lg: 'px-6 py-3 text-lg',
  };

  return (
    <button
      disabled={disabled || loading}
      className={cn(
        baseStyles,
        variantStyles[variant],
        sizeStyles[size],
        disabled && 'opacity-50 cursor-not-allowed',
        className
      )}
      {...props}
    >
      {loading ? <Spinner size="sm" /> : children}
    </button>
  );
};
```

## 3. Using shadcn/ui

shadcn/ui provides unstyled, accessible components built on Radix UI.

**Install a component:**
```bash
npx shadcn-ui@latest add button
npx shadcn-ui@latest add card
npx shadcn-ui@latest add input
npx shadcn-ui@latest add dialog
npx shadcn-ui@latest add select
```

**Usage:**
```typescript
import { Button } from '@/components/ui/button';
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card';
import { Input } from '@/components/ui/input';

export function MyComponent() {
  return (
    <Card>
      <CardHeader>
        <CardTitle>Welcome</CardTitle>
      </CardHeader>
      <CardContent>
        <Input placeholder="Enter email" />
        <Button>Submit</Button>
      </CardContent>
    </Card>
  );
}
```

**Don't rebuild Button, Input, etc. — use shadcn versions, then style with className.**

## 4. Form Component Pattern

```typescript
// src/components/forms/LoginForm.tsx
import React, { useState } from 'react';
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';
import { Button } from '@/components/common/Button';
import { Input } from '@/components/ui/input';
import { useAuth } from '@/lib/hooks';

const loginSchema = z.object({
  email: z.string().email('Invalid email'),
  password: z.string().min(8, 'Password must be at least 8 characters'),
});

type LoginFormData = z.infer<typeof loginSchema>;

export function LoginForm() {
  const { login } = useAuth();
  const [error, setError] = useState<string | null>(null);
  
  const { register, handleSubmit, formState: { errors, isSubmitting } } = useForm<LoginFormData>({
    resolver: zodResolver(loginSchema),
  });

  const onSubmit = async (data: LoginFormData) => {
    try {
      await login(data.email, data.password);
    } catch (err) {
      setError(err instanceof Error ? err.message : 'Login failed');
    }
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)} className="space-y-4">
      <div>
        <label className="block text-sm font-medium mb-1">Email</label>
        <Input
          {...register('email')}
          type="email"
          placeholder="Enter email"
          className={errors.email ? 'border-red-500' : ''}
        />
        {errors.email && <span className="text-red-500 text-sm">{errors.email.message}</span>}
      </div>

      <div>
        <label className="block text-sm font-medium mb-1">Password</label>
        <Input
          {...register('password')}
          type="password"
          placeholder="Enter password"
          className={errors.password ? 'border-red-500' : ''}
        />
        {errors.password && <span className="text-red-500 text-sm">{errors.password.message}</span>}
      </div>

      {error && <div className="bg-red-100 text-red-800 p-3 rounded">{error}</div>}

      <Button type="submit" loading={isSubmitting} className="w-full">
        Login
      </Button>
    </form>
  );
}
```

## 5. Async Data Fetching (React Query)

```typescript
// src/components/features/session/SessionList.tsx
import { useQuery } from '@tanstack/react-query';
import { fetchSessions } from '@/lib/api';
import { Spinner } from '@/components/common/Spinner';
import { SessionCard } from './SessionCard';

export function SessionList() {
  const { data: sessions, isLoading, error } = useQuery({
    queryKey: ['sessions'],
    queryFn: fetchSessions,
  });

  if (isLoading) return <Spinner />;
  if (error) return <div className="text-red-500">Error loading sessions</div>;
  if (!sessions?.length) return <p className="text-gray-500">No sessions available</p>;

  return (
    <div className="grid gap-4">
      {sessions.map(session => (
        <SessionCard key={session.id} session={session} />
      ))}
    </div>
  );
}
```

## 6. Custom Hooks Pattern

```typescript
// src/lib/hooks.ts
import { useState, useCallback } from 'react';
import * as api from './api';

export function useAuth() {
  const [user, setUser] = useState(null);
  const [isAuthenticated, setIsAuthenticated] = useState(false);

  const login = useCallback(async (email: string, password: string) => {
    const response = await api.login(email, password);
    localStorage.setItem('access_token', response.access_token);
    setUser(response.user);
    setIsAuthenticated(true);
  }, []);

  const logout = useCallback(() => {
    localStorage.removeItem('access_token');
    setUser(null);
    setIsAuthenticated(false);
  }, []);

  return { user, isAuthenticated, login, logout };
}

export function useGeolocation() {
  const [location, setLocation] = useState<{lat: number; lng: number} | null>(null);
  const [error, setError] = useState<string | null>(null);

  const requestLocation = useCallback(async () => {
    if (!navigator.geolocation) {
      setError('Geolocation is not supported');
      return;
    }

    navigator.geolocation.getCurrentPosition(
      position => {
        setLocation({
          lat: position.coords.latitude,
          lng: position.coords.longitude,
        });
      },
      err => {
        setError(err.message);
      }
    );
  }, []);

  return { location, error, requestLocation };
}
```

## 7. Modal/Dialog Pattern

```typescript
// src/components/common/Modal.tsx
import { Dialog, DialogContent, DialogHeader, DialogTitle } from '@/components/ui/dialog';

interface ModalProps {
  open: boolean;
  onOpenChange: (open: boolean) => void;
  title: string;
  children: React.ReactNode;
}

export function Modal({ open, onOpenChange, title, children }: ModalProps) {
  return (
    <Dialog open={open} onOpenChange={onOpenChange}>
      <DialogContent className="sm:max-w-[600px]">
        <DialogHeader>
          <DialogTitle>{title}</DialogTitle>
        </DialogHeader>
        {children}
      </DialogContent>
    </Dialog>
  );
}

// Usage:
const [isOpen, setIsOpen] = useState(false);
return (
  <>
    <Button onClick={() => setIsOpen(true)}>Create Session</Button>
    <Modal open={isOpen} onOpenChange={setIsOpen} title="Create New Session">
      <SessionForm onClose={() => setIsOpen(false)} />
    </Modal>
  </>
);
```

## 8. Camera Capture Pattern (Phase 1)

```typescript
// src/components/features/attendance/CameraCapture.tsx
import { useRef, useState } from 'react';
import { Button } from '@/components/common/Button';

interface CameraCaptureProps {
  onCapture: (blob: Blob) => void;
}

export function CameraCapture({ onCapture }: CameraCaptureProps) {
  const videoRef = useRef<HTMLVideoElement>(null);
  const canvasRef = useRef<HTMLCanvasElement>(null);
  const [streaming, setStreaming] = useState(false);

  const startCamera = async () => {
    try {
      const stream = await navigator.mediaDevices.getUserMedia({
        video: { facingMode: 'user' },
      });
      if (videoRef.current) {
        videoRef.current.srcObject = stream;
        setStreaming(true);
      }
    } catch (err) {
      console.error('Camera error:', err);
    }
  };

  const capturePhoto = () => {
    if (videoRef.current && canvasRef.current) {
      const ctx = canvasRef.current.getContext('2d');
      if (ctx) {
        ctx.drawImage(videoRef.current, 0, 0, canvasRef.current.width, canvasRef.current.height);
        canvasRef.current.toBlob(blob => {
          if (blob) onCapture(blob);
        });
      }
    }
  };

  return (
    <div className="space-y-4">
      <video
        ref={videoRef}
        autoPlay
        playsInline
        className="w-full rounded-lg"
        style={{ transform: 'scaleX(-1)' }}
      />
      <canvas ref={canvasRef} style={{ display: 'none' }} />
      {!streaming && <Button onClick={startCamera}>Start Camera</Button>}
      {streaming && <Button onClick={capturePhoto}>Capture Photo</Button>}
    </div>
  );
}
```

## 9. Animation Example (Framer Motion)

```typescript
import { motion } from 'framer-motion';

export function CheckinSuccess() {
  return (
    <motion.div
      initial={{ opacity: 0, scale: 0.9 }}
      animate={{ opacity: 1, scale: 1 }}
      transition={{ duration: 0.3 }}
      className="p-6 bg-green-50 border border-green-200 rounded-lg"
    >
      <h2 className="text-green-800 font-semibold">✓ Check-in Successful</h2>
    </motion.div>
  );
}
```

## 10. Component Checklist

Before marking a component as done:

- [ ] Props are typed with TypeScript
- [ ] Component is reusable (not hardcoded values)
- [ ] Handles loading state
- [ ] Handles error state
- [ ] Has proper focus/keyboard handling
- [ ] Styled with Tailwind + shadcn
- [ ] Responsive (mobile-first)
- [ ] Has unit tests (simple ones)


## 11. Phase 2 Component Architecture

Phase 2 extends the component architecture to support the complete attendance lifecycle, live monitoring, and administrator workflows.

---

## 11.1 Updated Feature Structure

```
src/
├── components/
│
├── common/
│   ├── LoadingSkeleton.tsx
│   ├── EmptyState.tsx
│   ├── StatusBadge.tsx
│   ├── ConfirmationDialog.tsx
│   └── ProgressBar.tsx
│
├── layout/
│   ├── Sidebar.tsx
│   ├── DashboardLayout.tsx
│   └── PageHeader.tsx
│
├── features/
│
│   ├── attendance/
│   │   ├── ActiveSessionCard.tsx
│   │   ├── ActiveSessionDetails.tsx
│   │   ├── PresenceTimeline.tsx
│   │   ├── AttendanceSummary.tsx
│   │   ├── AttendanceStatus.tsx
│   │   ├── SessionProgress.tsx
│   │   ├── EarlyCheckoutDialog.tsx
│   │   ├── ApprovalStatusBanner.tsx
│   │   └── AttendanceHistory.tsx
│
│   ├── monitoring/
│   │   ├── LiveAttendancePanel.tsx
│   │   ├── MemberAttendanceTable.tsx
│   │   ├── PendingRequestsTable.tsx
│   │   ├── MonitoringStatistics.tsx
│   │   └── PresenceTimeline.tsx
│
│   └── session/
│       ├── SessionCard.tsx
│       ├── SessionList.tsx
│       ├── SessionOverview.tsx
│       └── SessionDetail.tsx
```

---

## 11.2 Dashboard Layout Components

The dashboard should be composed of reusable layout components.

```
DashboardLayout

├── Sidebar

├── PageHeader

├── ActiveSessionCard

├── AvailableSessionsGrid

└── AttendanceHistory
```

Each section should remain independently reusable.

---

## 11.3 Active Session Components

The Active Session page should consist of small focused components.

```
ActiveSessionPage

├── SessionHeader

├── AttendanceStatus

├── SessionProgress

├── PresenceTimeline

├── ApprovalStatusBanner

└── ActionButtons
```

Avoid creating one large component containing all logic.

---

## 11.4 Timeline Component

The timeline should be reusable by both Members and Administrators.

Props:

```typescript
interface PresenceTimelineProps {
    events: TimelineEvent[];
}
```

The component only renders data.

Fetching should occur in the parent component.

---

## 11.5 Status Components

Attendance status should always be displayed through reusable components.

Example:

```
<AttendanceStatus
    status="Present"
/>

<AttendanceStatus
    status="Pending Approval"
/>
```

Avoid repeating status color logic throughout the application.

---

## 11.6 Progress Component

Session progress should be implemented as a reusable component.

Example:

```
<SessionProgress
    current={45}
    total={60}
/>
```

The component is responsible only for presentation.

---

## 11.7 Early Check-out Dialog

The dialog should contain:

- Reason field
- Validation messages
- Cancel button
- Submit button

Business logic belongs inside hooks, not inside the dialog.

---

## 11.8 Administrator Components

Administrator monitoring should reuse small focused components.

```
SessionDetail

├── SessionOverview

├── MonitoringStatistics

├── MemberAttendanceTable

├── PendingRequestsTable

└── PresenceTimeline
```

Each component should have a single responsibility.

---

## 11.9 Component Responsibilities

Presentation Components

Responsible for:

- Rendering UI
- Styling
- User interaction

Should NOT:

- Call APIs directly
- Perform business logic

Container Components

Responsible for:

- Fetching data
- Managing mutations
- Passing props
- Handling loading and error states

---

## 11.10 Component Communication

Parent components should own server state.

Example:

```
Page

↓

Fetch Data

↓

Pass Props

↓

Child Components
```

Avoid unnecessary prop drilling.

Use Context only for shared application state such as authentication.

---

## 11.11 Component Guidelines

Every new Phase 2 component should:

- Be reusable.
- Be strongly typed.
- Have a single responsibility.
- Handle loading states.
- Handle empty states.
- Handle error states.
- Follow the design system.
- Be responsive.
- Support keyboard accessibility.

---

## 11.12 Component Naming

Use descriptive PascalCase names.

Examples:

```
ActiveSessionCard

AttendanceSummary

PresenceTimeline

SessionProgress

MonitoringStatistics

PendingRequestsTable

ApprovalStatusBanner

EarlyCheckoutDialog
```

Avoid generic names such as:

```
Card2

Panel

Widget

Component
```

Names should clearly describe the component's purpose.

---

## 11.13 Phase 2 Design Principles

All new components should follow these principles:

- Small and composable.
- Presentation separated from business logic.
- Reusable across multiple pages.
- Built using Tailwind CSS and shadcn/ui.
- Consistent with the established design system.
- Optimized for desktop-first layouts while remaining responsive.

# 12. Phase 3 Activity Layer Component Architecture

Phase 3 extends the component architecture by introducing reusable components that support activity planning, volunteer assignments, progress tracking, evidence submission, administrative review, templates, and activity reporting.

The same architectural principles continue to apply:

- presentation separated from business logic;
- reusable components;
- strong typing;
- consistent design system;
- accessibility;
- responsive layouts.

---

## 12.1 Activity Feature Structure

```
src/

├── components/

│   └── features/

│       └── activity/

│           ├── ActivityCard.tsx
│           ├── ActivityList.tsx
│           ├── ActivityHeader.tsx
│           ├── ActivitySummary.tsx
│           ├── ActivityDetail.tsx
│           ├── ActivityTimeline.tsx
│           ├── TimelineEntry.tsx
│           ├── AssignmentPanel.tsx
│           ├── VolunteerSelector.tsx
│           ├── ProgressEditor.tsx
│           ├── ProgressHistory.tsx
│           ├── EvidenceGallery.tsx
│           ├── EvidenceUploader.tsx
│           ├── ImagePreview.tsx
│           ├── VideoPreview.tsx
│           ├── ReviewPanel.tsx
│           ├── ReviewSummary.tsx
│           ├── TemplateCard.tsx
│           ├── TemplateList.tsx
│           ├── ActivityReportCard.tsx
│           └── ActivityStatusBadge.tsx
```

The Activity feature should remain isolated from attendance components while reusing common components whenever practical.

---

## 12.2 Activity Detail Composition

The Activity Detail page should be composed of small reusable components.

```
ActivityDetail

├── ActivityHeader

├── ActivitySummary

├── AssignmentPanel

├── ActivityTimeline

├── ProgressHistory

├── EvidenceGallery

├── ReviewSummary

└── ActivityActions
```

Avoid implementing the entire screen as one large component.

---

## 12.3 Activity Card

The Activity Card provides a summarized view of an activity.

Typical information includes:

- activity title;
- priority;
- status;
- assigned volunteers;
- progress summary.

The component should remain presentation-focused.

---

## 12.4 Assignment Components

Assignment functionality should be divided into focused components.

Example:

```
AssignmentPanel

├── VolunteerSelector

├── AssignmentList

└── AssignmentStatus
```

Assignment management logic belongs in parent containers or hooks.

---

## 12.5 Timeline Components

The timeline should remain reusable throughout the application.

Example:

```typescript
interface ActivityTimelineProps {
    entries: TimelineEntry[];
}
```

The component should render timeline information only.

Data retrieval belongs elsewhere.

---

## 12.6 Evidence Components

Evidence functionality should be separated into reusable components.

Examples include:

- EvidenceUploader;
- EvidenceGallery;
- ImagePreview;
- VideoPreview.

Each component should perform a single responsibility.

---

## 12.7 Review Components

Administrative review should consist of reusable presentation components.

Example:

```
ReviewPanel

├── SubmissionSummary

├── EvidenceGallery

├── ReviewRemarks

└── ReviewActions
```

Business decisions remain outside presentation components.

---

## 12.8 Template Components

Reusable activity templates should be represented through dedicated components.

Examples include:

- TemplateCard;
- TemplateList;
- TemplatePreview.

Template selection and creation should remain independent concerns.

---

## 12.9 Status Components

Activity-related status indicators should use reusable components.

Examples include:

```
<ActivityStatusBadge />

<ReviewStatusBadge />

<PriorityBadge />
```

Avoid duplicating badge styling throughout the application.

---

## 12.10 Upload Components

Upload functionality should provide reusable behavior for photographs and videos.

Components should communicate:

- upload progress;
- upload success;
- upload failure;
- retry availability.

Upload logic should remain independent from presentation.

---

## 12.11 Empty State Components

Reusable empty states should support situations including:

- no activities;
- no assignments;
- no templates;
- no progress history;
- no reports.

Empty states should encourage the user's next logical action.

---

## 12.12 Loading Components

Activity interfaces should use reusable loading components.

Examples include:

- ActivitySkeleton;
- TimelineSkeleton;
- GallerySkeleton;
- ReviewSkeleton.

Loading indicators should closely resemble the final layout.

---

## 12.13 Error Components

Reusable error components should communicate:

- loading failures;
- upload failures;
- synchronization failures;
- permission failures.

Recovery actions should remain obvious.

---

## 12.14 Component Communication

Activity components should communicate using parent-owned state.

Typical flow:

```
Container Component

↓

Fetch Activity Data

↓

Pass Props

↓

Presentation Components
```

Presentation components should never request backend data directly.

---

## 12.15 Activity Hooks

Business behavior should remain inside reusable hooks.

Examples include:

- useActivities()
- useActivity()
- useAssignments()
- useProgress()
- useEvidence()
- useReview()
- useTemplates()

Hooks should coordinate backend communication while components focus on rendering.

---

## 12.16 Accessibility

Activity components should support:

- keyboard navigation;
- screen readers;
- visible focus indicators;
- descriptive labels;
- accessible media previews.

Accessibility should be considered during component design rather than added afterward.

---

## 12.17 Phase 3 Design Principles

All Activity Layer components should be:

- small and composable;
- reusable across multiple pages;
- presentation-focused;
- strongly typed;
- responsive;
- accessible;
- consistent with the design system;
- independent of business logic.

# 13. Phase 4 Production Component Architecture

Phase 4 extends the component architecture to support production-ready capabilities including offline operation, automatic synchronization, audit and security management, and operational status monitoring.

These components continue following the existing architectural principles:

- presentation separated from business logic;
- reusable components;
- strong typing;
- accessibility;
- responsive layouts;
- backend authority.

---

## 13.1 Production Feature Structure

```
src/

├── components/

│   └── features/

│       ├── synchronization/
│       │   ├── SyncStatusIndicator.tsx
│       │   ├── SyncStatusPanel.tsx
│       │   └── SyncStatusBadge.tsx
│       │
│       ├── audit/
│       │   ├── AuditFilterBar.tsx
│       │   ├── AuditTable.tsx
│       │   ├── SecurityEventTable.tsx
│       │   ├── AuditSummaryCards.tsx
│       │   └── EventDetailDrawer.tsx
│       │
│       └── production/
│           ├── OfflineBanner.tsx
│           ├── ConnectionStatus.tsx
│           └── ProductionStatusBadge.tsx
```

Production components should remain independent from attendance and activity components whenever practical.

---

## 13.2 Offline Components

Offline functionality should be represented through reusable presentation components.

Examples include:

- OfflineBanner;
- ConnectionStatus;
- SyncStatusIndicator.

These components should communicate connectivity status without exposing implementation details.

---

## 13.3 Synchronization Components

Synchronization should use reusable components shared throughout the application.

Examples include:

```
<SyncStatusIndicator />

<SyncStatusPanel />

<SyncStatusBadge />
```

The global synchronization indicator should remain lightweight and accessible from every authenticated screen.

Synchronization logic belongs inside reusable hooks rather than presentation components.

---

## 13.4 Audit & Security Components

The Audit & Security experience should be composed from reusable components.

Example:

```
AuditSecurityPage

├── AuditSummaryCards

├── AuditFilterBar

├── AuditTable

├── SecurityEventTable

└── EventDetailDrawer
```

Each component should maintain a single responsibility.

Audit and security information should remain visually unified while consuming independent backend data.

---

## 13.5 Production Status Components

Production status should be communicated using reusable components.

Examples include:

- ConnectionStatus;
- ProductionStatusBadge;
- SyncStatusBadge.

Status components should communicate operational information consistently throughout the application.

---

## 13.6 Component Communication

Production components should continue using parent-owned state.

Typical flow:

```
Container Component

↓

Reusable Hooks

↓

Presentation Components
```

Presentation components should never communicate directly with backend services.

---

## 13.7 Production Hooks

Production functionality should remain inside reusable hooks.

Examples include:

- useOfflineQueue()
- useSynchronization()
- useAuditLogs()
- useSecurityEvents()

Hooks should coordinate backend communication, synchronization, and state updates while presentation components focus exclusively on rendering.

---

## 13.8 Accessibility

Production components should support:

- keyboard navigation;
- screen readers;
- visible focus indicators;
- accessible status announcements;
- meaningful icons with accompanying text.

Connectivity and synchronization status should be understandable without relying solely on color.

---

## 13.9 Phase 4 Design Principles

All Phase 4 production components should be:

- small and composable;
- strongly typed;
- reusable;
- presentation-focused;
- independent of business logic;
- accessible;
- responsive;
- consistent with the existing design system;
- suitable for production-ready operational workflows.