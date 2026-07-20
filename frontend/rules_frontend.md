# rules_frontend.md — Frontend Development Rules

> For the frontend developer and any agent (Claude Code, Kiro) writing frontend code in
> `/frontend`. The product goal for this UI is explicitly a **rich, premium feel** — not a bare
> functional form. These rules exist to get there consistently while staying learnable for a
> first-time frontend developer.

## 1. Project layout

Follow `system_architecture.md §6`. Route-level views go in `src/pages/`, reusable pieces in
`src/components/`, API/auth helpers in `src/lib/`.

## 2. Stack conventions

| Concern | Convention |
|---|---|
| Framework | React + TypeScript, built with Vite |
| Styling | Tailwind CSS utility classes; no separate hand-written CSS files unless truly necessary |
| Component library | shadcn/ui (Radix-based) for buttons, dialogs, forms, cards, tables — don't hand-roll a component shadcn already provides |
| Motion/polish | Framer Motion for transitions (page transitions, status changes, check-in confirmation) — used deliberately, not on every element |
| State/data fetching | TanStack Query (React Query) for all API calls — gives you loading/error states and caching for free, important since beginners often forget to handle these |
| Forms | React Hook Form + Zod for validation, matching the shapes in `API_contract.md` |
| Routing | React Router |
| Icons | lucide-react (pairs with shadcn/ui) |

## 3. "Premium feel" — concrete guidance, not vibes

Because "make it look premium" is subjective, here's what that means concretely for Phase 1:

- Consistent spacing scale (use Tailwind's default scale — don't invent arbitrary pixel values).
- One accent color + neutral grays; don't use more than 2-3 colors total in the UI.
- Every async action (login, check-in submit, export) has a visible loading state — no dead
  buttons or silent waits.
- Every error is shown as a real message from `API_contract.md`'s error shape, never a raw
  console error or blank screen.
- Empty states (no sessions yet, no check-ins yet) are designed, not left blank.
- Live camera capture (see §5) gets a deliberate, camera-app-like UI — this is the most visible
  "premium" moment in the whole app since it's the one place using the device hardware directly.

## 4. Non-negotiable rules (from the Rulebook — do not work around these)

1. **No gallery/file upload for check-in photos.** The check-in photo must come from
   `getUserMedia` (live camera stream) captured in-app, never `<input type="file">`. This is a
   direct anti-fraud requirement (Rulebook §7.2), not a style choice — do not add a "upload from
   gallery" fallback even if it seems more convenient for testing.
2. **Never let the client compute or submit `final_status`.** The client may show an optimistic
   "Submitting..." state, but the authoritative status always comes back from the API response.
3. **Location must come from the device's live GPS** (`navigator.geolocation.getCurrentPosition`),
   not a manually-entered lat/long field, for the member check-in flow.
4. **Show the user what's being collected and why** before requesting camera/GPS permission
   (Rulebook §11.1 — privacy notice before collection) — a short explanatory line before the
   permission prompt is enough for Phase 1.

## 5. Check-in flow UX (specific to this app)

1. Member opens an open session → sees venue name, time window, and a short "what happens on
   check-in" note.
2. Tap "Check In" → request GPS permission (if not granted) → request camera permission → open
   live camera view.
3. Capture photo → show a confirm/retake step (matches Working Book §5.2 "Live photo / OTP / QR").
4. Submit → loading state → success (shows returned `final_status`, distance) or a clear rejection
   reason from the API error shape.

## 6. API integration pattern

- One typed API client module in `src/lib/api.ts` wrapping `fetch`, attaching the JWT from storage
  automatically, and throwing a typed error matching `API_contract.md §1`.
- Types for requests/responses live alongside the API client and must mirror `API_contract.md`
  exactly — if the contract changes, update the types in the same PR, not later.
- No component calls `fetch` directly — always through the API client + a React Query hook.

## 7. Testing

- Vitest + React Testing Library for component tests.
- At minimum, cover: login form validation, check-in success path, check-in rejection (outside
  radius) rendering the correct error message, duplicate check-in message.

## 8. Git / commit conventions (frontend branch)

- Work happens on the `frontend` branch; PR into `main` when a feature is complete.
- Commit messages: `type(scope): summary`, e.g. `feat(checkin): add live camera capture flow`.
- Any PR that assumes a specific API response shape should link the relevant section of
  `API_contract.md`.

## 9. When you (the agent) notice something worth changing

Don't silently switch component libraries, state managers, or restructure folders because it seems
cleaner. Log the suggestion in `enhancements.md` with rationale and wait for developer approval,
unless it's a trivial non-behavioral fix.

## 10. Explicitly out of scope for Phase 1 frontend

No offline/PWA support, no left-venue/returned live monitoring UI, no activity submission screens,
no correction/dispute UI, no admin analytics charts beyond a simple check-in list. These are
Phase 2+ (see `PRD.md §3`, `development_roadmap.md`). Also: **no Flutter/mobile work** — Phase 1 is
web-only.
