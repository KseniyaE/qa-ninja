# QA NiNJA — Analysis Rules

These rules define how QA NiNJA analyzes requirements. Apply them to the supplied input only. Every finding must point back to specific text in the input (or to its absence). These rules govern the complete internal analysis; [report-format.md](report-format.md) decides what the report shows.

---

## 0. MVP Focus

**MVP First — find what can block delivery; test what matters for release.** Think about release risk, not theoretical completeness.

- Identify the feature purpose, the core user journey and the MVP scope (in this release / explicitly out of scope).
- Apply §1–§8 to the core journey and MVP scope only. Do not build a complete requirement inventory or exhaustive lists.
- Classify each issue with the MVP priority in [classification-rules.md](classification-rules.md) §3. Assess real business and release impact (likelihood and severity), not only whether something is missing.
- P0, P1 and P2 questions are reported in the Questions table. P2 questions do not by themselves create checklist rows, but no check may assume an unresolved point of any priority is true (report-format.md).
- Do not treat every ambiguity as a blocker. Do not create speculative risks. Never downgrade a genuinely critical risk to shorten the report.
- **Question gate — stay within the ticket's scope.** §1–§8 find candidate gaps; ask a clarification question only if all of these hold: (1) the information is needed to understand or validate an explicitly stated requirement or AC; (2) the expected product behaviour is genuinely undefined in the ticket; (3) the answer would change an expected outcome or the acceptance decision. One question per underlying uncertainty. Do not ask about scenarios the ACs do not cover merely because they are common (e.g. failed or cancelled payment when only successful payment is specified; validation rules, file types or size limits, or extra roles and permissions the ticket does not mention). A possible additional scenario or future enhancement is not a requirement gap.
- **QA execution dependencies are not requirement questions** (classification-rules §3): how to access an admin portal or test inbox, which account, test data or environment to use.
- Behaviour the input explicitly marks as out of scope (e.g. in a comment) is never a mandatory checklist check, test value or clarification question, and its absence is never a blocker. Mention it only when needed to explain a scope boundary.
- Unreadable images and attachments: analyse only the readable ticket text. Never infer, describe or assume their contents, and do not mention them in the report — no question, check, assumption or other line about them.

## 1. Requirement Completeness

A requirement is complete when a tester can determine, without guessing:

- **Trigger** — what user action or system event starts the behavior.
- **Preconditions** — required state, data, roles or configuration.
- **Input** — accepted values, formats and sources.
- **Processing rule** — the business logic applied.
- **Expected result** — observable outcome (UI, data, message, state change).
- **Failure behavior** — what happens on invalid input or dependency failure.

For each requirement, check every dimension. A missing dimension is a **missing requirement** only if it affects testability or user/business outcome for this feature.

For every missing requirement you report (P0/P1), state:

- What is missing.
- **Why it matters** (risk to users, business, data, or testability).
- Who should answer (BA or Developer).

Do not list missing items that are irrelevant to the feature (e.g. "localization" for an internal admin-only API with no UI text), unless the input suggests they apply.

## 2. Missing Acceptance Criteria

Flag missing acceptance criteria when:

- A user story exists but has no acceptance criteria.
- An acceptance criterion describes intent but no verifiable outcome ("the system should handle errors gracefully").
- A behavior mentioned in the story or technical context has no corresponding criterion.
- Only the happy path is covered, and the ticket itself states or requires an unhappy path (e.g. a mandatory step, a named error or failure) without defining it.
- Error messages, states or UI feedback are referenced but not specified.

Good acceptance criteria are observable, verifiable and unambiguous. When proposing a missing criterion, phrase it as a **question**, not as a decided rule.

## 3. Ambiguity Detection

A statement is ambiguous when it has more than one reasonable interpretation that would lead to different tests. Typical signals:

- Vague qualifiers: *fast, quickly, user-friendly, appropriate, valid, correct, gracefully, etc., some, usually*.
- Undefined terms or domain words used without definition (e.g. "eligible area", "valid postcode").
- Unspecified quantities, units, time zones, formats or limits.
- Unclear actors ("the system", "they", "the user" when several roles exist).
- Pronouns or references with unclear targets.
- "and/or", optional steps, or conditions without an explicit else branch.

For each ambiguity: quote the text, list the competing interpretations, and explain the testing impact.

## 4. Contradiction Detection

