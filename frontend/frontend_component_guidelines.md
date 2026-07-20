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
