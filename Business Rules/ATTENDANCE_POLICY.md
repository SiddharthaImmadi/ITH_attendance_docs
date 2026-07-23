# Attendance Policy

**Project:** ITH Attendance Management System

**Document Version:** 2.0

**Status:** Approved

---

# Purpose

This document defines the attendance lifecycle for volunteers participating in an event.

It establishes the business rules governing attendance from check-in until the attendance session is completed.

Implementation details such as APIs, databases, or application logic are intentionally excluded.

---

# Attendance Lifecycle

A volunteer attendance session consists of the following stages:

1. Authentication
2. Check-in
3. Active Presence Monitoring
4. Leave Detection (if applicable)
5. Return (if applicable)
6. Check-out
7. Attendance Completion

Attendance officially begins only after a successful check-in.

Attendance officially ends only after a successful or automatic check-out.

---

# Eligibility for Attendance

A volunteer is eligible to attend only when:

- The volunteer has a valid account.
- The volunteer is authenticated.
- The volunteer is not restricted from volunteering.
- The event session is open.
- The volunteer has not already checked in.

If any of these conditions are not satisfied, attendance shall not begin.

---

# Check-in Policy

A volunteer may check in only once per attendance session.

Successful check-in confirms:

- Identity verification.
- Location verification.
- Attendance registration.

Once check-in is complete:

- Attendance becomes active.
- Presence monitoring begins.

---

# Active Attendance

A volunteer is considered actively attending while:

- The attendance session remains open.
- The volunteer has not checked out.
- The volunteer has not completed a full leave from the event.

During active attendance:

- Presence monitoring remains enabled.
- Attendance events continue to be recorded.

---

# Presence Monitoring Policy

Presence monitoring begins immediately after check-in.

Monitoring continues until:

- Manual check-out, or
- Automatic check-out.

The objective of monitoring is to determine whether the volunteer remains inside the approved venue boundary.

The system records significant presence events instead of storing continuous movement history.

Recorded events include:

- CHECK_IN
- LEFT
- RETURNED
- CHECK_OUT

---

# Venue Exit Policy

A volunteer leaving the approved venue boundary shall be monitored.

If the volunteer remains outside the approved boundary for more than **10 seconds**:

- The volunteer is considered to have left the venue.
- The volunteer shall receive an alert.
- The volunteer shall either:
  - Return to the venue, or
  - Submit a leave request.

---

# Return Policy

If a volunteer returns to the approved venue before completing a leave request:

- Attendance continues.
- A RETURNED event is recorded.
- Presence monitoring continues.

---

# Check-out Policy

Attendance may end in one of two ways.

## Manual Check-out

The volunteer voluntarily ends attendance.

---

## Automatic Check-out

If a volunteer does not manually check out:

- Attendance remains active until the event ends.
- A configurable grace period is applied.
- At the end of the grace period:
  - Attendance is automatically closed.
  - Presence monitoring stops.
  - No additional attendance events are accepted.

---

# Attendance Completion

An attendance session is considered complete only when:

- A check-out event exists.
- Presence monitoring has stopped.
- The attendance timeline has been finalized.

---

# Attendance Records

Every attendance session shall maintain a chronological record of significant events.

Attendance records shall support:

- Administrative review.
- Attendance reporting.
- Audit requirements.

---

# Policy Exceptions

Attendance exceptions are governed by:

- Leave Policy
- Approval Workflow
- Volunteering Policy

These policies define how attendance may continue or end under exceptional circumstances.

---

# Compliance

Every attendance session shall comply with this policy.

Any deviation requires administrative approval and must be recorded.

---

# Related Documents

- RULEBOOK.md
- LEAVE_POLICY.md
- APPROVAL_WORKFLOW.md