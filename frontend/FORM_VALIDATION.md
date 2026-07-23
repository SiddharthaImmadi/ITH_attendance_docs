
# Form Validation

**Project:** ITH Attendance Management System

**Document Version:** 2.0

**Status:** Approved

---

# Purpose

This document defines frontend validation principles for user input.

Frontend validation improves usability but does not replace backend validation.

---

# Validation Principles

The frontend shall:

- Validate obvious input errors before submission.
- Display clear validation messages.
- Prevent submission of incomplete forms where possible.

Backend validation remains authoritative.

---

# Login Validation

Typical validation includes:

- Required email
- Required password
- Email format

---

# Check-in Validation

Typical validation includes:

- Event selected
- Required verification data available

---

# Leave Request Validation

Typical validation includes:

- Leave type selected
- Leave reason selected
- Required explanation provided when applicable

---

# Administrative Forms

Administrative forms shall validate required information before submission.

---

# Validation Feedback

Validation messages should:

- Clearly identify the problem.
- Explain what must be corrected.
- Appear near the affected input where practical.

---

# Related Documents

- System Design/VALIDATION_RULES.md
- ERROR_HANDLING.md