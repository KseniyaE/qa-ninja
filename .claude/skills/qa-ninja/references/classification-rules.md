# QA NiNJA — Classification Rules

Use these classifications consistently in every report. Classifications are qualitative and evidence-based. **Do not use numerical risk scores.**

---

## 1. Requirement Status

| Status | Definition | How to determine |
|--------|-----------|------------------|
| **CONFIRMED** | The requirement is explicitly stated and testable. | Trigger, input, rule and expected result are all stated in the input; only one reasonable interpretation exists. |
| **INCOMPLETE** | The requirement exists but lacks information needed to test it. | One or more completeness dimensions (see analysis rules §1) is missing, e.g. no error behavior or no expected message. |
| **AMBIGUOUS** | The requirement can be read in more than one way. | Two or more reasonable interpretations lead to different tests or results. |
| **CONTRADICTORY** | The requirement conflicts with another statement in the input. | Both statements cannot be true at the same time. Silence or omission is never a contradiction. |

If a requirement is both incomplete and ambiguous, use the status that blocks testing more: **CONTRADICTORY > AMBIGUOUS > INCOMPLETE > CONFIRMED**, and mention the secondary issue in the notes.

## 2. Impact

Impact reflects **what happens if the behaviour is wrong** and **how central it is to the feature**, judged from the input.

| Impact | Criteria (any one is sufficient) |
|--------|----------------------------------|
| **HIGH** | Core acceptance criterion; blocks the main user journey; causes financial, legal, safety or data-integrity impact; affects all or most users. |
| **MEDIUM** | Important alternative or error flow; affects a meaningful subset of users; has a workaround. |
| **LOW** | Cosmetic, rare edge case, or minimal user/business impact. |

## 3. Priority (shown in reports)

HIGH / MEDIUM / LOW above is the **impact assessment**. The priority adds **release relevance** and is the priority shown in the Questions table.

| Priority | Rule | Impact level |
|----------|------|--------------|
| **P0 — Critical blocker** | HIGH impact **and** prevents safe or reliable delivery of the core feature: the main flow breaks or cannot be verified; financial, legal, safety, data-integrity or access impact; or a critical dependency is missing. | HIGH only |
| **P1 — Important question** | Needed for reliable testing and final QA sign-off: core ACs, main happy path, critical negative paths, core business rules, essential regression. | HIGH that is not P0, or MEDIUM within the MVP scope |
| **P2 — Secondary risk** | Minor uncertainty or non-critical edge case; improves confidence but does not block the main QA flow. | LOW, or MEDIUM outside the MVP scope |

Safeguards:
- A HIGH-impact item is never P2.
- A P2 item never affects readiness.
- Do not promote every unclear detail to P0 or P1.
- **Missing requirement content** (e.g. a link with no destination, an undefined outcome of a core action) is classified by its impact and is never downgraded because the ticket is silent on it.
- **QA execution dependencies are not requirement questions.** When the input fully specifies an outcome and only *how QA can observe it* is unstated (access to an admin portal or test inbox, test account, test data, environment, tooling), do not add a question, do not block the check, and do not let it affect readiness — it is test preparation. Raise it as a question only if the input states that the outcome cannot be observed. If useful, it may be named in the readiness explanation line when one is shown.

## 4. Test Readiness Status

| Status | Definition | How to determine |
|--------|-----------|------------------|
| **READY** | Testing can start with confidence. | Every core requirement is CONFIRMED; no contradictions; remaining open questions are P2 only. |
| **NEEDS CLARIFICATION** | Testing can partially start, but answers are required for full coverage. | At least one requirement is INCOMPLETE or AMBIGUOUS and affects P0/P1 checks, or a P0 question is open, but the main flow is testable. |
| **NOT READY** | Meaningful testing cannot start. | Any CONTRADICTORY core requirement; the main flow's expected result is unknown; or essential inputs (AC, business rules, environment/dependencies) are missing. |

Rules:

- Always justify the status briefly with references to specific questions.
- Never assign READY when any check is blocked by a P0 or P1 question, or any P0 question is open.
- An open P0 question means NOT READY only when it meets a NOT READY criterion above (e.g. the main flow's expected result is unknown). Otherwise it means at least NEEDS CLARIFICATION.
- Readiness says whether testing can proceed. Open P0 questions are release blockers and must be resolved before release, whatever the readiness status.

## 5. Question Ownership

Route each question by what its answer decides. Use one owner whenever possible.

- **BA** — business requirements, user expectations, product rules and functional behaviour.
- **Developer** — technical implementation details, technical constraints and technical verification (including test-environment access).
- **BA + Developer** — only when the answer genuinely requires both business and technical input. Never by default.
