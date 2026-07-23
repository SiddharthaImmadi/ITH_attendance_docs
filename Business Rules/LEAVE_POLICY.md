# Leave Policy

**Project:** ITH Attendance Management System

**Document Version:** 2.0

**Status:** Approved

---

# Purpose

This document defines the business rules governing volunteer leave during an active attendance session.

A leave request is generated when a volunteer exits the approved event venue and does not immediately return.

This document specifies the permitted leave types, approval requirements, volunteer responsibilities, and attendance consequences.

Implementation details such as APIs, database structures, or application logic are intentionally excluded.

---

# Policy Objective

The leave policy balances operational flexibility with attendance accountability.

Volunteers may occasionally need to leave the venue for legitimate reasons. This policy ensures that such situations are documented, reviewed, and handled consistently.

---

# Leave Types

The system supports two leave types.

## Partial Leave

A temporary leave from the event venue for official event-related work.

Characteristics:

- Intended only for official work.
- Volunteer is expected to return.
- Requires administrator approval.
- Attendance session remains active while awaiting review.
- A RETURNED event shall be recorded if the volunteer comes back.

Examples include:

- Collecting event materials.
- Coordinating with another venue.
- Official organizer instructions.

Personal reasons shall not be submitted as Partial Leave.

---

## Complete Leave

A permanent exit from the current event.

Characteristics:

- Volunteer does not intend to return.
- Requires administrator approval.
- Ends participation in the current event.
- Volunteer shall not continue volunteering during the same event.

Examples include:

- Medical emergency.
- Personal emergency.
- Family emergency.
- Other approved circumstances.

---

# Leave Trigger

A leave request becomes necessary when:

- The volunteer exits the approved venue boundary.
- The volunteer remains outside for more than **10 seconds**.
- The volunteer does not immediately return.

The volunteer shall receive a notification requesting further action.

---

# Volunteer Responsibilities

The volunteer shall:

- Respond to the leave notification.
- Select the appropriate leave type.
- Select a predefined leave reason.
- Provide additional explanation where required.

Failure to respond may result in administrative review.

---

# Leave Request Requirements

Every leave request shall contain:

- Leave type.
- Selected reason.
- Additional explanation.
- Submission timestamp.

Once submitted, the request enters the approval workflow.

---

# Administrator Review

Every leave request requires administrator review.

The administrator may:

- Approve the request.
- Reject the request.

Administrative decisions shall be recorded.

---

# Approved Leave

If approved:

## Partial Leave

- Attendance remains valid.
- Volunteer may continue participating after returning.
- No volunteering restriction is applied.

---

## Complete Leave

- Attendance session is concluded according to the attendance policy.
- Volunteer leaves the current event without penalty.
- No volunteering restriction is applied.

---

# Rejected Leave

If a leave request is rejected:

The administrator determines that the provided justification does not satisfy the organization's attendance policy.

Consequences may include:

- Attendance review.
- Temporary volunteering restriction.
- Administrative follow-up.

Repeated misuse of the leave process may result in stricter administrative action.

---

# Volunteer Restrictions

Volunteer restrictions are governed by the Volunteering Policy.

Restrictions:

- Are temporary.
- Apply only after administrative review.
- May be removed by an administrator.

---

# Attendance Impact

Partial Leave

- Attendance continues.
- Presence monitoring continues.
- Volunteer is expected to return.

---

Complete Leave

- Attendance ends.
- Volunteer participation ends.
- Presence monitoring stops after attendance completion.

---

# Audit Requirements

Every leave request shall maintain:

- Submission history.
- Administrator decision.
- Decision timestamp.
- Decision history.

These records support reporting and future administrative review.

---

# Policy Exceptions

Only an administrator may approve exceptions to this policy.

All exceptions shall be documented.

---

# Compliance

All volunteers participating in an event are subject to this policy.

Failure to comply may affect future volunteering opportunities.

---

# Related Documents

- ATTENDANCE_POLICY.md
- APPROVAL_WORKFLOW.md
- VOLUNTEERING_POLICY.md
- BUSINESS_RULES.md