# QA NiNJA — Report Format

QA NiNJA produces one report format: the **Quick Report**. Render it with [../templates/qa-quick-template.md](../templates/qa-quick-template.md).

**MVP First — find what can block delivery; test what matters for release.** Analyse with [analysis-rules.md](analysis-rules.md) §0. Priorities follow [classification-rules.md](classification-rules.md) §3. Do not show internal reasoning, methodology or exhaustive inventories.

---

## Structure

Exactly three parts, in this order: a readiness line, a Questions table, a QA Execution Checklist. No other sections: no scope, findings, scenarios, next actions, recommendations or legend.

### Readiness line

- Title `QA Quick Report — <Ticket title>`, then one line: readiness status, blocker reference, Ticket ID (or `Not provided`), date, source.
- Readiness is assigned per classification-rules §4 (READY / NEEDS CLARIFICATION / NOT READY). An open P0 never results in READY. Add a short explanation (one line, citing question IDs) under the status line when the status is not READY.
- Blocker: the question IDs of open P0 questions (e.g. `Blocker: Q1`), or `None`.
- Intake mode: add one line `Intake: final validation <PASSED | PASSED WITH LIMITATIONS> · cleaned copy SHA-1 <sha> · procedure SHA-1 <sha> · V-GATE passed`.
- Assumptions, if any: one extra line, each labelled `Assumption:`.

### Questions table

Exactly these four columns: `Q | P | Owner | Question (why)`.

- IDs `Q1`, `Q2`, `Q3`… (stable within the report). Order by priority: P0, then P1, then P2.
- **P:** P0, P1 or P2, per classification-rules §3. **Owner:** BA, Developer or BA + Developer, per classification-rules §5.
- **Question (why):** one question about **one** specific missing point, in one short, direct sentence. The question itself names the missing information. No background, no analysis, no reason in brackets, no list of speculative alternatives unless essential.
  - Bad: "What should happen when a user enters an invalid value, because the ticket does not specify whether the input should be blocked, an error should be displayed, or the user should be prevented from proceeding?"
  - Good: "What should happen when the user enters an invalid value?"
- No duplicate questions. Two different missing points are two questions; one underlying uncertainty is one question.
- Ask only questions that pass the question gate (analysis-rules §0): needed for an explicitly stated requirement or AC, behaviour genuinely undefined, answer would change an expected outcome. No questions about scenarios outside the stated ACs or about QA execution dependencies (access, accounts, test data, environment).
- No questions → write one line `No open questions.` instead of the table.

### QA Execution Checklist

Exactly these four columns, in this order: `AC N | Check | Screenshot | Result`. Never add other columns (Do, Expect, Wait, Priority, Owner, Status, Expected Result).

- **AC N:** the original AC ID from the ticket (`AC1`, `AC2`…). Never invent one. A check that maps to no AC (e.g. essential regression, a rule stated only in a comment or rules table) uses `General`. One check that verifies several ACs in the same action may list them all (`AC1, AC2`).
- **Every AC has at least one check.** If an AC has both confirmed and unresolved behaviour, test the confirmed part. If the unresolved part prevents meaningful validation of the AC, add a blocked check referencing the relevant question. Never invent the unresolved expected result, and never raise a question's priority just to satisfy coverage.
- **Never assume an unresolved precondition.** A row never states a pass condition that depends on an open question of any priority (P0, P1 or P2). Limit the executable row to the conditions the ticket confirms (e.g. "on pages that have the existing FAQ"), and represent the unresolved part as a blocked row referencing that question (e.g. "on a page without the existing FAQ → TBD — Wait: Q4"). Keep the question's original priority.
- **Check:** `action → expected result`, short. The expected result must be supported by the input (analysis-rules §9); the AC N cell is its evidence. A `General` regression row may expect "still present and unchanged", but never describes existing behaviour the input does not state.
  - Unknown outcome → `action → TBD — Wait: Qn`.
  - Partly known → `action → <confirmed part>; <unresolved part> TBD — Wait: Qn`.
  - Several blocking questions → `TBD — Wait: Q1, Q2`.
  - **Exact syntax:** `TBD — Wait: ` with an em dash (—), then the question IDs separated by `, `. Never parentheses, a hyphen or another dash, or any other form (e.g. not `TBD (Wait: Q1)`, `TBD - Wait: Q1`, `Wait Q1`).
  - Do not repeat the question's explanation; the `Wait:` reference is enough. A blocked row never states a pass condition for the unresolved part.
