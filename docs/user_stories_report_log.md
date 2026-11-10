# User stories — Reporting & Analytics, Activity Log

Each story has acceptance criteria (AC). Related use cases: [use_cases_report_log.md](use_cases_report_log.md). `BR-xx` rules: [mvp.md](mvp.md); `BR-WF-xx`: [business-rules.md](requirements/business-rules.md); `BR-BC-xx`, `BR-NK-xx`, metrics `M-xx` and proposed permissions: [use_cases_report_log.md](use_cases_report_log.md).

## Reporting & Analytics

**US-BC-01 — Ticket statistics by type and group** (UC-BC-01)
As an Admin, I want to see how many tickets come in, stay open and get closed per ticket type and group, so that I can spot overloaded groups and recurring problems.
- AC1: Choosing a time range shows new, open, closed, cancelled, reopened and escalated counts per type and per group, with a daily / weekly / monthly trend.
- AC2: Counts assign tickets to the period by creation date; `CANCELLED` tickets appear in volume only.
- AC3: Tickets in `TRIAGE` appear under "Unassigned".
- AC4: Range with from > to, or longer than the maximum → `422` `INVALID_DATE_RANGE`.

**US-BC-02 — Statistics limited to my scope** (UC-BC-01, UC-BC-02, UC-BC-04)
As a technician, I want to see figures for tickets assigned to me and to my groups, so that I can follow my team's workload without seeing other teams' data.
- AC1: A TECHNICIAN's figures are computed only on tickets assigned to them or their groups; the totals equal the count of tickets they can open.
- AC2: Filtering by a group they do not belong to → `403`.
- AC3: After being removed from a group, the next request no longer includes that group's tickets (BR-07).
- AC4: REQUESTER or AI_AGENT calling a report API → `403`.
- AC5: A TECHNICIAN does not see other technicians' individual figures.

**US-BC-03 — Handling time and SLA breaches** (UC-BC-02)
As an Admin, I want to see response and resolution times and which tickets breached their SLA, so that I can fix bottlenecks before they become a habit.
- AC1: The report shows M-01 and M-02 (mean, median, P90) and the M-03 breach rate by priority, type and group.
- AC2: Time spent in `NEEDS_INFO` is not counted in M-02 (BR-WF-07).
- AC3: A ticket counts as breached exactly when Workflow set `sla_breached = true`; the report and Workflow never disagree on the same ticket.
- AC4: The at-risk list shows open tickets with an `SLA_WARNING` and no breach yet.
- AC5: If SLA data is unavailable, M-03 shows "unavailable", not 0%.

**US-BC-04 — AI self-resolution rate** (UC-BC-03)
As an Admin, I want to know how many tickets the AI really solved on its own, so that I can decide whether to widen or narrow what the AI is allowed to do.
- AC1: A ticket closed with no TECHNICIAN or ADMIN handling action and not reopened within N days counts in the numerator of M-04.
- AC2: A ticket where a technician retried or manually resolved a workflow run does **not** count as AI-resolved.
- AC3: `CANCELLED` tickets, tickets auto-closed for no response, and tickets closed less than N days ago are excluded from both numerator and denominator; the last group is shown as "under observation".
- AC4: The report also shows M-05 (AI classification corrected), M-06 (helpful KB suggestions) and M-08 (route A / B / C share with route C reasons).
- AC5: Each report shows the N in use; changing N in UC-BC-06 changes the figure on the next view.
- AC6: TECHNICIAN calling this report → `403`.

**US-BC-05 — Requester satisfaction** (UC-BC-04)
As an Admin, I want to see satisfaction scores together with how many people actually rated, so that I do not draw conclusions from a handful of ratings.
- AC1: The report shows the 1–5 average, the count per score and the response rate (M-07) side by side.
- AC2: Response rate below the threshold → warning "few ratings, not representative".
- AC3: Tickets rated 1–2 are listed with a link to the ticket.
- AC4: Rating comments are visible to ADMIN only.

**US-BC-06 — Filter and export a report** (UC-BC-05)
As an Admin, I want to export the report I am looking at to CSV or XLSX, so that I can share it with management.
- AC1: The exported file contains the same rows and figures as the preview with the same filters.
- AC2: The file contains no internal notes, passwords, tokens, API keys, workflow payloads or AI extracted fields (BR-BC-04).
- AC3: A cell starting with `=`, `+`, `-` or `@` is written with a leading `'`.
- AC4: Every export creates an audit log entry `REPORT_EXPORTED` with the report, filters, format and row count.
- AC5: If the audit log cannot be written → `503` `AUDIT_UNAVAILABLE`, no file is returned.
- AC6: More rows than the limit → `422` `EXPORT_TOO_LARGE`.
- AC7: TECHNICIAN calling the export API → `403`.

**US-BC-07 — Configure report parameters** (UC-BC-06)
As an Admin, I want to set the observation window N and the report thresholds, so that the figures match how our organization works.
- AC1: Saving creates an audit log entry `REPORT_CONFIG_UPDATED` with before and after values.
- AC2: Invalid values (negative, N = 0, limit = 0) → `422`.
- AC3: The screen does not edit SLA targets; it links to the Workflow SLA configuration (UC-WF-02).
- AC4: Non-ADMIN → `403`.

## Activity Log

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
