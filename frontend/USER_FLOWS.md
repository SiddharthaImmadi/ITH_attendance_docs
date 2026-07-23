
# User Flows

**Project:** ITH Attendance Management System

**Document Version:** 2.0

**Status:** Approved

---

# Purpose

This document defines the major user journeys through the Attendance Management System.

---

# Login Flow

```
Open Application

↓

Enter Credentials

↓

Authentication

↓

Dashboard
```

---

# Attendance Flow

```
Dashboard

↓

Select Event

↓

Check-in

↓

Attendance Active

↓

Check-out

↓

Attendance Complete
```

---

# Presence Monitoring Flow

```
Attendance Active

↓

Location Updates

↓

Inside Venue
        │
        └── Continue Monitoring

Outside Venue

↓

10-Second Threshold

↓

Leave Alert

↓

Submit Leave Request

↓

Pending Approval
```

---

# Partial Leave Flow

```
Pending Approval

↓

Approved

↓

Partial Leave

↓

Return to Venue

↓

Attendance Continues
```

---

# Complete Leave Flow

```
Pending Approval

↓

Approved

↓

Complete Leave

↓

Attendance Ends
```

---

# Administrative Approval Flow

```
Pending Requests

↓

Review Request

↓

Approve
    │
    ├── Partial Leave
    └── Complete Leave

or

Reject

↓

Restriction (if applicable)
```

---

# Reporting Flow

```
Reports

↓

Select Filters

↓

Retrieve Report

↓

Display Results

↓

Export (Optional)
```

---

# Related Documents

- Business Rules/APPROVAL_WORKFLOW.md
- System Design/STATE_MACHINE.md