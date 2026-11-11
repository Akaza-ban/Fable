# F08 Activity Log

> Design status: TARGET V1 (draft for team review)
>
> Implementation status: NOT STARTED

This folder specifies the **Activity Log** branch (F08) of the mind map for the *AI-powered IT ticket routing and processing system*. With about **8,000 tickets per day**, most of them handled by AI agents and automation (F04 D-AI-01, F05 flow A), people need to be able to answer afterwards: **who did what, when, why, and with what result**. F08 is the single, append-only audit trail that every branch writes to, and the screens that read it: the history of a ticket, the approval chain of an access request, the actions of a person or of the AI, and the workflow runs and their errors.

F08 **does not decide anything**. It records what other branches did and shows it to the people allowed to see it.

## Business context

- The AI approves access requests on its own within policy (F05 UC-WF-07) and resolves how-to tickets with KB answers (F04 UC-AI-10). When something goes wrong, the Admin must be able to show the exact chain: the request, what the AI understood, each policy condition, the decision, the API calls, the verification.
- The mind map asks for a **SOC-2 style audit trail**: immutable, chronological, tracking users, technicians, admins and automation (leaf 8.5).
- Requesters want to follow their ticket without calling IT, but must never see internal notes, AI confidence or policy details.
- Logs must never become a source of leaks: no password, token or key may be written.

## Scope

| Mind-map leaf | Use case |
|---|---|
| 8.5 Immutable chronological action logging | UC-NK-05 |
| 8.5 Actor tracking: user, technician, admin and automation | UC-NK-02 |
| 1.2 View ticket status & timeline; 2.4 ticket state history (history part) | UC-NK-01 |
| 7.3 / 7.5 Approval and verification trail ("append audit log") | UC-NK-03 |
| 7.6 Failure recovery: connector timeouts, API rejection, fallback to a human (history part) | UC-NK-04 |

## Boundary with other branches

**Other branches act and write log entries. F08 stores them unchanged and shows them by role.**

| Topic | F08 (this folder) | Owned elsewhere |
|---|---|---|
| Audit record structure (`actor_type`, action, before / after) | Stores and reads it; extends the scope (BR-NK-01) | F01 BR-09 defines it (`docs/mvp.md` on `feature/author`) |
| Ticket timeline | Renders the `#history` tab of SCR-T-02 (SEC-NK-01) | F03 owns SCR-T-02 and its other tabs |
| AI decisions, policy-condition results | Shows them as steps of the approval chain (SEC-NK-02) | F05 writes `ai_decisions`; **SCR-WF-31** (F05) is the Admin's full decision page |
| AI analyses, corrections, agent actions | Logs and shows them with model and prompt version | F04 produces them (UC-AI-01, UC-AI-07, UC-AI-10); F04 SEC-AI-15 shows analysis versions |
| Workflow runs and attempts | Lists and filters runs (SCR-NK-04, 05) | F05 writes them; *Retry* / *Resolved manually* are F05 actions (UC-WF-09) on the `#automation` tab |
| Masking secrets | Masks before writing (BR-NK-03) | F04 BR-AI-07 masks ticket text; F05 BR-WF-15 masks payloads; F08 is the last guard |
| Report exports and report settings | Records `REPORT_EXPORTED`, `REPORT_CONFIG_UPDATED`, `REPORT_DIGEST_SENT` | F07 |
| Figures computed from the log (M-04, M-05) | — | F07 |

## Documents

| File | Content |
|---|---|
| [01-features.md](01-features.md) | Feature breakdown F08.1–F08.7, out of scope |
| [02-use-cases.md](02-use-cases.md) | Actors, permission matrix, audit record, event catalog, use case diagram, UC-NK-01 … UC-NK-05 |
| [03-user-stories-invest.md](03-user-stories-invest.md) | Backlog US-NK-01 … US-NK-10 with MoSCoW and size, INVEST review |
| [04-acceptance-criteria.md](04-acceptance-criteria.md) | Given / When / Then criteria |
| [05-business-rules.md](05-business-rules.md) | BR-NK-01 … BR-NK-07 |
| [information-architecture/](information-architecture/README.md) | Pages and sections (SCR-NK, SEC-NK), routes and permissions, navigation, user flows, UI states |

## Open questions

| # | Question | Value used in these documents until decided |
|---|---|---|
| Q-NK-01 | The mind map lists manager approval for elevated permissions (7.2, 7.3); F05 V1 has no human approver. Is one needed later? | No human approver in V1. The approval chain can show that step without redesign if it is added. |
| Q-NK-02 | F01 says the `AI_AGENT` service account "may not grant permissions itself"; F05 lets the AI approve grants within policy. | The AI approves only within F05 BR-WF-02; the grant itself is executed by `AUTOMATION`. F01 and F05 owners to align wording. |
| Q-NK-03 | Automated actions: `SYSTEM` (F01 BR-09) or `AUTOMATION` (F04, F05 BR-WF-05)? | **`AUTOMATION`**, as in F04 and F05; Microsoft sync entries written by F01 as `SYSTEM` are shown as `AUTOMATION`. |
| Q-NK-04 | Does the F01 owner accept extending BR-09 to tickets, AI and workflows, and adding evidence ids (`ticket_id`, `decision_id`, `analysis_id`, `rule_id`, `sla_policy_version`, `run_id`) next to `suggestion_id`? | Yes (proposal). |
| Q-NK-05 | Are AI **reads** (every AI request) logged in the audit log? At 8,000 tickets a day this would make it huge. | No. Reads go to a separate access log with shorter retention; only AI actions that change or decide something go to the audit log. |
| Q-NK-06 | When `actor_type = AI`, does `actor_id` point to the service account or to a user? | The `AI_AGENT` service account; the domain agent (F04 `agent`) is stored in the entry's detail. |
| Q-NK-07 | How long is the log kept? Does "nobody can delete" need a retention exception? | 2 years online, then archived by a scheduled job outside the application account. To agree with the client's compliance rules. |
| Q-NK-08 | May a requester see the name of the technician handling their ticket? | Yes, the display name only. |
| Q-NK-09 | F05 shows AI decisions to the **assigned** technician; F01 scope is assigned **and group**. Which applies to the approval chain? | Assigned technician: full chain; other group members: summary. |
| Q-NK-10 | Is exporting the audit log needed? | Not in V1. If added: `audit:export`, logged like F07 exports. |

## Progress

- [x] Scope and boundary with F01, F03, F04, F05, F07
- [x] Audit record, event catalog, actor model aligned with F04 / F05 (`USER` / `AI` / `AUTOMATION`)
- [x] Use cases UC-NK-01 … UC-NK-05
- [x] User stories US-NK-01 … US-NK-10 (INVEST, MoSCoW)
- [x] Acceptance criteria and business rules
- [x] Information architecture (screens, routes, navigation, flows, UI states)
- [ ] Decide the open questions (Q-NK-02, Q-NK-03, Q-NK-04 with the F01 and F05 owners; Q-NK-07 with the client)
- [ ] Non-functional requirements and traceability (API, tables)

**Related:** [README](../../README.md) · [Previous: F07 Reports & Analytics](../f07-reports-analytics/README.md)
