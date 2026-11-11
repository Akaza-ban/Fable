# User Stories (INVEST) — F08 Activity Log

> Design status: TARGET V1
>
> Scope: US-NK-01 … US-NK-10

Each story is small enough to build and test on its own, or its review says why not. Authorization, ticket scope, masking and append-only storage are **shared acceptance constraints**, not hidden features.

## INVEST checklist

| Principle | Application in this backlog |
|---|---|
| **Independent** | Reading stories depend only on entries existing; writing stories (US-NK-03, 08, 09) are the foundation and are built first. |
| **Negotiable** | Timeline layout and storage engine may change, while append-only, masking and visibility rules stay fixed. |
| **Valuable** | Every story names who gains trust or time from the trail. |
| **Estimable** | Event types, roles, visibility classes and failure cases are explicit, and every story has a size. |
| **Small** | **S** = one rule or filter; **M** = one view with role variants; **L** = a guarantee across branches (transactions, ordering). |
| **Testable** | Every story has Given / When / Then criteria in [04-acceptance-criteria.md](04-acceptance-criteria.md). |

## Product backlog

MoSCoW: **Must** (required for V1), **Should** (planned, but the core works without it), **Could** (if time allows).

| ID | User story | Value | Dependencies | MoSCoW | Size | AC |
|---|---|---|---|---|---|---|
| US-NK-01 | As a **Technician**, I want to see everything that happened to a ticket, by whom and when, so that I can pick it up without asking around. | Faster hand-over | US-NK-03, F03 SCR-T-02 | Must | M | AC-UC-NK-01 |
| US-NK-02 | As a **Requester**, I want to follow what happened to my ticket, so that I know where it stands without calling IT. | Fewer follow-up calls | US-NK-01 | Must | S | AC-UC-NK-01 |
| US-NK-03 | As an **Admin**, I want the log to be impossible to edit or delete, so that it can be trusted when something goes wrong. | Trustworthy trail (SOC-2) | — | Must | M | AC-UC-NK-05 |
| US-NK-04 | As an **Admin**, I want to search everything a person, the AI or automation did, so that I can review AI behaviour and investigate incidents. | Investigation in minutes | US-NK-03 | Must | M | AC-UC-NK-02 |
| US-NK-05 | As an **Admin**, I want to see step by step why an access request was approved, asked back or sent to a person, so that I can check the AI's decisions and adjust its scope. | Explainable approvals | F05 `ai_decisions`, runs | Must | M | AC-UC-NK-03 |
| US-NK-06 | As an **Admin**, I want to filter workflow runs by status and error type, so that I can find broken connectors or AI outages quickly. | Faster recovery | F05 runs | Must | M | AC-UC-NK-04 |
| US-NK-07 | As a **Requester**, I want my ticket to reach a person when the AI fails, and the history to say so, so that it is never stuck. | No silent failures | F05 UC-WF-04 | Must | S | AC-UC-NK-04 |
| US-NK-08 | As an **Admin**, I want passwords, tokens and keys masked before anything is logged, so that the log is not a source of leaks. | No leaked secrets | US-NK-03 | Must | M | AC-UC-NK-05 |
| US-NK-09 | As an **Admin**, I want a permission to be granted only if the action is logged, so that there is never an access change nobody can trace. | No untraceable grants | US-NK-03, F05 UC-WF-08 | Must | L | AC-UC-NK-05 |
| US-NK-10 | As an **Admin**, I want every AI analysis, correction, KB answer and escalation logged with the domain agent and model / prompt version, so that I can trace why the AI did something after a change. | Safe AI changes | F04 UC-AI-01, 07, 10 | Should | M | AC-UC-NK-02 |

Grouped by use case:

```text
US-NK-01–02      UC-NK-01  Ticket history
US-NK-04, 10     UC-NK-02  Track actions of people and AI
US-NK-05         UC-NK-03  Approval chain
US-NK-06–07      UC-NK-04  Workflow runs and errors
US-NK-03, 08, 09 UC-NK-05  Write the log
```

## INVEST review

| Story | I | N | V | E | S | T | Note |
|---|---|---|---|---|---|---|---|
| US-NK-01 | PARTIAL | PASS | PASS | PASS | PASS | PASS | Needs entries written by other branches (UC-NK-05). |
| US-NK-02 | PARTIAL | PASS | PASS | PASS | PASS | PASS | Reuses the US-NK-01 timeline with the allowlist. |
| US-NK-03 | PASS | PASS | PASS | PASS | PASS | PASS | Enforced in the API (`405`) and in the database grants. |
| US-NK-04 | PASS | PASS | PASS | PASS | PASS | PASS | |
| US-NK-05 | PASS | PASS | PASS | PASS | PASS | PASS | Full decision detail stays in F05 SCR-WF-31; technicians see a summary (Q-NK-09 resolved). |
| US-NK-06 | PASS | PASS | PASS | PASS | PASS | PASS | Run detail overlaps F05 `getWorkflowRun`: one endpoint owned by F05. |
| US-NK-07 | PASS | PASS | PASS | PASS | PASS | PASS | Routing is F05; F08 records and shows it. |
| US-NK-08 | PASS | PASS | PASS | PASS | PASS | PASS | Last guard after F04 BR-AI-07 and F05 BR-WF-15. |
| US-NK-09 | PASS | PASS | PASS | PASS | NEEDS CONFIRM | PASS | Cross-branch guarantee with F05; may split into "write before call" and "retry result entry". |
| US-NK-10 | PASS | PASS | PASS | PASS | PASS | PASS | Fields from F04 BR-AI-02. |

## Cross-cutting acceptance constraints

- Only 3 roles exist: `ADMIN`, `TECHNICIAN`, `REQUESTER` (no Team Lead); the only system actor is the **AI Service**. Entry sources are `USER`, `AI`, `AUTOMATION`.
- Nobody, including Admin, can edit or delete an entry (BR-NK-02).
- Scope and the requester allowlist are applied in the API, not in the UI (BR-NK-04).
- No secret reaches an entry (BR-NK-03).
- API errors use `ProblemDetails` (`403`, `405`, `422`, `503`).
- Times are stored in UTC and shown in `Asia/Ho_Chi_Minh`.
- The log sustains **8,000 tickets per day** and every event they produce (see NFRs).

**Related:** [Use cases](02-use-cases.md) · [Acceptance criteria](04-acceptance-criteria.md) · [Business rules](05-business-rules.md)
