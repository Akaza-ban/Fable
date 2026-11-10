# INVEST Requirements — Workflow & Automation (V1)

> Design status: TARGET V1
>
> Scope: F05 — US-WF-01 … US-WF-20

Each user story is small enough to implement and test independently. Authentication, authorization, validation, auditing and error handling are **shared acceptance constraints**, not hidden features.

## INVEST checklist

| Principle | Application in this backlog |
|---|---|
| **Independent** | Each story produces a demonstrable outcome and lists only unavoidable prerequisites. |
| **Negotiable** | UI layout and implementation details may change as long as the goal, permissions and business rules stay the same. |
| **Valuable** | Every story names the benefit to the user or the business. |
| **Estimable** | Inputs, outputs, roles, dependencies and failure outcomes are explicit. |
| **Small** | Each story targets one capability and can be split into API, application, persistence, UI and test tasks. |
| **Testable** | Every story links to observable acceptance criteria in [06-acceptance-criteria.md](06-acceptance-criteria.md). |

## Product backlog

MoSCoW priority: **Must** (required for V1), **Should** (important), **Could** (if time allows).

| ID | User story | Value | Dependencies | Priority | AC |
|---|---|---|---|---|---|
| US-WF-01 | As an **Admin**, I want to create and edit assignment rules based on conditions (request type, category, priority, department) pointing to a technical team and a distribution strategy, so that tickets that need a human reach the right team automatically. | No manual dispatching | Technical teams (F09) | Must | AC-UC-WF-01 |
| US-WF-02 | As an **Admin**, I want to order the rules and dry-run them on a sample ticket, so that I know the assignment outcome before enabling them. | Fewer configuration mistakes | US-WF-01 | Should | AC-UC-WF-01 |
| US-WF-03 | As an **Admin**, I want to define SLAs per priority and business calendar, so that every ticket has a clear deadline. | Measurable service commitment | — | Must | AC-UC-WF-02 |
| US-WF-04 | As a **Technician**, I want the SLA to pause while the AI is waiting for the requester's answer, so that I am not penalized for delays outside my control. | Fair SLA | US-WF-03 | Should | AC-UC-WF-02 |
| US-WF-05 | As an **Admin**, I want to enable/disable each AI action (including auto-approval), set confidence thresholds and the maximum clarification rounds, so that I control how much the AI may decide on its own. | AI risk control | AI Service (F04) | Must | AC-UC-WF-03 |
| US-WF-06 | As an **Admin**, I want to declare the automation action catalog (account check, account creation, access grant) with connectors and parameters, so that only predefined tasks can be handled by the AI. | Bounded automation | Declared connectors | Must | AC-UC-WF-03 |
| US-WF-07 | As an **Admin**, I want to declare, per department, the entitlements and account templates the AI may grant, with a maximum duration, so that the AI never grants access beyond scope. | Prevents wrongful grants | US-WF-06, departments (F09) | Must | AC-UC-WF-03 |
| US-WF-08 | As a **Requester**, I want my basic requests (account check, account creation, additional access) to be recognized and handled by the AI immediately, so that I don't have to wait for a technician. | Instant resolution of most tickets | US-WF-05, US-WF-06 | Must | AC-UC-WF-04 |
| US-WF-09 | As a **Requester**, I want the AI to ask me a specific question when it is not confident (max 2 rounds), so that my request is understood correctly instead of being handled wrongly. | Fewer wrong actions | US-WF-08 | Must | AC-UC-WF-04 |
| US-WF-10 | As a **Technician**, I want requests that need a human (hardware, outside AI scope, policy check failed) to be assigned to me automatically with an AI summary and the reason, so that I can start without re-reading everything. | Technicians only get work that needs them | US-WF-01, US-WF-08 | Must | AC-UC-WF-05 |
| US-WF-11 | As a **Team Lead**, I want tickets that match no rule to go to a manual triage queue, and to be able to reassign tickets with a reason, so that no ticket is missed or misassigned. | Nothing slips through | US-WF-10 | Must | AC-UC-WF-05 |
| US-WF-12 | As a **Technician**, I want to be reminded when a ticket reaches its SLA warning threshold, so that I can act before it is overdue. | Fewer SLA breaches | US-WF-03 | Must | AC-UC-WF-06 |
| US-WF-13 | As a **Team Lead**, I want SLA-breached tickets to be escalated automatically (notification, flag, optional priority raise), so that I can intervene in time. | Fast reaction to delays | US-WF-03 | Must | AC-UC-WF-06 |
| US-WF-14 | As a **Requester**, I want to be reminded when the AI is waiting for my answer, and the ticket to close automatically after N days without an answer (still reopenable), so that tickets don't hang forever. | Clean queues | US-WF-09 | Should | AC-UC-WF-06 |
| US-WF-15 | As a **Requester**, I want requests within my department's allowed catalog to be approved by the AI right after the policy check, so that I don't have to wait for anyone's approval. | Approval in seconds | US-WF-07, US-WF-08 | Must | AC-UC-WF-07 |
| US-WF-16 | As an **Admin**, I want to see each AI decision with its confidence, extracted information and the result of each policy condition, so that I can review it and adjust the configuration when the AI gets it wrong. | Explainable AI decisions | US-WF-15 | Must | AC-UC-WF-07 |
| US-WF-17 | As a **Requester**, I want the system to call the API automatically after AI approval (no duplicate grants, automatic retry on transient errors), so that I get my account / access right away without manual work. | Self-resolution without technicians | US-WF-15 | Must | AC-UC-WF-08 |
| US-WF-18 | As an **Admin**, I want time-bound access to be revoked automatically on expiry, so that stale access does not accumulate. | Security | US-WF-17 | Could | AC-UC-WF-08 |
| US-WF-19 | As a **Requester**, I want the system to verify the result on the external access-management system and then ask me to confirm, so that I am sure my request was really fulfilled. | Trustworthy result | US-WF-17 | Must | AC-UC-WF-09 |
| US-WF-20 | As a **Technician**, I want tickets whose execution or verification failed to be handed to me with the error log so I can retry or resolve manually, so that automation failures are never forgotten. | Clear ownership of failures | US-WF-17, US-WF-10 | Must | AC-UC-WF-09 |

