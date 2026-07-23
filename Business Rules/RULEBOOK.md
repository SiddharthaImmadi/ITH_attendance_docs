# Rulebook

**Project:** ITH Attendance Management System

**Version:** 2.0

**Status:** Approved

---

# Purpose

This document defines the governing rules of the attendance system.

These rules describe how attendance should be managed from a business perspective.

Technical implementation details are intentionally excluded.

Whenever there is a conflict between implementation and this document, this document takes precedence until formally updated.

---

# Attendance Philosophy

Attendance is more than recording arrival and departure.

The objective is to ensure that volunteers actively participate in the event while allowing legitimate operational movement under administrator supervision.

Every attendance decision must be:

- Fair
- Transparent
- Traceable
- Reviewable

---

# Core Principles

## Principle 1 — Identity

Only authenticated volunteers may participate.

Attendance cannot exist without authentication.

---

## Principle 2 — Presence

Attendance begins only after successful check-in.

Remaining inside the approved venue is considered active participation.

---

## Principle 3 — Accountability

Every important attendance action shall be recorded.

Examples include:

- Check-in
- Leaving the venue
- Returning
- Leave request
- Approval
- Rejection
- Check-out

---

## Principle 4 — Administrative Authority

Administrators have final authority regarding attendance decisions.

Administrative decisions override volunteer requests where applicable.

---

## Principle 5 — Transparency

Attendance decisions shall never occur silently.

Every significant decision shall be visible through reports or attendance history.

---

## Principle 6 — Fairness

Volunteers leaving for legitimate reasons shall not be penalized.

Repeated misuse of the attendance system may result in administrative restrictions.

---

## Principle 7 — Documentation First

Business rules shall be documented before implementation.

Undocumented behavior shall not be introduced into the system.

---

# Rule Hierarchy

Business Rules

↓

System Design

↓

API Contracts

↓

Implementation

↓

Testing

If implementation conflicts with documented business rules, the implementation shall be corrected.

---

# Governance

Changes to this rulebook require:

1. Documentation review.
2. Approval.
3. Version update.
4. Implementation.

---

# Related Documents

- ATTENDANCE_POLICY.md
- LEAVE_POLICY.md
- APPROVAL_WORKFLOW.md
- BUSINESS_RULES.md