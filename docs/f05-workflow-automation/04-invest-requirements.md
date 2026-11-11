# INVEST Requirements — Workflow & Automation (V1)

> Design status: TARGET V1
>
> Scope: F05 — US-WF-01 … US-WF-30

Each user story is small enough to implement and test independently. Authentication, authorization, validation, auditing and error handling are **shared acceptance constraints**, not hidden features.

## INVEST checklist

| Principle | Application in this backlog |
|---|---|
| **Independent** | Each story produces a demonstrable outcome and lists only unavoidable prerequisites. |
| **Negotiable** | UI layout and implementation details may change as long as the goal, permissions and business rules stay the same. |
| **Valuable** | Every story names the benefit to the user or the business. |
| **Estimable** | Inputs, outputs, roles, dependencies and failure outcomes are explicit. |
| **Small** | Each story targets one capability and can be split into API, application, persistence, UI and test tasks. |
| **Testable** | Every story links to observable acceptance criteria in [06-acceptance-criteria.md](03-acceptance-criteria.md). |

## Product backlog

MoSCoW priority: **Must** (required for V1), **Should** (important), **Could** (if time allows).

| ID | User story | Value | Dependencies | Priority | AC |
|---|---|---|---|---|---|
| US-WF-01 | ~~Configure assignment rules~~ — **Removed.** The team comes from the AI category and the categories each team is responsible for (F09). | — | — | — | — |
| US-WF-02 | ~~Order and dry-run assignment rules~~ — **Removed** with US-WF-01. A preview of the assignment is part of US-WF-21. | — | — | — | — |
| US-WF-03 | As an **Admin**, I want to define SLAs per priority and business calendar, so that every ticket has a clear deadline. | Measurable service commitment | — | Must | AC-UC-WF-02 |
| US-WF-04 | As a **Technician**, I want the SLA to pause while the AI is waiting for the requester's answer, so that I am not penalized for delays outside my control. | Fair SLA | US-WF-03 | Should | AC-UC-WF-02 |
| US-WF-05 | As an **Admin**, I want to enable/disable each AI action (including auto-approval), set confidence thresholds and the maximum clarification rounds, so that I control how much the AI may decide on its own. | AI risk control | AI Service (F04) | Must | AC-UC-WF-03 |
| US-WF-06 | As an **Admin**, I want to declare the automation action catalog (account check, account creation, access grant) with connectors and parameters, so that only predefined tasks can be handled by the AI. | Bounded automation | Declared connectors | Must | AC-UC-WF-03 |
| US-WF-07 | As an **Admin**, I want to declare, per department, the entitlements and account templates the AI may grant, with a maximum duration, so that the AI never grants access beyond scope. | Prevents wrongful grants | US-WF-06, departments (F09) | Must | AC-UC-WF-03 |
| US-WF-08 | As a **Requester**, I want my basic requests (account check, account creation, additional access) to be recognized and handled by the AI immediately, so that I don't have to wait for a technician. | Instant resolution of most tickets | US-WF-05, US-WF-06 | Must | AC-UC-WF-04 |
| US-WF-09 | As a **Requester**, I want the AI to ask me a specific question when it is not confident (max 2 rounds), so that my request is understood correctly instead of being handled wrongly. | Fewer wrong actions | US-WF-08 | Must | AC-UC-WF-04 |
| US-WF-10 | As a **Technician**, I want requests that need a human (hardware, outside AI scope, policy check failed) to be assigned to me automatically with an AI summary and the reason, so that I can start without re-reading everything. | Technicians only get work that needs them | US-WF-08, US-WF-21, team categories (F09) | Must | AC-UC-WF-05 |
| US-WF-11 | As a **Technician**, I want the system to respect my status (ON / OFF) and my current workload, and to move my not-started tickets to colleagues who are ON when I switch OFF, so that I am never given work I cannot do and nothing waits on me. | Fair, automatic distribution without a Team Lead | US-WF-10, availability status (F09) | Must | AC-UC-WF-05 |
| US-WF-12 | As a **Technician**, I want to be reminded when a ticket reaches its SLA warning threshold, so that I can act before it is overdue. | Fewer SLA breaches | US-WF-03 | Must | AC-UC-WF-06 |
| US-WF-13 | As an **Admin**, I want SLA-breached tickets to be escalated automatically (flag, notification to me, optional priority raise, automatic reassignment when not started or the assignee is unavailable), so that delays are fixed without anyone redistributing work by hand. | Fast reaction to delays | US-WF-03, US-WF-11 | Must | AC-UC-WF-06 |
| US-WF-14 | As a **Requester**, I want to be reminded when the AI is waiting for my answer, and the ticket to close automatically after N days without an answer (still reopenable), so that tickets don't hang forever. | Clean queues | US-WF-09 | Should | AC-UC-WF-06 |
| US-WF-15 | As a **Requester**, I want requests within my department's allowed catalog to be approved by the AI right after the policy check, so that I don't have to wait for anyone's approval. | Approval in seconds | US-WF-07, US-WF-08 | Must | AC-UC-WF-07 |
| US-WF-16 | As an **Admin**, I want to see each AI decision with its confidence, extracted information and the result of each policy condition, so that I can review it and adjust the configuration when the AI gets it wrong. | Explainable AI decisions | US-WF-15 | Must | AC-UC-WF-07 |
| US-WF-17 | As a **Requester**, I want the system to call the API automatically after AI approval (no duplicate grants, automatic retry on transient errors), so that I get my account / access right away without manual work. | Self-resolution without technicians | US-WF-15 | Must | AC-UC-WF-08 |
| US-WF-18 | As an **Admin**, I want time-bound access to be revoked automatically on expiry, so that stale access does not accumulate. | Security | US-WF-17 | Could | AC-UC-WF-08 |
| US-WF-19 | As a **Requester**, I want the system to verify the result on the external access-management system and then ask me to confirm, so that I am sure my request was really fulfilled. | Trustworthy result | US-WF-17 | Must | AC-UC-WF-09 |
| US-WF-20 | As a **Technician**, I want tickets whose execution or verification failed to be handed to me with the error log so I can retry or resolve manually, so that automation failures are never forgotten. | Clear ownership of failures | US-WF-17, US-WF-10 | Must | AC-UC-WF-09 |
| US-WF-21 | As an **Admin**, I want to configure workload scoring (points per priority, complexity of each request type, maximum load per technician, default support team), and to preview who would receive a sample ticket, so that tickets go to the technician who is really the least busy, not just the one holding the fewest tickets. | Realistic load balancing | Technical teams and their categories (F09) | Must | AC-UC-WF-01 |
| US-WF-22 | As a **Requester**, I want the ticket fields (request type, category, support level, requested for, business unit, site) to be filled automatically from what I wrote, whether I wrote an email or used the portal, so that I never have to pick a category or fill a form. | No category form | F04 analysis (UC-AI-01) | Must | AC-UC-WF-04 |
| US-WF-23 | As a **Technician**, I want every ticket to arrive with a priority already set from its impact and urgency, and its SLA attached, and to be able to correct it with a reason, so that I work on the right tickets first. | Correct order of work | US-WF-03, F04 priority (UC-AI-05) | Must | AC-UC-WF-04 |
| US-WF-24 | As a **Requester**, I want my ticket to go first to the AI agent of its domain and, when a person is needed, to a technician of the right sub-group, so that the most specialized handler takes it without being passed around. | Fewer hand-offs | US-WF-22, groups and sub-groups (F09), domain agents (F04) | Must | AC-UC-WF-04, AC-UC-WF-05 |
| US-WF-25 | As a **Requester**, I want a request I send twice (for example the same email sent again) to be added to my open ticket instead of creating a second one, with a way to undo it, so that my problem is handled once and I follow a single ticket. | No parallel work on the same request | F04 duplicate candidates (UC-AI-06), F03 link operation | Must | AC-UC-WF-10 |
| US-WF-26 | As a **Technician**, I want tickets from other people about the same problem I am already handling to be linked to mine and assigned to me, so that one person fixes the problem once and answers everyone. | One owner per problem | US-WF-25, US-WF-24 | Must | AC-UC-WF-10 |
| US-WF-27 | As an **Admin**, I want a burst of similar tickets on the same system to be grouped under one main ticket with a higher priority and to be alerted, so that a possible incident is noticed early. | Early incident detection | US-WF-26, F04 suspected-bug burst | Should | AC-UC-WF-10 |
| US-WF-28 | As a **Technician**, I want to return a ticket that is not for my team with a short reason and have the AI route it to the right team, so that I do not have to know every team or dispatch it myself. | No manual dispatching after a misroute | US-WF-24 | Must | AC-UC-WF-11 |
| US-WF-29 | As a **Requester**, I want my ticket to move automatically to a person or to the right team when the AI agent or the automation cannot handle it, so that I never have to submit it again. | Nothing gets stuck after a failure | US-WF-24, F04 domain agents | Must | AC-UC-WF-11 |
| US-WF-30 | As an **Admin**, I want re-routing to stop after a maximum number of attempts and never send a ticket back to the team that just returned it, and to be alerted when the limit is reached, so that tickets cannot bounce between teams. | Loop protection | US-WF-28 | Should | AC-UC-WF-11 |

