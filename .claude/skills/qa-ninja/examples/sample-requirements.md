# Sample Requirements — Serial Number Validation for Appliance Registration

> **Purpose:** Demonstration and evaluation input for QA NiNJA. This ticket deliberately contains gaps, ambiguities and a contradiction. The "Evaluator Notes" section at the end lists them so a QA Engineer can check QA NiNJA's output. **When running QA NiNJA on this file for evaluation, ask it to ignore the Evaluator Notes section.**

---

## Ticket

- **Ticket ID:** APP-5042
- **Title:** Validate appliance serial number before warranty registration
- **Product area:** E-commerce — Appliance Warranty Registration
- **Market:** United Kingdom

## User Story

As a customer purchasing a new household appliance online,
I want to enter my appliance serial number at the start of the warranty registration process,
so that I know quickly whether the appliance is eligible for coverage before I complete the registration.

## Acceptance Criteria

**AC-1** — The serial number field is displayed on the "Check appliance eligibility" step, before warranty plan selection.

**AC-2** — The user must enter a valid serial number.

**AC-3** — The serial number must be between 8 and 20 characters long.

**AC-4** — Serial numbers are not case-sensitive (e.g. `xyz123abc` and `XYZ123ABC` are both accepted).

**AC-5** — If the serial number is in an eligible product line, the user proceeds to the warranty plan selection page.

**AC-6** — If the serial number is not eligible, an appropriate message is shown.

**AC-7** — Validation should be fast.

**AC-8** — The entered serial number is pre-filled on the warranty confirmation form.

## Technical Context

- Serial number format validation happens on the frontend; eligibility is checked via the internal **Product Registry Service** (`GET /registry?serial=`), which returns `eligible: true | false`.
- The Product Registry Service is maintained by another team and is also used by the **Appliance Support** booking flow.
- The warranty confirmation form uses a third-party form that accepts serial numbers **up to 25 characters including hyphens**.
- Eligible product lines are configured by the Product team and may change quarterly.
- Analytics event `serial_checked` is fired on submit.

## Known Missing Requirements (as acknowledged by the BA)

- Exact wording of the "not eligible" message has not been agreed.
- Behavior when the Product Registry Service is unavailable is not defined.
- It is not decided whether users with ineligible serial numbers can leave contact details for future eligibility.

---

## Evaluator Notes (do not include in analysis)

Expected findings QA NiNJA should surface (non-exhaustive):

- **Contradiction:** AC-3 (8–20 characters) vs. Technical Context (form accepts up to 25 characters including hyphens) — also unclear whether hyphens count in AC-3.
- **Ambiguity:** "valid serial number" (AC-2) — format only, or must it exist? Are special characters allowed? Regional variants?
- **Ambiguity:** "appropriate message" (AC-6), "fast" (AC-7) — no wording / no measurable threshold.
- **Ambiguity:** whitespace and hyphen handling — leading/trailing spaces, internal hyphens, multiple spaces.
- **Missing:** error message for invalid format; empty field behavior; Product Registry Service timeout/failure behavior; whether the user can edit the serial number later; what happens if the warranty confirmation serial number differs from the checked serial number (AC-8).
- **Missing:** which analytics properties are sent; whether invalid submissions fire the event.
- **What-if:** eligible product lines change between the eligibility check and warranty confirmation; user navigates back and changes serial number; double submit.
- **Regression:** Appliance Support flow (shared Product Registry Service); warranty confirmation form; analytics.
- **Expected readiness:** NOT READY or NEEDS CLARIFICATION (contradiction on a core length rule and undefined failure behavior).