A contradiction exists only when two statements in the input **cannot both be true** at the same time.

- Contradiction: "Postcode must be 5–7 characters" and "Accept postcodes up to 8 characters including the space".
- **Not** a contradiction: one statement is silent on something the other mentions — that is a missing or ambiguous requirement.
- **Not** a contradiction: the input conflicts with your general domain knowledge — record this as a question/assumption instead.

For each contradiction, quote both statements with their source (AC number, section) and explain the conflict.

## 5. What-if Analysis

Keep only What-ifs with a credible risk, prioritised per classification-rules §3; omit speculative ones. Generate What-if questions that expose unspecified behavior **specific to this feature**. Derive them from:

- State changes (session expiry, back navigation, refresh, concurrent edits).
- Dependency failures named in the technical context (API timeout, service unavailable, stale cache).
- Data variations implied by the domain (formats, casing, whitespace, special characters).
- Business rule edges (eligibility changes, boundary dates, configuration changes).
- User behavior (double submit, editing after validation, switching options).

Each What-if must state why it is relevant. Skip What-ifs whose answer is already explicit in the input.

## 6. Negative Scenarios

Negative scenarios verify that the system rejects or safely handles invalid input and failure conditions. Derive them from:

- Each validation rule (one negative per rule violation, not per random bad value).
- Required fields left empty.
- Wrong format, wrong type, out-of-range values.
- Unauthorized or wrong-role access, if roles are in scope.
- Dependency failure modes named in the input.
- Security-relevant input only when the feature accepts free text that is stored, displayed or sent to backend (e.g. script/markup injection).

Do not assume invalid input is rejected or blocked unless the input says so. If the expected rejection behavior (message, state) is not specified, apply §9.

## 7. Boundary-Value Analysis

Apply boundary-value analysis wherever the input defines (or implies) a limit:

- Lengths, counts, numeric ranges, dates/times, monetary amounts.
- Test at: **min−1, min, max, max+1**, and one representative valid value.
- For formats, test the edges of each pattern segment.
- If a limit is implied but not stated, raise it as a missing requirement and still list the boundary scenario, applying §9 to its Expected Result.
- If the input defines no meaningful limits, do not create boundary scenarios.
- Report only business-critical boundaries (money, eligibility, access, data integrity) at P0/P1.

Do not invent limits. Use only limits stated in the input, or mark them as unknown.

## 8. Regression Recommendations

Recommend regression checks only for areas the input shows (or strongly implies) are affected, and report only those at P0/P1:

- Shared components or services named in the technical context.
- Upstream/downstream flows (e.g. checkout, quote, booking) that consume this feature's output.
- Existing validations or data that could change behavior.
- Integrations, analytics or notifications that depend on this feature.

For each recommendation, state the affected area, why it is at risk, and the priority. Do not recommend "full regression" without justification.

## 9. Expected Result Evidence

Applies to every checklist check.

1. Every expected result must be supported by an explicit requirement or confirmed system behavior in the input. The AC N cell is its evidence.
2. Never invent frontend, backend, API, database, checkout, analytics or integration behavior.
3. Never infer that data is blocked, rejected, transformed, stored, transmitted or not transmitted unless the input explicitly confirms it.
4. An outcome is confirmed only under the conditions the input states. If it depends on a precondition the input does not establish, that part is unresolved.

Handling unknown outcomes:

| Situation | Check | Questions |
|-----------|-------|-----------|
| Fully specified | `action → confirmed expected result` | — |
| Partly specified | `action → <confirmed part>; <unresolved part> TBD — Wait: Qn` | Add the unresolved point as a question |
| Not specified | `action → TBD — Wait: Qn` | Add the unresolved point as a question |

A blocked check may still be run as an exploratory test, but it never states a pass condition for the unresolved part.

---

## Anti-patterns (avoid)

- Generic checklists unrelated to the feature (e.g. browser compatibility for a backend rule) without evidence.
- Multiple scenarios that test the same rule with equivalent data.
- Expected results phrased as facts when they are guesses.
- Expected results describing system behavior (rejection, blocking, storage, transmission, transformation) that the input does not confirm.
- Labelling a gap as a contradiction.
- Exhaustive inventories, speculative risks, or treating every ambiguity as a release blocker.
- Declaring requirements "complete" or READY without checking all dimensions.
