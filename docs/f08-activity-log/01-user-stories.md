# User stories — Activity Log

Each story has acceptance criteria (AC). Related use cases, `BR-NK-xx` and proposed permissions: [02-use-cases.md](02-use-cases.md). `BR-xx` rules: [mvp.md](../mvp.md); `BR-WF-xx`: [business-rules.md](../requirements/business-rules.md).

**US-NK-01 — Ticket history for staff** (UC-NK-01)
As a technician, I want to see everything that happened to a ticket, by whom and when, so that I can pick it up without asking around.
- AC1: The History tab lists every change with actor (name, "AI" or "System"), time, and before / after values.
- AC2: Automated entries link to their evidence (`decisionId`, `ruleId`, `runId`).
- AC3: Opening the history of a ticket outside my scope → `403`.

**US-NK-02 — Ticket history for the requester** (UC-NK-01)
As a requester, I want to follow what happened to my ticket, so that I know where it stands without calling IT.
- AC1: I only see allowlisted events: created, status changes, public replies, AI questions and my answers, approval / execution outcome, resolved, closed, reopened, cancelled, rated.
- AC2: Internal notes, AI confidence, policy check details, workflow attempts and SLA events never appear, neither in the UI nor in the API response.
- AC3: A new event type that has not been classified is hidden from me (BR-NK-04).
- AC4: Opening another requester's ticket history → `403`.

**US-NK-03 — Log cannot be changed** (UC-NK-01, UC-NK-02, UC-NK-05)
As an Admin, I want the activity log to be impossible to edit or delete, so that it can be trusted when something goes wrong.
- AC1: Any `PUT`, `PATCH` or `DELETE` on log endpoints → `405`, for every role including ADMIN.
- AC2: The application's database account cannot `UPDATE` or `DELETE` rows in the audit log table.
- AC3: A correction appears as a new entry referencing the old one; the old entry is unchanged.

**US-NK-04 — Track actions by users and AI** (UC-NK-02)
As an Admin, I want to search everything a person, the AI or the system did, so that I can review AI behaviour and investigate incidents.
- AC1: Filtering by `actor_type = AI` returns only AI actions; `SYSTEM` returns automated actions (Microsoft sync, assignment rules, SLA timers, API execution).
- AC2: Filters by actor, action, target, evidence id and time range can be combined; results are paginated.
- AC3: Opening an entry shows `before` / `after` as a diff.
- AC4: A disabled user still appears with their name in old entries.
- AC5: TECHNICIAN or REQUESTER calling the audit log API → `403`.

**US-NK-05 — Approval chain of a request** (UC-NK-03)
As an Admin, I want to see why a request was approved, asked back or sent to a human, step by step, so that I can check the AI's decisions and adjust its scope.
- AC1: The chain shows request → AI understanding → result of each policy condition → decision → execution attempts → verification → requester confirmation, each with actor and time.
- AC2: An auto-approved request shows `actor_type = AI` with its `decisionId`.
- AC3: A request approved but failed at execution is highlighted as "Approved — execution failed", with the assigned technician.
- AC4: A retry appears as new steps; earlier steps are unchanged.
- AC5: A requester sees only a summary status and short reason for requests on their own tickets.

**US-NK-06 — Workflow errors** (UC-NK-04)
As an Admin, I want to filter workflow runs by status and error type, so that I can find broken connectors or AI outages quickly.
- AC1: Filters by status (`QUEUED` … `MANUALLY_RESOLVED`), error type, operation, connector and time range.
- AC2: A run shows each attempt with time, duration, result code and masked request / response.
- AC3: The assigned technician sees the full attempt history of runs on their tickets; other technicians in the group see a summary only.
- AC4: REQUESTER calling the workflow log API → `403`.

**US-NK-07 — AI failure goes to a human** (UC-NK-04, UC-NK-05)
As a requester, I want my ticket to reach a person when the AI fails, so that it is not stuck.
- AC1: If the AI Service does not respond or returns an invalid result, the AI result is not applied and the ticket is routed to a human (assigned by rule, or `TRIAGE`).
- AC2: The history shows `TICKET_ROUTED_TO_HUMAN` with the error type (`AI_UNAVAILABLE` / `AI_INVALID_OUTPUT`).

**US-NK-08 — Secrets never reach the log** (UC-NK-05)
As an Admin, I want passwords, tokens and keys masked before anything is logged, so that the log does not become a source of leaks.
- AC1: A field named like `password`, `token`, `secret`, `api_key`, `authorization`, `cookie` or `otp` is stored as `"***"`.
- AC2: A `Bearer …` string, a JWT or an API key inside free text is stored as `"***"`.
- AC3: `password_hash`, `totp_secret_enc`, `api_key_hash` and connector credentials never appear in `before` / `after`.

**US-NK-09 — No grant without a log** (UC-NK-05)
As an Admin, I want a permission to be granted only if the action is logged, so that there is never an access change nobody can trace.
- AC1: If the "execution started" entry cannot be written, the connector is not called and the run ends `FAILED` with `AUDIT_UNAVAILABLE`.
- AC2: The ticket is then routed to a technician.
- AC3: If the connector call succeeded but writing the result failed, the entry is retried and the Admin is alerted; the connector is not called a second time.
