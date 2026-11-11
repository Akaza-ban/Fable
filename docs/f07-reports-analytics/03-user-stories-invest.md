# User Stories (INVEST) — F07 Reports & Analytics

> Design status: TARGET V1
>
> Implementation status: NOT STARTED
>
> Scope: US-BC-01 … US-BC-09, US-BC-11

Each story is small enough to build and test on its own, or its review says why not. Authorization, ticket scope, the shared metric definitions and audit logging are **shared acceptance constraints**, not hidden features.

## INVEST checklist

| Principle | Application in this backlog |
|---|---|
| **Independent** | Each story is one report or one capability; they share only the metric definitions and the filter bar. |
| **Negotiable** | Charts, layout and pre-aggregation may change, while definitions, scope and read-only behaviour stay fixed. |
| **Valuable** | Every story names the decision the figure supports. |
| **Estimable** | Data sources, metric codes, roles and failure cases ("not enough data", "unavailable") are explicit, and every story has a size. |
| **Small** | **S** = one figure or setting; **M** = a report tab with several figures and lists; **L** = several data sources joined with exclusion rules. |
| **Testable** | Every story has Given / When / Then criteria in [04-acceptance-criteria.md](04-acceptance-criteria.md), checked against the reference data set RD-BC-01. |

## Product backlog

MoSCoW: **Must** (required for V1), **Should** (planned, but the core works without it), **Could** (if time allows).

| ID | User story | Value | Dependencies | MoSCoW | Size | AC |
|---|---|---|---|---|---|---|
| US-BC-01 | As an **Admin**, I want to see how many tickets come in, stay open and get closed per type, domain, group, priority and channel over time, so that I can spot overloaded groups and recurring problems. | Visible backlog | F03 tickets | Must | M | AC-UC-BC-01 |
| US-BC-02 | As a **Technician**, I want report figures limited to tickets assigned to me and my groups, so that I can follow my team's work without seeing other teams' data. | Safe self-service figures | US-BC-01, F01 scope | Must | M | AC-UC-BC-01 |
| US-BC-03 | As an **Admin**, I want response and resolution times and the tickets that breached or are about to breach their SLA, so that I can fix bottlenecks. | Fewer breaches | F05 SLA events | Must | M | AC-UC-BC-02 |
| US-BC-04 | As an **Admin**, I want the share of tickets closed with no technician, by path and by domain agent, against the target, so that I can decide whether to widen or narrow what the AI may do. | Measures agent first (D-AI-01) | F04 resolution codes, audit log | Must | L | AC-UC-BC-03 |
| US-BC-05 | As an **Admin**, I want satisfaction scores next to how many people rated, so that I do not draw conclusions from a few ratings. | Honest satisfaction figure | F03 ratings | Should | S | AC-UC-BC-04 |
| US-BC-06 | As an **Admin**, I want to export the report on screen to CSV or XLSX, so that I can share it with management. | Shareable figures | US-BC-01 … 05, F08 log | Must | M | AC-UC-BC-05 |
| US-BC-07 | As an **Admin**, I want to set the thresholds, target and limits, and see the observation window in use, so that the figures match how we work. | Correct figures | — | Must | S | AC-UC-BC-06 |
| US-BC-08 | As an **Admin**, I want the route A / B / C share and the escalations to technicians by class (cannot resolve, system bug, software requirement), so that I know why tickets still reach people. | Targets the right fix | F05 `ai_decisions`, F04 `escalationClass` | Must | M | AC-UC-BC-03 |
| US-BC-09 | As an **Admin**, I want AI corrections, helpful votes and failed analyses by model and prompt version, so that I can tell whether a change made the AI better or worse. | Safe AI changes | F04 UC-AI-07 data | Should | M | AC-UC-BC-03 |
| US-BC-11 | As an **Admin**, I want a weekly email with the key figures, so that management follows the trend without signing in. | Regular visibility | US-BC-01, 03, 04 | Could | S | AC-UC-BC-08 |

Grouped by use case:

```text
US-BC-01–02   UC-BC-01  Ticket volume and status
US-BC-03      UC-BC-02  Handling time and SLA
US-BC-04,08,09 UC-BC-03 AI self-resolution and quality
US-BC-05      UC-BC-04  Requester satisfaction
US-BC-06      UC-BC-05  Filter and export
US-BC-07      UC-BC-06  Report parameters
US-BC-11      UC-BC-08  Weekly digest
```

## INVEST review

| Story | I | N | V | E | S | T | Note |
|---|---|---|---|---|---|---|---|
| US-BC-01 | PASS | PASS | PASS | PASS | PASS | PASS | Count by creation date. |
| US-BC-02 | PARTIAL | PASS | PASS | PASS | PASS | PASS | Needs US-BC-01 to have something to scope. |
| US-BC-03 | PASS | PASS | PASS | PASS | PASS | PASS | Breach read from F05, never recomputed. |
| US-BC-04 | PASS | NEEDS CONFIRM | PASS | PASS | PASS | PASS | Target 50% (Q-AI-14) vs 25–35% (mind map): Q-BC-13. |
| US-BC-05 | PASS | PASS | PASS | PASS | PASS | PASS | |
| US-BC-06 | PARTIAL | PASS | PASS | PASS | PASS | PASS | Needs at least one report; needs the F08 log write. |
| US-BC-07 | PASS | PASS | PASS | PASS | PASS | PASS | SLA targets stay in F05. |
| US-BC-08 | PASS | PASS | PASS | PASS | PASS | PASS | `escalationClass` is set by F04 and F05 (F04 README, change 6). |
| US-BC-09 | PASS | PASS | PASS | PASS | PASS | PASS | Uses AC-AI-14.3 (votes countable by model and prompt version). |
| US-BC-11 | PASS | NEEDS CONFIRM | PASS | PASS | PASS | PASS | Recipients: Q-BC-14. |

## Cross-cutting acceptance constraints

- Only 3 roles exist: `ADMIN`, `TECHNICIAN`, `REQUESTER` (no Team Lead); the only system actor is the **AI Service**.
- Every figure follows the definitions M-01 … M-11, whatever page shows it (BR-BC-01).
- Scope is filtered in the API query before aggregation, never in the UI (BR-BC-02).
- No report endpoint changes data (BR-BC-03).
- Every export and parameter change is in the audit log with source `USER`; the digest with source `AUTOMATION`.
- API errors use `ProblemDetails` (`400`, `403`, `422`, `503`).
- Times are stored in UTC and shown in `Asia/Ho_Chi_Minh`.

**Related:** [Use cases](02-use-cases.md) · [Acceptance criteria](04-acceptance-criteria.md) · [Business rules](05-business-rules.md)
