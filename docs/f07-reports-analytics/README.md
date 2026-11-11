# F07 Reports & Analytics

> Design status: TARGET V1 (draft for team review)
>
> Implementation status: NOT STARTED

This folder specifies the **Reports & Analytics** branch (F07) of the mind map for the *AI-powered IT ticket routing and processing system*. The system receives about **8,000 internal IT tickets per day**. F07 turns the data that other branches record (tickets, SLA events, AI decisions, AI analyses, KB ratings, satisfaction ratings, the audit log) into **one set of agreed figures**: ticket volume, handling time and SLA, how much the AI resolves on its own, how good the AI output is, and requester satisfaction. It also exports those figures and sends a weekly digest.

F07 **only reads**. It never changes a ticket, an SLA, an AI decision or a log entry.

## Business context

- The client wants the AI to resolve as many tickets as possible without a technician (F04 decision **D-AI-01**, agent first). Whether that works must be measured the same way every week: F04 Q-AI-14 proposes **50% of all tickets closed without a technician within 3 months of go-live, measured weekly in F07**.
- Today the figures live in ManageEngine views and spreadsheets; each team counts differently. Management needs one definition per metric, the same on the Dashboard (F02), in reports and in exported files.
- At ~8,000 tickets per day (~2.9 million per year), reports must stay fast over long periods and must never show a team data it may not see.

## Scope

The leaves of mind-map branch **8. Dashboard & Governance Telemetry** that belong to F07. Leaf 8.5 (audit trail) belongs to [F08 Activity Log](../f08-activity-log/README.md); leaf 8.4 (team workload) and the live operational view belong to [F02 Dashboard](../f02-dashboard/README.md).

| Mind-map leaf | Use case |
|---|---|
| 8.1 Ticket volume & status metrics (incoming volume, backlog, closure rate, priority distribution P1–P4) | UC-BC-01 |
| 8.2 SLA compliance & resolution times (MTTA, MTTR, SLA %) | UC-BC-02 |
| 8.3 AI deflection & automation efficiency (tickets resolved without a human, KB deflection) | UC-BC-03 |
| *(added from F04)* AI output quality: corrections, helpful votes, by model and prompt version | UC-BC-03 |
| *(added from the mind map, 1.6)* Requester satisfaction (1–5 rating) | UC-BC-04 |
| 8.6 Operational reporting & export (Excel, CSV; PDF: Q-BC-11) | UC-BC-05 |
| 8.6 Automated weekly executive digest | UC-BC-08 |
| *(needed to run the reports)* Report parameters | UC-BC-06 |

## Boundary with other branches

**Other branches record the data. F07 computes figures from it and never writes back.**

| Topic | F07 (this folder) | Owned elsewhere |
|---|---|---|
| Ticket status, priority, resolution codes (`ANSWERED_BY_KB`, `SOLVED_BY_KB`, …) | Counts and times them | F03 stores them; F05 owns the state diagram |
| SLA targets, business calendar, breach | Reads `sla_breached`, `SLA_WARNING` / `SLA_BREACHED`; never recomputes | F05 UC-WF-02, UC-WF-06 (BR-WF-07, BR-WF-08) |
| AI routing outcome A / B / C | Aggregates `ai_decisions` (M-08) | F05 UC-WF-04 writes `ai_decisions` (NFR-WF-OBS-001) |
| AI cards on **SCR-WF-30** | Defines the metrics and serves `/reports/ai`: daily M-08 (auto-approved, asked back, routed to a person), weekly M-04 | F05 shows the cards; asked to call the same API and label them by these definitions (BR-BC-01) |
| AI analysis, escalation class, domain agent, corrections and votes | Reports on them (M-05, M-06, M-10) | F04 records them (UC-AI-07, UC-AI-10, AC-AI-14.3) |
| KB articles and their helpful ratings | Reads the ratings | F06 |
| Satisfaction rating 1–5 and comment | Aggregates (M-07) | F03 (mind-map 1.6) |
| Live counters for the home page | Same definitions, same API | F02 Dashboard |
| Audit log | Writes `REPORT_EXPORTED`, `REPORT_CONFIG_UPDATED`, `REPORT_DIGEST_SENT` through the shared log; reads it for M-04, M-05 | F08 Activity Log (BR-NK-01) |
| Users, groups, service accounts | Scope filtering and names | F01 / F09 |

