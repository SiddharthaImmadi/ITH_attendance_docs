
# Approval Workflow

**Project:** ITH Attendance Management System

**Document Version:** 2.0

**Status:** Approved

---

# Purpose

This document defines the business workflow for reviewing and deciding leave requests submitted by volunteers during an active attendance session.

The workflow establishes the responsibilities of volunteers and administrators while ensuring that attendance decisions remain transparent, consistent, and reviewable.

Implementation details such as APIs, database structures, notifications, or application logic are intentionally excluded.

---

# Workflow Objective

The approval workflow exists to:

- Ensure every leave request receives administrative review.
- Distinguish legitimate leave from attendance misuse.
- Maintain fairness across all volunteers.
- Preserve an auditable history of every decision.

---

# Workflow Participants

## Volunteer

Responsible for:

- Responding to leave notifications.
- Selecting the appropriate leave type.
- Selecting an appropriate leave reason.
- Providing additional explanation where required.

---

## Administrator

Responsible for:

- Reviewing submitted leave requests.
- Evaluating supporting information.
- Approving or rejecting requests.
- Applying administrative actions where necessary.

Administrative decisions are considered final unless organizational policy specifies otherwise.

---

# Workflow Trigger

The approval workflow begins when:

- A volunteer exits the approved venue boundary.
- The volunteer remains outside the venue for more than **10 seconds**.
- The volunteer submits a leave request.

Once submitted, the leave request enters the **Pending Approval** state.

---

# Pending Approval

Pending Approval is a temporary review state.

While a request is pending:

- The request awaits administrator review.
- No approval or rejection has yet been determined.
- The request remains part of the volunteer's attendance history.
- The final attendance outcome depends on the administrator's decision.

---

# Administrator Review Process

The administrator reviews:

- Leave type.
- Selected reason.
- Additional explanation.
- Attendance history, where appropriate.
- Previous leave history, where appropriate.

The administrator may consider any additional organizational context before making a decision.

---

# Approval Decision

The administrator shall make one of the following decisions.

## Approve

The leave request satisfies organizational attendance requirements.

The request proceeds according to its leave type.

### Approved Partial Leave

- Attendance remains valid.
- Volunteer may return and continue participating.
- No volunteering restriction is applied.

### Approved Complete Leave

- Participation in the current event ends.
- Attendance concludes according to the Attendance Policy.
- No volunteering restriction is applied.

---

## Reject

The administrator determines that the leave request does not satisfy organizational attendance requirements.

A rejected request may result in one or more administrative actions in accordance with organizational policy.

Possible actions include:

- Attendance review.
- Temporary volunteering restriction.
- Administrative follow-up.

---

# Decision Recording

Every administrator decision shall record:

- Decision type.
- Decision timestamp.
- Reviewing administrator.
- Administrative remarks, where applicable.

These records form part of the permanent attendance history.

---

# Notification Principles

The volunteer shall be informed of the administrator's decision.

The notification shall clearly communicate:

- The decision.
- Any resulting attendance outcome.
- Any administrative action affecting future participation.

---

# Fair Review Principles

Every review shall be:

- Consistent.
- Evidence-based.
- Transparent.
- Documented.
- Free from arbitrary decision-making.

---

# Exceptional Circumstances

An administrator may approve exceptions where justified by organizational policy.

Every exception shall be documented.

---

# Compliance

All leave requests submitted during an attendance session shall follow this workflow.

Administrative decisions shall be recorded for audit and reporting purposes.

---

# Related Documents

- RULEBOOK.md
- ATTENDANCE_POLICY.md
- LEAVE_POLICY.md
- VOLUNTEERING_POLICY.md
- BUSINESS_RULES.md