- **Screenshot** and **Result:** always empty. Reserved for the tester.
- Keep similar scenarios in one row. Split only when they need meaningfully different test conditions or expected results. Typically 1–3 rows per AC. No speculative or duplicate rows.
- A P2 question does not by itself require a checklist row. It needs a blocked row only when its unresolved point would otherwise be assumed by a check, or prevents meaningful validation of an AC (see above). Explicitly out-of-scope behaviour is never a checklist row (analysis-rules §0).

#### Row order (two-level sort)

1. **Primary — execution status.** A row is **blocked** if its Check contains `Wait:` (one or several questions, fully or partly unresolved). Every other row is **executable**. All executable rows come before all blocked rows.
2. **Secondary — AC category, inside each execution group.** Rows with specific AC references (`AC1`, `AC2`, `AC1, AC2`) first, `General` rows last **within that same group**.

The result is always exactly this sequence (empty blocks are skipped):

| Block | Execution group | AC category |
|-------|-----------------|-------------|
| 1 | Executable | Specific AC |
| 2 | Executable | `General` |
| 3 | Blocked | Specific AC |
| 4 | Blocked | `General` |

- `General` is **not** a block at the end of the whole checklist. An executable `General` row always comes before the first blocked row.
- `General` never moves a row between groups; only `Wait:` decides the group.
- Within a block, keep the rows' relative order (normally AC order). Priority never changes the order.

---

## Pre-save validation (before showing or saving)

Run both checks silently on the finished report. They check the report only; never change the analysis to pass them.

**A. Coverage, integrity and questions** — fix the report if any item fails:

1. Every AC in the ticket has at least one checklist row; every row maps to an AC or `General`.
2. Every P0/P1 finding from the analysis is represented by at least one question or checklist row.
3. No expected result is invented; confirmed and unresolved parts are clearly separated. No row states a pass condition that depends on an open question of any priority.
4. Every row containing `TBD` contains `Wait:` with at least one existing question ID, and every unresolved part of a row has one.
5. No explicitly out-of-scope behaviour is a checklist row or question; no question, row, assumption or other line mentions an unreadable image or attachment; a P2 question has a row only when rule "Never assume an unresolved precondition" or AC coverage requires it.
6. Every question covers one missing point in one short sentence, has a priority and an owner, follows classification-rules §5, and is not a duplicate. Questions are ordered P0, P1, P2. Every question passes the question gate (analysis-rules §0); none is about a scenario outside the stated ACs or a QA execution dependency, and no check is blocked only because QA access is undocumented.
7. Priorities, owners, blocker reference and readiness match the analysis.

**B. Checklist structure and order** — walk the rows top to bottom and label each one `executable`/`blocked` and `specific`/`General`, then confirm:

1. The checklist has exactly four columns: `AC N | Check | Screenshot | Result`; Screenshot and Result cells are empty.
2. Every blocked row (including partly blocked rows) contains the exact text `TBD — Wait: Q1` or `TBD — Wait: Q1, Q2` (em dash, no parentheses), and every referenced question exists in the Questions table. Search every row for `Wait` and `TBD`: any other form (e.g. `TBD (Wait: Q1)`, `TBD - Wait:`) fails — rewrite it in the exact syntax before showing or saving.
3. No executable row appears after the first blocked row.
4. Executable `General` rows come after the executable specific-AC rows and before every blocked row.
5. Blocked `General` rows come after every blocked specific-AC row.

If B fails: reorder the rows per **Row order**, then run B again. If it still fails, stop — do not show or save the report as valid; report the failure instead.

---

## Conciseness Rules

1. Prioritise by release impact: P0 first, then P1, then P2.
2. Short sentences, plain language for QA, BA and Developer.
3. Explain each issue once, in the Questions table; the checklist only references it.
4. No exhaustive inventories, low-value What-ifs, internal reasoning or methodology.
5. Narrow tables with only the columns defined above; readable on a laptop screen.

All item counts are defaults, not caps. Never drop a critical question or check to shorten the report.

## ID Conventions

| Prefix | Use |
|--------|-----|
| `Q` | Questions table, `Q1`, `Q2`… |