## Documents

| File | Content |
|---|---|
| [01-features.md](01-features.md) | Feature breakdown F07.1–F07.9, out of scope |
| [02-use-cases.md](02-use-cases.md) | Actors, permission matrix, data sources, metric definitions M-01 … M-11, use case diagram, UC-BC-01 … UC-BC-06, UC-BC-08 |
| [03-user-stories-invest.md](03-user-stories-invest.md) | Backlog US-BC-01 … US-BC-09, US-BC-11 with MoSCoW and size, INVEST review |
| [04-acceptance-criteria.md](04-acceptance-criteria.md) | Given / When / Then criteria, reference data set |
| [05-business-rules.md](05-business-rules.md) | BR-BC-01 … BR-BC-07 |
| [information-architecture/](information-architecture/README.md) | Pages, modals, sections and emails (SCR-BC, SEC-BC, EML-BC), routes and permissions, navigation, user flows, UI states |

## Open questions

Roles, ticket scope and `BR-05 … BR-20` come from F01 (`docs/mvp.md` on the `feature/author` branch, not yet on `main`).

| # | Question | Value used in these documents until decided |
|---|---|---|
| Q-BC-02 | May a technician see colleagues' individual figures? | No. |
| Q-BC-03 | Is the AI effectiveness report (UC-BC-03) Admin only? | Admin only. |
| Q-BC-04 | Does an AI reply (question, KB answer) count as the first response for the SLA? | F05 decides; M-01 always reports human and AI first responses separately. |
| Q-BC-05 | Does `SLA_BREACHED` say which target was missed (response or resolution)? | Assumed yes; M-03 is split by target. F05 owner to confirm. |
| Q-BC-06 | A ticket whose priority was raised by SLA escalation: group by initial or final priority? | Final priority, with a flag. |
| Q-BC-07 | A ticket moved across groups is attributed to which group? | The last group that handled it. |
| Q-BC-08 | Who can rate a KB suggestion as helpful? | Requester and technician (F04 US-AI-14); owned by F04 / F06. |
| Q-BC-09 | Live queries over 12 months at ~2.9 million tickets per year will be slow. | Live queries for the demo; nightly pre-aggregated tables in production. F02 uses the same API. |
| Q-BC-10 | Is the reopen window the same for every `CLOSED` ticket? | **Resolved:** N in M-04 is the F03 reopen window (BR-TK-11, 7 days, same value as F05). F07 reads it and does not set it. |
| Q-BC-11 | Is PDF export needed (mind-map 8.6)? May rating comments be exported? | V1: CSV and XLSX; the weekly digest is an email. Comments are not exported. |
| Q-BC-12 | Permission names: `report:read` (F01 style) or camelCase (F05 `workflowConfigManage`)? | `resource:action`, mapped 1:1 when the team decides. |
| Q-BC-13 | The mind map targets 25–35% of tickets resolved without a human; F04 Q-AI-14 proposes 50% within 3 months. Which target does the report show? | Show the F04 target line (50%), configurable in UC-BC-06. To agree with the client. |
| Q-BC-14 | Who receives the weekly digest? | Admins, plus addresses an Admin adds (management). |

## Progress

- [x] Scope and boundary with F02, F03, F04, F05, F06, F08
- [x] Metric definitions M-01 … M-11, aligned with F04 agent-first (D-AI-01) and F05 SLA / routing
- [x] Use cases UC-BC-01 … UC-BC-06, UC-BC-08 (UC-BC-07 Team workload removed: leaf 8.4 is F02)
- [x] User stories US-BC-01 … US-BC-09, US-BC-11 (INVEST, MoSCoW)
- [x] Acceptance criteria and business rules
- [x] Information architecture (screens, routes, navigation, flows, UI states)
- [ ] Decide the open questions with the team (Q-BC-05 with the F05 owner; Q-BC-13 with the client)
- [ ] Non-functional requirements and traceability (API, tables)

**Related:** [README](../../README.md) · [Previous: F06 Knowledge Base](../f06-knowledge-base/README.md) · [Next: F08 Activity Log](../f08-activity-log/README.md)