Grouped by use case:

```text
US-WF-21          UC-WF-01  Configure workload scoring (US-WF-01, 02 removed)
US-WF-03–04   UC-WF-02  Configure SLA
US-WF-05–07   UC-WF-03  Configure AI processing scope
US-WF-08–09, 22–24   UC-WF-04  Route ticket (full analysis, level, fields, priority, owner)
US-WF-10–11   UC-WF-05  Auto-assign technician (group / sub-group)
US-WF-12–14   UC-WF-06  SLA reminders & warnings
US-WF-15–16   UC-WF-07  AI approves in-scope request
US-WF-17–18   UC-WF-08  Execute via API
US-WF-19–20   UC-WF-09  Verify / hand off on failure
US-WF-25–27   UC-WF-10  Handle duplicate / similar tickets
US-WF-28–30   UC-WF-11  AI re-routes on failure or misroute
```

## Per-story INVEST review

`PASS` means the current design provides enough information for planning and acceptance. It does **not** claim that code or tests already exist.

| Story | I | N | V | E | S | T | Notes |
|---|---|---|---|---|---|---|---|
| US-WF-01 | — | — | — | — | — | — | Removed: no assignment rules. |
| US-WF-02 | — | — | — | — | — | — | Removed with US-WF-01. |
| US-WF-03 | PASS | PASS | PASS | PASS | PASS | PASS | Three priority levels P1–P3 with defaults; policies are versioned. |
| US-WF-04 | PASS | PASS | PASS | PASS | PASS | PASS | Only pause state: `NEEDS_INFO`. |
| US-WF-05 | PASS | PASS | PASS | PASS | PASS | PASS | 6 AI actions plus `AGENT_FIRST` per domain, threshold 0.00–1.00, default priority P2, default 2 clarification rounds. |
| US-WF-06 | PASS | PASS | PASS | PASS | PASS | PASS | 4 V1 operations: `CHECK_ACCOUNT`, `CREATE_ACCOUNT`, `GRANT_ACCESS`, `REVOKE_ACCESS`. |
| US-WF-07 | PASS | PASS | PASS | PASS | PASS | PASS | Catalog per department; check does not use AI. |
| US-WF-08 | PASS | PASS | PASS | PASS | PASS | PASS | Flows A/B/C have explicit conditions. |
| US-WF-09 | PASS | NEEDS CONFIRM | PASS | PASS | PASS | PASS | Max 2 clarification rounds is a proposed value, pending confirmation. |
| US-WF-10 | PASS | PASS | PASS | PASS | PASS | PASS | Notification must include the routing reason. |
| US-WF-11 | PASS | PASS | PASS | PASS | PASS | PASS | Two statuses `ON` / `OFF`; only not-started tickets move when going `OFF`. Status itself is provided by F09. |
| US-WF-12 | PASS | PASS | PASS | PASS | PASS | PASS | One reminder per threshold. |
| US-WF-13 | PASS | PASS | PASS | PASS | PASS | PASS | Priority raise is a configuration option; `IN_PROGRESS` tickets whose assignee is `ON` are not moved. |
| US-WF-14 | PASS | PASS | PASS | PASS | PASS | PASS | N defaults to 5 business days; reopen within 7 days. |
| US-WF-15 | PASS | NEEDS CONFIRM | PASS | PASS | PASS | PASS | Open: whether sensitive entitlements still need Admin approval (see Open questions in README). |
| US-WF-16 | PASS | PASS | PASS | PASS | PASS | PASS | Read from `ai_decisions`; no need to re-run the AI. |
| US-WF-17 | PASS | PASS | PASS | PASS | PASS | PASS | Idempotency key, 3 retries at 1/5/15 minutes; demoable with a mock connector. |
| US-WF-18 | PASS | PASS | PASS | PASS | PASS | PASS | Revocation reuses the `workflow_run` mechanism. |
| US-WF-19 | PASS | PASS | PASS | PASS | PASS | PASS | Connectors must expose a check operation. |
| US-WF-20 | PASS | PASS | PASS | PASS | PASS | PASS | Retry always re-checks the current policy. |
| US-WF-21 | PASS | NEEDS CONFIRM | PASS | PASS | PASS | PASS | Default points (P1 = 3, P2 = 2, P3 = 1), factors (×1/×1.5/×2) and maximum load 20 are proposed values; complexity comes from the request type in V1. |
| US-WF-22 | PASS | PASS | PASS | PASS | PASS | PASS | Fields come from the F04 analysis; each is marked "Filled by AI"; low-confidence fields are asked for. |
| US-WF-23 | PASS | NEEDS CONFIRM | PASS | PASS | PASS | PASS | Default P2 when the AI is not confident. F04 still suggests P1–P4 while F05 uses P1–P3: the scales must be aligned. |
| US-WF-24 | PASS | PASS | PASS | PASS | PASS | PASS | Depends on F09 for sub-groups and on F04 for domain agents. |
| US-WF-25 | PASS | PASS | PASS | PASS | PASS | PASS | Same requester only; undo within 7 days; uses F04's duplicate threshold 0.85. |
| US-WF-26 | PASS | PASS | PASS | PASS | PASS | PASS | Never closes another person's ticket; falls back to the normal selection when the owner is OFF or full. |
| US-WF-27 | PASS | NEEDS CONFIRM | PASS | PASS | PASS | PASS | Burst rule 5 tickets / 60 minutes is the F04 default, to be calibrated. |
| US-WF-28 | PASS | PASS | PASS | PASS | PASS | PASS | Reason required; the group just left is excluded from the new analysis. |
| US-WF-29 | PASS | PASS | PASS | PASS | PASS | PASS | Agent failure keeps the group; wrong target changes it. Depends on F04 agent result codes. |
| US-WF-30 | PASS | NEEDS CONFIRM | PASS | PASS | PASS | PASS | Maximum 2 re-routes is a proposed value. |

## Cross-cutting acceptance constraints

- Only 3 roles exist: `ADMIN`, `TECHNICIAN`, `REQUESTER`; the only system actor is the **AI Service**. There is **no Team Lead** and no `is_lead` flag: work is distributed by the system, and only the Admin may reassign manually.
- The AI only self-approves when the **deterministic policy check** passes; the API — not the React client nor the AI — enforces RBAC, the whitelist and the entitlement catalog.
- Every action not performed by a user is logged with source `AI` or `AUTOMATION` and its grounds (`decisionId`, category and team, `runId`).
- Connector secrets never appear in the UI, API responses, logs or audit records.
- API errors use `ProblemDetails` (`400`, `403`, `404`, `409`).
- Times are stored in UTC; SLAs are computed in business hours of the configured calendar.
- The system must sustain the reference load of **8,000 tickets per day** (see NFRs).

**Related:** [Use cases](02-use-cases.md) · [Acceptance criteria](03-acceptance-criteria.md) · [Business rules](../requirements/business-rules.md)
