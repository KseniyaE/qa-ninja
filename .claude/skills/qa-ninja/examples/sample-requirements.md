# Sample Requirements — Postcode Validation for Boiler Installation

> **Purpose:** Demonstration and evaluation input for QA NiNJA. This ticket deliberately contains gaps, ambiguities and a contradiction. The "Evaluator Notes" section at the end lists them so a QA Engineer can check QA NiNJA's output. **When running QA NiNJA on this file for evaluation, ask it to ignore the Evaluator Notes section.**

---

## Ticket

- **Ticket ID:** BOIL-1427
- **Title:** Validate customer postcode before booking a boiler installation
- **Product area:** E-commerce — Boiler Installation Quote & Booking
- **Market:** United Kingdom

## User Story

As a homeowner shopping for a new boiler online,
I want to enter my postcode at the start of the installation quote journey,
so that I know quickly whether installation is available in my area before I choose a boiler.

## Acceptance Criteria

**AC-1** — The postcode field is displayed on the "Check installation availability" step, before boiler selection.

**AC-2** — The user must enter a valid UK postcode.

**AC-3** — The postcode must be between 5 and 7 characters long.

**AC-4** — Postcodes are not case-sensitive (e.g. `sw1a 1aa` and `SW1A 1AA` are both accepted).

**AC-5** — If the postcode is in a serviceable area, the user proceeds to the boiler selection page.

**AC-6** — If the postcode is not in a serviceable area, an appropriate message is shown.

**AC-7** — Validation should be fast.

**AC-8** — The entered postcode is pre-filled on the checkout address form.

## Technical Context

- Postcode format validation happens on the frontend; serviceability is checked via the internal **Coverage Service** (`GET /coverage?postcode=`), which returns `serviceable: true | false`.
- The Coverage Service is maintained by another team and is also used by the **Boiler Servicing** booking flow.
- The checkout address form uses a third-party address lookup that accepts postcodes **up to 8 characters including the space**.
- Serviceable areas are configured by the Operations team and may change weekly.
- Analytics event `postcode_checked` is fired on submit.

## Known Missing Requirements (as acknowledged by the BA)

- Exact wording of the "not serviceable" message has not been agreed.
- Behavior when the Coverage Service is unavailable is not defined.
- It is not decided whether users in non-serviceable areas can register interest / leave contact details.

---

## Evaluator Notes (do not include in analysis)

Expected findings QA NiNJA should surface (non-exhaustive):

- **Contradiction:** AC-3 (5–7 characters) vs. Technical Context (address lookup accepts up to 8 characters including the space) — also unclear whether the space counts in AC-3.
- **Ambiguity:** "valid UK postcode" (AC-2) — format only, or must it exist? BFPO / Crown Dependencies (GY, JE, IM) / special postcodes?
- **Ambiguity:** "appropriate message" (AC-6), "fast" (AC-7) — no wording / no measurable threshold.
- **Ambiguity:** whitespace handling — leading/trailing spaces, missing internal space (`SW1A1AA`), multiple spaces.
- **Missing:** error message for invalid format; empty field behavior; Coverage Service timeout/failure behavior; whether the user can edit the postcode later; what happens if the checkout address postcode differs from the checked postcode (AC-8).
- **Missing:** which analytics properties are sent; whether invalid submissions fire the event.
- **What-if:** serviceable area changes between the availability check and checkout; user navigates back and changes postcode; double submit.
- **Regression:** Boiler Servicing flow (shared Coverage Service); checkout address lookup; analytics.
- **Expected readiness:** NOT READY or NEEDS CLARIFICATION (contradiction on a core length rule and undefined failure behavior).
