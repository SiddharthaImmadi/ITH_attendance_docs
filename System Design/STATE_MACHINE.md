# State Machine

**Project:** ITH Attendance Management System

**Document Version:** 2.0

**Status:** Approved

---

# Purpose

This document defines the attendance state machine for volunteers participating in an event.

The state machine describes every valid attendance state, the events that trigger transitions between states, and the resulting outcomes.

This document is the authoritative reference for attendance state transitions and shall be followed by all implementations.

---

# Objectives

The state machine ensures:

- Predictable attendance behavior.
- Consistent business rule enforcement.
- Valid state transitions.
- Clear handling of exceptional situations.
- Alignment between backend, frontend, APIs, and reports.

---

# State Transition Overview

```

                 NOT_CHECKED_IN
                       │
                 Successful Check-in
                       │
                       ▼
                 INSIDE_VENUE
                 │          │
         Leave Venue     Check-out
                 │          │
                 ▼          ▼
            OUTSIDE_VENUE  CHECKED_OUT
                 │
        Outside > 10 Seconds
                 │
                 ▼
          PENDING_APPROVAL
          │              │
     Approve         Reject
          │              │
          ▼              ▼

   ┌──────────────┐   RESTRICTED
   │              │
PARTIAL_LEAVE  COMPLETE_LEAVE
   │              │
Return         Attendance Ends
   │              │
   ▼              ▼
INSIDE_VENUE   CHECKED_OUT

```

---

# Attendance States

## NOT_CHECKED_IN

### Description

The volunteer has not started attendance for the current event.

### Entry Conditions

- Event session opened.
- Volunteer eligible.

### Allowed Actions

- Check-in

### Exit Events

- Successful Check-in

---

## INSIDE_VENUE

### Description

The volunteer is actively participating within the approved venue.

### Entry Events

- Successful Check-in
- Return from Partial Leave

### Allowed Actions

- Continue volunteering
- Leave venue
- Check-out

### Exit Events

- Leave venue
- Manual Check-out
- Automatic Check-out

---

## OUTSIDE_VENUE

### Description

The volunteer has crossed the approved venue boundary.

The system begins monitoring the duration outside the venue.

### Entry Event

- Venue exit detected

### Allowed Actions

- Return immediately
- Remain outside

### Exit Events

- Return inside venue
- Outside duration exceeds 10 seconds

---

## PENDING_APPROVAL

### Description

The volunteer has submitted a leave request.

The request is awaiting administrator review.

### Entry Event

- Leave request submitted

### Allowed Actions

None

Volunteer awaits administrator decision.

### Exit Events

- Leave approved
- Leave rejected

---

## PARTIAL_LEAVE

### Description

Administrator approved temporary leave.

Volunteer is expected to return.

### Entry Event

- Partial Leave approved

### Allowed Actions

- Return to venue

### Exit Events

- Volunteer returns

---

## COMPLETE_LEAVE

### Description

Administrator approved permanent departure from the event.

Volunteer participation ends.

### Entry Event

- Complete Leave approved

### Allowed Actions

None

### Exit Events

- Attendance completion

---

## RESTRICTED

### Description

Administrator rejected the leave request and applied a temporary volunteering restriction.

### Entry Event

- Administrative restriction

### Allowed Actions

None

Restrictions remain until removed by an administrator.

---

## CHECKED_OUT

### Description

Attendance session has ended.

### Entry Events

- Manual Check-out
- Automatic Check-out
- Attendance completion following approved Complete Leave

### Allowed Actions

None

This is a terminal state.

---

# Valid Transitions

| Current State | Event | Next State |
|---------------|-------|------------|
| NOT_CHECKED_IN | Successful Check-in | INSIDE_VENUE |
| INSIDE_VENUE | Venue Exit | OUTSIDE_VENUE |
| OUTSIDE_VENUE | Return Before 10 Seconds | INSIDE_VENUE |
| OUTSIDE_VENUE | Outside > 10 Seconds | PENDING_APPROVAL |
| PENDING_APPROVAL | Partial Leave Approved | PARTIAL_LEAVE |
| PENDING_APPROVAL | Complete Leave Approved | COMPLETE_LEAVE |
| PENDING_APPROVAL | Leave Rejected | RESTRICTED |
| PARTIAL_LEAVE | Volunteer Returns | INSIDE_VENUE |
| INSIDE_VENUE | Manual Check-out | CHECKED_OUT |
| INSIDE_VENUE | Automatic Check-out | CHECKED_OUT |
| COMPLETE_LEAVE | Attendance Completion | CHECKED_OUT |

---

# Invalid Transitions

The following transitions shall not be permitted.

- CHECKED_OUT → INSIDE_VENUE
- CHECKED_OUT → OUTSIDE_VENUE
- COMPLETE_LEAVE → INSIDE_VENUE
- RESTRICTED → INSIDE_VENUE
- NOT_CHECKED_IN → CHECKED_OUT
- PARTIAL_LEAVE → CHECKED_OUT without attendance completion

Any attempt to perform an invalid transition shall be rejected.

---

# State Invariants

The following conditions shall always remain true.

## NOT_CHECKED_IN

- No attendance exists.

---

## INSIDE_VENUE

- Attendance is active.
- Presence monitoring is active.

---

## OUTSIDE_VENUE

- Attendance remains active.
- Outside timer is active.

---

## PENDING_APPROVAL

- Leave request exists.
- Administrator decision is pending.

---

## PARTIAL_LEAVE

- Attendance remains active.
- Volunteer is expected to return.

---

## COMPLETE_LEAVE

- Volunteer participation has ended.

---

## RESTRICTED

- Volunteer is temporarily restricted from future participation.
- Restriction remains until administrative removal.

---

## CHECKED_OUT

- Attendance is closed.
- Presence monitoring has stopped.
- No further attendance events shall be accepted.

---

# Related Documents

- ATTENDANCE_POLICY.md
- LEAVE_POLICY.md
- APPROVAL_WORKFLOW.md
- DATABASE_SCHEMA.md
- DATA_FLOW.md