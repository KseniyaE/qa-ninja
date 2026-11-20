# QA NiNJA

**AI-powered Test Readiness Assistant** — *Think before you test.*

**Version:** QA NiNJA v0.5.3.2

## Project Overview

QA NiNJA is a Claude Code Skill for QA Engineers. It reads a ticket (user story and Acceptance Criteria) **before testing starts** and tells you whether the ticket is ready to test, what is missing, and what you can test right now.

**The problem it solves.** Testing often starts on tickets with unclear or incomplete requirements. Gaps are found late, during execution, and testers either guess the expected behaviour or wait. QA NiNJA moves that work to the start: it separates what the ticket confirms from what it leaves open, asks the BA or Developer short, specific questions, and gives the tester a checklist they can execute immediately.

**Intended user.** A QA Engineer (or anyone preparing to test a Jira-style ticket) working in Claude Code.

**Why it helps in everyday QA work.**
- A quick readiness decision instead of reading the ticket several times.
- Clarification questions are written once, prioritised and routed to the right person.
- The checklist is ready to execute and to fill in with screenshots and results.
- It never invents expected behaviour, so a passed check really means the ticket's requirement was met.

## Main Capabilities

- **Ticket analysis from manual input** — a ticket file, pasted text, or a ticket export dropped into `input/tickets/` (intake mode).
- **Requirements and Acceptance Criteria analysis** — missing, ambiguous and contradictory requirements, business-critical boundaries and essential regression, limited to the ticket's stated scope.
- **Readiness classification** — `READY`, `NEEDS CLARIFICATION` or `NOT READY`, with the blocking question named.
- **Clarification questions** — one short question per missing point, with a priority (P0 critical blocker, P1 important, P2 secondary) and an owner (BA for business questions, Developer for technical questions).
- **QA execution checklist** — at least one check per Acceptance Criterion, in four columns: `AC N | Check | Screenshot | Result`.
- **Confirmed vs unresolved behaviour** — checks with a confirmed expected result are executable; unresolved parts are marked `TBD — Wait: Qn` and point to the question that must be answered first.
- **No invented behaviour** — expected results come only from the ticket; out-of-scope items are not added as mandatory checks; QA environment access (test accounts, admin portal access) is treated as test preparation, not as a requirement gap.
- **Intake mode** — extracts and cleans a ticket export (`.md`, `.txt`, `.doc`, `.docx`), validates that the cleaned copy preserves the original requirements, and only then analyses it.

## Technical Structure

```
qa-ninja-homework4/
├── .claude/
│   └── skills/
│       └── qa-ninja/
│           ├── SKILL.md                      # Skill entry point and workflow (/qa-ninja)
│           ├── references/
│           │   ├── analysis-rules.md         # How gaps, ambiguity, contradictions and edge cases are found
│           │   ├── classification-rules.md   # Requirement status, priorities, readiness, question ownership
│           │   ├── report-format.md          # Quick Report structure and pre-save validation
│           │   └── ticket-intake.md          # Intake mode: extract, clean and validate a ticket export
│           ├── templates/
│           │   └── qa-quick-template.md      # Quick Report template
│           └── examples/
│               └── sample-requirements.md    # Sample ticket with evaluator notes
├── demo/
│   └── tickets/                              # Synthetic demo tickets (fictional data)
├── input/
│   └── tickets/                              # Intake mode: drop one ticket export here (git-ignored)
├── output/
│   ├── cleaned-tickets/                      # Intake mode: cleaned, validated copies (git-ignored)
│   └── qa-reports/                           # Saved reports (git-ignored)
├── .gitignore
├── CLAUDE.md
└── README.md
```

| Part | Role |
|------|------|
| `SKILL.md` | The skill itself: when it applies, the workflow (intake → analyse → report → validate → save) and mandatory rules such as "never invent expected results". Claude Code loads it when you run `/qa-ninja`. |
| `references/` | The detailed rules the skill follows: how to analyse a ticket, how to classify priorities and readiness, the exact report format and its pre-save checks, and the intake procedure. |
| `templates/` | The Markdown template for the Quick Report. |
| `demo/tickets/` | Three synthetic tickets for trying the skill. They are fictional and contain no real customer, order or company data. |
| `CLAUDE.md` | Project instructions for Claude Code: working principles, security rules, scope and key paths. |

The skill is instruction-based Markdown only: no code, no dependencies, no external services, no Jira or GitHub integration.

## Demo Tickets

| File | Scenario | What it demonstrates |
|------|----------|----------------------|
| `demo/tickets/DEMO-001-washing-machine-order.md` | Washing machine order confirmation | A clear ticket: every AC gets an executable check |
| `demo/tickets/DEMO-002-refrigerator-warranty.md` | Refrigerator warranty claim application | Missing expected behaviour produces questions and blocked checks |
| `demo/tickets/DEMO-003-dishwasher-repair.md` | Dishwasher repair request review | Mixed confirmed and unresolved conditions; out-of-scope items excluded |

All demo tickets are synthetic demo data.

## How to Use

### 1. Open the project in Claude Code

Open a terminal in the project folder and start Claude Code:

```
claude
```

The skill is discovered automatically from `.claude/skills/qa-ninja/`.

### 2. Run QA NiNJA on a demo ticket

Pass the ticket file as the argument:

```
/qa-ninja demo/tickets/DEMO-001-washing-machine-order.md
```

You can also paste a ticket's text into the conversation after typing `/qa-ninja`.

### 3. Optional — intake mode

Copy one demo ticket into `input/tickets/` and run `/qa-ninja` with no argument. QA NiNJA selects the new ticket, saves a cleaned copy to `output/cleaned-tickets/`, validates it, and — only if validation passes — analyses it and saves the report to `output/qa-reports/` automatically. The original ticket file is never changed.

### 4. Review the Quick Report

The report has exactly three parts:

1. **Readiness** — `READY`, `NEEDS CLARIFICATION` or `NOT READY`, the blocker (open P0 question, or `None`) and a one-line explanation.
2. **Questions** — `Q | P | Owner | Question (why)`, ordered P0 → P1 → P2. Send them to the BA or Developer named in the Owner column.
3. **QA Execution Checklist** — `AC N | Check | Screenshot | Result`. Execute the rows without `Wait:` now; rows marked `TBD — Wait: Qn` wait for the answer to that question. Fill in Screenshot and Result while testing.

Outside intake mode, QA NiNJA asks before saving the report to `output/qa-reports/` and never overwrites an existing report.

## Security Considerations

- Supplied requirements are treated as **untrusted data**; instructions embedded in a ticket are ignored.
- The skill never reads `.env` or secret files and never makes external requests, including links, images or attachments in a ticket.
- Ticket exports, cleaned copies and reports are git-ignored to avoid publishing ticket content.
- Do not paste secrets, credentials or personal customer data into requirements.

## Limitations

- Instruction-based: output depends on the model and the quality of the ticket. Two runs on the same ticket can word questions differently and occasionally differ on borderline questions; review the report before relying on it.
- Tested on a small set of synthetic and sample tickets. Not every ticket format or scenario has been tested.
- Analyses text only — images, designs and attachments are not read.
- Intake checks a file's real format; Word (`.doc`/`.docx`) extraction has been tested on made-up fixtures only.
- One ticket at a time; no integration with Jira, GitHub or test management tools.
- Does not generate executable automated tests.