Grouped by use case:

```text
US-WF-01–02   UC-WF-01  Configure assignment rules
US-WF-03–04   UC-WF-02  Configure SLA
US-WF-05–07   UC-WF-03  Configure AI processing scope
US-WF-08–09   UC-WF-04  Route ticket (AI self-service / clarification)
US-WF-10–11   UC-WF-05  Auto-assign technician
US-WF-12–14   UC-WF-06  SLA reminders & warnings
US-WF-15–16   UC-WF-07  AI approves in-scope request
US-WF-17–18   UC-WF-08  Execute via API
US-WF-19–20   UC-WF-09  Verify / hand off on failure
```

## Per-story INVEST review

`PASS` means the current design provides enough information for planning and acceptance. It does **not** claim that code or tests already exist.

| Story | I | N | V | E | S | T | Notes |
|---|---|---|---|---|---|---|---|
| US-WF-01 | PASS | PASS | PASS | PASS | PASS | PASS | AND conditions; new rules are disabled by default. |
| US-WF-02 | PASS | PASS | PASS | PASS | PASS | PASS | Dry run writes no data; first match wins. |
| US-WF-03 | PASS | PASS | PASS | PASS | PASS | PASS | P1–P4 defaults provided; policies are versioned. |
| US-WF-04 | PASS | PASS | PASS | PASS | PASS | PASS | Only pause state: `NEEDS_INFO`. |
| US-WF-05 | PASS | PASS | PASS | PASS | PASS | PASS | 5 AI actions, threshold 0.00–1.00, default 2 clarification rounds. |
| US-WF-06 | PASS | PASS | PASS | PASS | PASS | PASS | 4 V1 operations: `CHECK_ACCOUNT`, `CREATE_ACCOUNT`, `GRANT_ACCESS`, `REVOKE_ACCESS`. |
| US-WF-07 | PASS | PASS | PASS | PASS | PASS | PASS | Catalog per department; check does not use AI. |
| US-WF-08 | PASS | PASS | PASS | PASS | PASS | PASS | Flows A/B/C have explicit conditions. |
| US-WF-09 | PASS | NEEDS CONFIRM | PASS | PASS | PASS | PASS | Max 2 clarification rounds is a proposed value, pending confirmation. |
| US-WF-10 | PASS | PASS | PASS | PASS | PASS | PASS | Notification must include the routing reason. |
| US-WF-11 | PASS | PASS | PASS | PASS | PASS | PASS | Reassignment reason required; SLA keeps running in `TRIAGE`. |
| US-WF-12 | PASS | PASS | PASS | PASS | PASS | PASS | One reminder per threshold. |
| US-WF-13 | PASS | PASS | PASS | PASS | PASS | PASS | Priority raise is a configuration option. |
| US-WF-14 | PASS | PASS | PASS | PASS | PASS | PASS | N defaults to 5 business days; reopen within 7 days. |
| US-WF-15 | PASS | NEEDS CONFIRM | PASS | PASS | PASS | PASS | Open: whether sensitive entitlements still need Admin approval (see Open questions in README). |
| US-WF-16 | PASS | PASS | PASS | PASS | PASS | PASS | Read from `ai_decisions`; no need to re-run the AI. |
| US-WF-17 | PASS | PASS | PASS | PASS | PASS | PASS | Idempotency key, 3 retries at 1/5/15 minutes; demoable with a mock connector. |
| US-WF-18 | PASS | PASS | PASS | PASS | PASS | PASS | Revocation reuses the `workflow_run` mechanism. |
| US-WF-19 | PASS | PASS | PASS | PASS | PASS | PASS | Connectors must expose a check operation. |
| US-WF-20 | PASS | PASS | PASS | PASS | PASS | PASS | Retry always re-checks the current policy. |

## Cross-cutting acceptance constraints

- Only 3 roles exist: `ADMIN`, `TECHNICIAN`, `REQUESTER`; the only system actor is the **AI Service**.
- The AI only self-approves when the **deterministic policy check** passes; the API — not the React client nor the AI — enforces RBAC, the whitelist and the entitlement catalog.
- Every action not performed by a user is logged with source `AI` or `AUTOMATION` and its grounds (`decisionId`, `ruleId`, `runId`).
- Connector secrets never appear in the UI, API responses, logs or audit records.
- API errors use `ProblemDetails` (`400`, `403`, `404`, `409`).
- Times are stored in UTC; SLAs are computed in business hours of the configured calendar.
- The system must sustain the reference load of **8,000 tickets per day** (see NFRs).

**Related:** [Use cases](03-use-cases.md) · [Acceptance criteria](06-acceptance-criteria.md) · [Business rules](requirements/business-rules.md)
