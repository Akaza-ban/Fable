# Use cases — Reporting & Analytics, Activity Log

Roles, ticket visibility scope, permission matrix and `BR-xx` rules are reused from [mvp.md](mvp.md). Ticket statuses, SLA policy, AI decisions and workflow runs are reused from the Workflow module ([03-use-cases.md](03-use-cases.md), `BR-WF-xx` in [business-rules.md](requirements/business-rules.md)). This file only adds `UC-BC-xx`, `UC-NK-xx`, `BR-BC-xx`, `BR-NK-xx` and **proposes** new permissions (section 6). The proposed permissions take effect only once the owner of `mvp.md` accepts them.

These two modules only **read** data owned by other modules; they never redefine or change it:

| Data | Owner | Used for |
|---|---|---|
| Ticket statuses `NEW`, `NEEDS_INFO`, `TRIAGE`, `ASSIGNED`, `IN_PROGRESS`, `AUTOMATING`, `RESOLVED`, `CLOSED`, `CANCELLED` | Workflow / Ticket | Time metrics, history |
| SLA policy versions, `sla_breached`, `SLA_WARNING` / `SLA_BREACHED` events (BR-WF-07, BR-WF-08) | Workflow | SLA metrics |
| `ai_decisions` (outcome `AUTO_APPROVED` / `NEEDS_INFO` / `ROUTE_TO_HUMAN`, immutable, BR-WF-17) | Workflow | AI metrics, approval view |
| `workflow_runs`, `workflow_run_attempts` (payload already masked, BR-WF-15), `access_grants` | Workflow | Approval view, workflow errors |
| Internal notes, replies, assignment, escalation, satisfaction ratings | Ticket | History, satisfaction |
| KB suggestions and "helpful" ratings | Knowledge Base | KB metric |
| `AuditLog` (BR-09) | Shared | Every log view |

## 1. Overview

| Code | Use case | Actor |
|---|---|---|
| UC-BC-01 | Ticket statistics by type and group | ADMIN; TECHNICIAN (within scope) |
| UC-BC-02 | Track handling time and SLA breaches | ADMIN; TECHNICIAN (within scope) |
| UC-BC-03 | Evaluate the AI self-resolution rate | ADMIN |
| UC-BC-04 | Summarize requester satisfaction | ADMIN; TECHNICIAN (within scope) |
| UC-BC-05 | Filter and export reports | ADMIN |
| UC-BC-06 | Configure report parameters | ADMIN |
| UC-NK-01 | View ticket change history | ADMIN; TECHNICIAN, REQUESTER (within scope) |
| UC-NK-02 | Track actions by users and AI | ADMIN |
| UC-NK-03 | View approval requests and their outcome | ADMIN; TECHNICIAN, REQUESTER (within scope) |
| UC-NK-04 | Look up errors and workflow run history | ADMIN; TECHNICIAN (within scope) |
| UC-NK-05 | Write the log (append-only, sensitive data masked) | System |

---

## 2. Metric definitions

Reports, the Dashboard and the demo all use **the same calculation** below (BR-BC-01). Parameters marked "Admin-configured" are edited in UC-BC-06 unless another module owns them. Default values are **suggestions for the demo**, not commitments.

| Code | Metric | Calculation | Parameters |
|---|---|---|---|
| M-01 | First response time | From ticket creation to the **first public reply** to the requester by a TECHNICIAN, ADMIN or AI_AGENT. Automatic notifications (`actor_type = SYSTEM`) do not count. Human and AI first responses are reported separately. | Whether an AI reply meets the response SLA: Q-13 |
| M-02 | Resolution time | From creation to the **last** transition to `RESOLVED`, **minus** time in `NEEDS_INFO` (SLA paused, BR-WF-07) and minus time spent `RESOLVED` before the requester reported "not fixed". Measured on the working calendar of the ticket's SLA policy version (BR-WF-08), so M-02 and M-03 agree. | Working calendar: owned by Workflow (UC-WF-02) |
| M-03 | SLA breach | **Read from Workflow, not recomputed:** a ticket is breached when Workflow set `sla_breached = true` (`SLA_BREACHED` event). "At risk" = an `SLA_WARNING` event without a breach yet. Breach rate = breached tickets / tickets with an SLA policy in the period. | Targets and warning threshold: owned by Workflow (UC-WF-02; demo P1–P4 defaults, warning 75%) |
| M-04 | AI self-resolution rate | Numerator: tickets `CLOSED` in the period that **(a)** had no handling action by a TECHNICIAN or ADMIN (reply, assignment, status change, reclassification, internal note, retry or manual resolution of a workflow run) **and (b)** were not reopened within N days of closing. Denominator: tickets `CLOSED` in the period. **Excluded from both:** `CANCELLED` tickets, tickets auto-closed for no response (`NEEDS_INFO` → `CLOSED`, UC-WF-06), and tickets closed less than N days ago (shown separately as "under observation"). The requester's confirmation is not a handling action. | N: Admin-configured (demo: 7 days, the same as the Workflow reopen window, Q-19) |
| M-05 | AI classification correction rate | Tickets whose type or group was set by the AI (`actor_type = AI`) and **later** changed by a human (`actor_type = USER`) / tickets classified by the AI. Derived from the audit log (BR-09). | — |
| M-06 | Helpful KB suggestion rate | KB suggestions rated "helpful" / KB suggestions **with a rating**. Also: rating rate = suggestions with a rating / suggestions shown. Ratings come from the KB module. | Who can rate: Q-17 |
| M-07 | Satisfaction | Average score (1–5) of rated tickets, with the count at each score from 1 to 5. Response rate = rated tickets / tickets `CLOSED` in the period. The response rate is always shown next to the average. | — |
| M-08 | AI routing outcome | Share of AI decisions per outcome: `AUTO_APPROVED` (route A), `NEEDS_INFO` (route B), `ROUTE_TO_HUMAN` (route C), plus the reasons for route C. Read from `ai_decisions`; this is the data Workflow provides to reporting (NFR-WF-OBS-001). | — |
| M-09 | Escalation rate | Tickets escalated at least once (manual escalation in the Ticket module, or SLA escalation by Workflow, UC-WF-06) / tickets created in the period. The two sources are reported separately. | — |

**Common calculation rules:**
- **Count** statistics assign tickets to a period by **creation date**. **Time, SLA, M-04 and M-07** assign tickets by **resolution / close date**. `CANCELLED` tickets are counted in volume only.
- Grouping by priority uses the ticket's **final** priority; tickets whose priority was raised by SLA escalation are flagged (Q-15).
- A ticket moved across groups is attributed to the **last group** that handled it (Q-16).
- Every metric is computed only on tickets **within the viewer's scope** (BR-BC-02).

---

## 3. Reporting & Analytics

### UC-BC-01 Ticket statistics by type and group
- **Main flow:** choose a time range (default: last 30 days) → the backend loads the tickets within the viewer's scope → counts new, open, closed, cancelled, reopened and escalated tickets (M-09), grouped by ticket type and technician group → returns a table with a daily / weekly / monthly trend.
- **Alternative:** a TECHNICIAN only sees figures for tickets assigned to them and to their groups; the group filter only lists their groups. If their membership just changed, the new scope applies from the next request (BR-07). Tickets in `TRIAGE` are shown as their own group "Unassigned".
- **Exceptions:** no `report:read` → `403`. Filtering by a group outside scope → `403`. Invalid range (from > to, or longer than the maximum) → `422` `INVALID_DATE_RANGE`.
- **BR:** BR-07, BR-BC-01, BR-BC-02, BR-BC-03.

### UC-BC-02 Track handling time and SLA breaches
- **Main flow:** choose a period and filters → show M-01, M-02 (mean, median, P90) and M-03 by priority, ticket type and group → list of breached tickets (code, type, group, priority, SLA policy version, overdue by how long) → clicking a ticket opens its detail, checked against `ticket:read`.
- **Alternative:** list tickets **at risk** (M-03). Tickets without an SLA policy are counted separately as "no SLA". Tickets whose SLA policy version changed after a priority change are evaluated by Workflow against their current version (BR-WF-08); the report does not re-evaluate.
- **Exceptions:** no permission or outside scope → `403`. SLA data unavailable from Workflow → M-01 and M-02 are still shown; M-03 shows "unavailable" rather than 0%.
- **BR:** BR-WF-07, BR-WF-08, BR-BC-01, BR-BC-02, BR-BC-03.

### UC-BC-03 Evaluate the AI self-resolution rate
- **Main flow:** choose a period → show M-04 (with the "under observation" count), M-05, M-06 and M-08 by ticket type → lists of: tickets the AI closed that were reopened; tickets whose AI classification was corrected (AI value → human value, from the audit log `before` / `after`); route C reasons ranked by frequency.
- **Alternative:** filter by AI service account. Compare periods to see whether the AI improves after suggestions are approved (BR-15) or after Admin changes the AI scope (UC-WF-03).
- **Exceptions:** TECHNICIAN, REQUESTER, AI_AGENT → `403` (proposal: ADMIN only, Q-03). No ticket in the period has been closed for N days → "not enough data" rather than 0%.
- **BR:** BR-09, BR-15, BR-WF-17, BR-BC-01, BR-BC-03.

### UC-BC-04 Summarize requester satisfaction
- **Main flow:** choose a period → show M-07 (average, distribution from 1 to 5, response rate) by ticket type, group and handler (AI only / human) → list of tickets rated 1–2.
- **Alternative:** a TECHNICIAN sees ratings for tickets within their scope, but not other technicians' individual figures (Q-01, Q-02). Comments attached to ratings are visible to ADMIN only.
- **Exceptions:** response rate below the threshold (Admin-configured, demo 20%) → the score is shown with the warning "few ratings, not representative". No permission → `403`.
- **BR:** BR-BC-01, BR-BC-02, BR-BC-03.

### UC-BC-05 Filter and export reports
- **Main flow:** choose a report (UC-BC-01…04) and filters (period, ticket type, group, priority, handler AI or human, SLA status, AI routing outcome) → preview → Export as CSV or XLSX → the system **writes audit log entry `REPORT_EXPORTED`** → builds a file with only allowlisted columns → returns the file.
- **Alternative:** row count above the limit (Admin-configured, demo 50,000) → `422` `EXPORT_TOO_LARGE`, asking to narrow the filters. Filters are kept in the URL so the same report can be reopened.
- **Exceptions:** no `report:export` → `403`. Audit log write fails → **no file is returned**, `503` `AUDIT_UNAVAILABLE` (BR-BC-05).
- **BR:** BR-09, BR-BC-02, BR-BC-04, BR-BC-05.

### UC-BC-06 Configure report parameters
- **Main flow:** the Admin edits report-only parameters: N days for M-04, satisfaction response-rate threshold, export row limit, maximum time range → save → audit log entry `REPORT_CONFIG_UPDATED` with before and after values.
- **Not here:** SLA targets, warning threshold and working calendar are configured in Workflow (UC-WF-02); this screen links to it.
- **Alternative:** a change only affects **how figures are computed when viewed**; files already exported do not change. Each report shows the parameters in use (for example "N = 7 days").
- **Exceptions:** not an ADMIN → `403`. Invalid value (negative, N = 0, limit = 0) → `422`.
- **BR:** BR-09, BR-BC-01.

---

## 4. Activity Log

The activity log uses the audit log exactly as defined in **BR-09**: who (`actor_type` = `USER` / `AI` / `SYSTEM`), what (`action`), when, on which target, **before and after values**, plus `suggestion_id` when present (BR-15). Workflow records automated actions with source `AUTOMATION` (BR-WF-05); this file treats `AUTOMATION` as `SYSTEM` until the team picks one name (Q-06). Automated entries carry their evidence (`decisionId`, `ruleId`, `slaPolicyVersion` or `runId`, BR-WF-05), and the log shows it as links.

### UC-NK-01 View ticket change history
- **Main flow:** open a ticket (checked against `ticket:read`) → History tab → events in time order, newest first: created, AI classification and routing decision, questions to the requester (`NEEDS_INFO`), type / group / assignee / priority / status changes, escalation, replies, internal notes, workflow runs, SLA warnings and breaches, resolved, closed, reopened, cancelled, rated → each row shows the actor (person's name, "AI" or "System"), time, before / after values and evidence links.
- **Alternative — REQUESTER:** only sees events on the **allowlist** (BR-NK-04): created, status change, public reply, AI question and their answer, approval / execution outcome (summary), resolved, closed, reopened, cancelled, rated. Not shown: internal notes, assignment between technicians, AI confidence, policy check details, workflow attempts, SLA events. An AI actor is shown as "AI Assistant".
- **Alternative — TECHNICIAN:** sees everything for tickets within scope.
- **Exceptions:** ticket outside scope → `403` (same as `ticket:read`; not `404`, so the response does not reveal whether the ticket exists). Calling an API to edit / delete a history entry → no such endpoint, `405` (BR-NK-02).
- **BR:** BR-09, BR-WF-05, BR-NK-01, BR-NK-02, BR-NK-04.

### UC-NK-02 Track actions by users and AI
- **Main flow:** the Admin opens the Activity Log page → filters by `actor_type`, actor (user or service account), `action`, target type, target id, evidence id (`suggestion_id`, `decisionId`, `ruleId`, `runId`), time range → paginated table → opening a row shows `before` / `after` as a diff.
- **Alternative:** "View log" on a user, service account, assignment rule or SLA policy opens the log pre-filtered to it. `actor_type = AI` reviews everything the AI did; `actor_type = SYSTEM` shows automated changes (Microsoft sync BR-18 / BR-20, assignment rules, SLA timers, API execution). Disabled or removed users still appear with their names (BR-20 keeps the `User` record).
- **Exceptions:** no `audit:read` → `403`. Range longer than the maximum → `422` `INVALID_DATE_RANGE`.
- **BR:** BR-09, BR-15, BR-18, BR-20, BR-WF-05, BR-NK-02, BR-NK-03.

### UC-NK-03 View approval requests and their outcome
- **Main flow:** open the request from its ticket or from the list → show the chain in time order, each step with actor, time, result and evidence:
  1. **Request:** requester, recipient, operation (`CHECK_ACCOUNT` / `CREATE_ACCOUNT` / `GRANT_ACCESS` / `REVOKE_ACCESS`), target system, requested duration, source ticket.
  2. **AI understanding:** category, confidence, extracted fields (from `ai_decisions`).
  3. **Fixed policy check:** the result of **each** condition (whitelist, department entitlement, maximum duration, valid recipient BR-WF-16, not already granted), read from `ai_decisions` (BR-WF-02, BR-WF-17).
  4. **Decision:** `AUTO_APPROVED` ("approved by policy", `actor_type = AI`, with `decisionId`), `NEEDS_INFO` (question to requester), or `ROUTE_TO_HUMAN` with the reason. Routed requests are handled by the assigned technician; their actions appear as further steps.
  5. **Execution:** `workflow_run` status and every attempt (time, result code, duration, masked payload), idempotency key (BR-WF-04), policy re-check at run time (BR-WF-12).
  6. **Verification:** connector check result → `SUCCEEDED` or mismatch.
  7. **Requester confirmation**, and for time-limited grants the scheduled / executed `REVOKE_ACCESS` (BR-WF-14).
- **Alternative — approved but execution failed:** run `FAILED` (permanent error, retries exhausted, `PRECONDITION_FAILED`) or verification mismatch → shown highlighted as "Approved — execution failed", with the assigned technician. Retry adds **new steps**; earlier steps are never overwritten. "Resolved manually" shows the technician's mandatory note (BR-WF-13).
- **Alternative — REQUESTER:** only requests on their own tickets, as a summary: received / need more info / approved / being processed / done / failed, being handled by IT, with the short reason. No confidence, policy details, payload or attempts.
- **Exceptions:** request on a ticket outside scope → `403`. No `approval:read` → `403`.
- **BR:** BR-09, BR-WF-02, BR-WF-04, BR-WF-12, BR-WF-13, BR-WF-14, BR-WF-17, BR-NK-03, BR-NK-04.

### UC-NK-04 Look up errors and workflow run history
- **Main flow:** the Admin opens the list of workflow runs → filters by operation, status (`QUEUED`, `RUNNING`, `RETRY_WAIT`, `VERIFYING`, `SUCCEEDED`, `FAILED`, `MANUALLY_RESOLVED`), error type, ticket, connector, time range → opens a run: attempts with time, duration, result code, masked request / response (BR-WF-15, BR-NK-03), error message.
- **Error types shown:** `AI_UNAVAILABLE` (AI Service did not respond, UC-WF-04); `AI_INVALID_OUTPUT` (missing fields or wrong format, UC-WF-07 — name proposed here); transient connector errors (timeout, `5xx`, `429`, retried after 1 / 5 / 15 minutes); permanent connector errors (`4xx`, access denied); `PRECONDITION_FAILED` (BR-WF-12); verification mismatch; `AUDIT_UNAVAILABLE` (BR-NK-06).
- **Alternative — AI failure → manual handling:** on `AI_UNAVAILABLE` or `AI_INVALID_OUTPUT`, Workflow does **not apply** the AI result and routes the ticket to route C (UC-WF-05): assigned by rule, or `TRIAGE` if no rule matches. The log shows a `TICKET_ROUTED_TO_HUMAN` entry with the error type, also visible in UC-NK-01.
- **Alternative — TECHNICIAN:** the **assigned** technician sees the full attempt history of runs on their tickets (needed to retry, UC-WF-09). Other technicians in the group see a summary only: operation, status, error type, time, current assignee.
- **Exceptions:** no `workflow_log:read` → `403`. Run of a ticket outside scope → `403`.
- **BR:** BR-09, BR-WF-05, BR-WF-12, BR-WF-15, BR-NK-01, BR-NK-03, BR-NK-05, BR-NK-06.

### UC-NK-05 Write the log (System)
- **Main flow:** an action in the scope of BR-09 or BR-NK-01 happens → mask sensitive data (BR-NK-03) → write a new, **append-only** record (BR-NK-02). For changes in the system database, the record is written **in the same transaction** as the change (as in NFR-WF-AUD-001); if the log write fails, the change is rolled back.
- **Alternative — operations that change an external system** (`CREATE_ACCOUNT`, `GRANT_ACCESS`, `REVOKE_ACCESS`): write an "execution started" record **before** calling the connector. If that write fails → **the connector is not called**, the run is `FAILED` with `AUDIT_UNAVAILABLE`, and the ticket goes to a technician (BR-NK-06). If the call succeeds but writing the result fails → queue the record for retry and alert the Admin; the connector is not called again (idempotency, BR-WF-04).
- **Exceptions:** a sensitive field not recognized by name is still masked by pattern (BR-NK-03). Any edit / delete request → `405`.
- **BR:** BR-09, BR-13, BR-WF-04, BR-WF-15, BR-NK-02, BR-NK-03, BR-NK-06.

---

## 5. New business rules

**Reporting**
- **BR-BC-01**: All reports and the Dashboard use the definitions M-01…M-09 in section 2. Each report states its period, time zone (`Asia/Ho_Chi_Minh`) and the parameters in use.
- **BR-BC-02**: Figures are computed only on tickets the viewer can read under `ticket:read` (mvp.md section 4). Scope is filtered **in the backend query**, before aggregation. Aggregating over all tickets and hiding parts in the UI is not allowed.
- **BR-BC-03**: Reports are **read-only**. No report endpoint creates, edits or deletes tickets, audit logs or business data. SLA status is read from Workflow, not recomputed. Queries should use a read-only database account.
- **BR-BC-04**: Exported files contain only allowlisted columns: ticket code, type, group, priority, status, timestamps, metrics M-01…M-09. They **never** contain internal notes, passwords or hashes, tokens, API keys, 2FA secrets, workflow payloads or AI extracted fields. Cells starting with `=`, `+`, `-` or `@` are prefixed with `'` to prevent formula (CSV) injection.
- **BR-BC-05**: Every export writes audit log entry `REPORT_EXPORTED` (`actor_type = USER`; `after` = report, filters, format, row count). If the log write fails, no file is returned.

**Activity log**
- **BR-NK-01**: Extend the logging scope of BR-09, keeping the record structure, to: ticket changes (created, type, group, assignee, priority, status, escalation, reply, internal note, resolved, closed, reopened, cancelled, rated); AI decisions; every workflow run step; routing to humans after AI failure; report exports; report parameter changes. AI actions use `actor_type = AI`; automated actions use `actor_type = SYSTEM` (Workflow's `AUTOMATION`) and carry their evidence id (BR-WF-05).
- **BR-NK-02**: The log is **append-only**. Nobody can edit or delete it, **including ADMIN**: there is no edit / delete API (`405`), and the application's database account only has `INSERT` and `SELECT` on the audit log table. Corrections are new records that reference the old one.
- **BR-NK-03**: Before writing, all `before` / `after` values, workflow payloads and API responses are **masked**: fields whose name contains `password`, `token`, `secret`, `api_key`, `authorization`, `cookie`, `otp` → `"***"`; strings matching `Bearer …`, JWT or API key patterns → `"***"`. `password_hash`, `totp_secret_enc`, `api_key_hash` and connector credentials are never written (BR-05, BR-13, BR-WF-15).
- **BR-NK-04**: A REQUESTER only sees events on the **allowlist** (UC-NK-01, UC-NK-03), filtered in the backend. A new event type not yet classified is **hidden** from REQUESTER by default.
- **BR-NK-05**: When the AI does not respond or returns an invalid result, the AI result is not applied and the ticket is routed to a human (UC-WF-04, UC-WF-07); the log records `TICKET_ROUTED_TO_HUMAN` with the error type.
- **BR-NK-06**: For operations that change an external system, a successful "execution started" log write is a precondition. If it fails → **the connector is not called** and the permission is not granted.

---

## 6. Proposed additions to the permission matrix

Proposals for the owner of [mvp.md](mvp.md) to decide. This file does **not** edit `mvp.md`. Names follow the `resource:action` style of `mvp.md` (Q-22).

| Proposed permission | ADMIN | TECHNICIAN | REQUESTER | AI_AGENT | Used in |
|---|---|---|---|---|---|
| `report:read` | all | assigned, group (no other technicians' individual figures) | ❌ | ❌ | UC-BC-01, 02, 04 (UC-BC-03 ADMIN only) |
| `report:export` | ✅ | ❌ | ❌ | ❌ | UC-BC-05 |
| `report:configure` | ✅ | ❌ | ❌ | ❌ | UC-BC-06 |
| `ticket_history:read` | all | assigned, group | own (allowlist, BR-NK-04) | all (read-only, to learn from corrections, mvp.md section 8) | UC-NK-01 |
| `audit:read` | ✅ (system-wide) | ❌ | ❌ | ❌ | UC-NK-02 |
| `approval:read` | all | assigned, group (Q-12) | own (summary) | ❌ | UC-NK-03 |
| `workflow_log:read` | ✅ (detail) | assigned: detail; group: summary | ❌ | ❌ | UC-NK-04 |

No `audit:write`, `audit:update` or `audit:delete` permission is proposed for any role: only the backend writes the log (UC-NK-05), and nobody may edit or delete it (BR-NK-02).

---

## 7. Open questions (for the team to decide)

**Roles and actors**
- **Q-01**: Workflow already marks some technicians as **Team lead** (`is_lead`). Should a team lead see per-technician figures (SLA, satisfaction) for their own group? Proposal: yes, as a scope flag rather than a new role. This needs the owner of `mvp.md` to accept `is_lead`.
- **Q-02**: Can a technician who is not a lead see colleagues' individual figures, or only the group total and their own?
- **Q-03**: Is UC-BC-03 (AI effectiveness) for ADMIN only, or can team leads see it for their group?
- **Q-04**: The mindmap lists "Approve access requests", but Workflow V1 has **no human approver**: the AI approves after a fixed policy check, and anything outside scope goes to a technician. Is a human approver (manager, resource owner) needed later? UC-NK-03 can show that step without changes if it is added.
- **Q-05**: `mvp.md` defines `AI_AGENT` as a role that "may not grant permissions itself". Workflow defines "AI Service" as a system actor that **does** auto-approve grants within policy. The two owners need to agree on one wording; this file assumes AI auto-approval is allowed only within BR-WF-02.

**Logging**
- **Q-06**: Automated actions are `actor_type = SYSTEM` in BR-09 but `source = AUTOMATION` in BR-WF-05. Pick one name; this file uses `SYSTEM` and maps `AUTOMATION` to it.
- **Q-07**: Does the owner of `mvp.md` agree to extend BR-09 to tickets, AI decisions and workflows (BR-NK-01)? Should `AuditLog` get fields for the evidence ids (`ticket_id`, `decision_id`, `rule_id`, `sla_policy_version`, `run_id`), today only `suggestion_id` exists?
- **Q-08**: UC-03 logs "every AI request", including reads. Do reads go into `AuditLog` or a separate access log? At ~8,000 tickets/day (Workflow NFR), logging every read would make UC-NK-02 very large.
- **Q-09**: When `actor_type = AI`, does `actor_id` point to `ServiceAccount` or `User`?
- **Q-10**: How long is the audit log kept? Does "nobody can delete it" (BR-NK-02) need an exception for automatic deletion after retention?
- **Q-11**: Can a REQUESTER see the name of the technician handling their ticket in the history?
- **Q-12**: Workflow lets a technician see AI decisions only for tickets **assigned** to them; `mvp.md` scope is assigned **and group**. Which applies to `approval:read`?

**Metric calculation**
- **Q-13**: Does an AI reply count as the first response for the SLA (Workflow's SLA engine decides; M-01 reports both)?
- **Q-14**: Does the `SLA_BREACHED` event say whether the response or the resolution target was missed? Needed to split M-03.
- **Q-15**: When SLA escalation raises a ticket's priority, should reports group it under its initial or final priority? Proposal: final, with a flag.
- **Q-16**: A ticket moved across groups is attributed to the last group (proposal), the first, or each group that handled it?
- **Q-17**: Who can rate a KB suggestion as helpful (requester, technician, both)? Owned by the KB module.
- **Q-18**: At ~2.9 million tickets/year, live queries over a 12-month range will be slow. Proposal: live queries for the demo, daily pre-aggregated tables in production. Does the Dashboard share the same API?
- **Q-19**: Workflow lets a requester reopen an auto-closed ticket within 7 days. Is the reopen window the same for every `CLOSED` ticket? If yes, N in M-04 should equal it.

**Export and conventions**
- **Q-20**: Is PDF needed? Can satisfaction comments be exported (they may contain personal information)?
- **Q-21**: Is exporting the audit log (UC-NK-02) needed? If so, add `audit:export` and log the export.
- **Q-22**: Permission names differ between documents: `mvp.md` uses `user:read`, Workflow uses `workflowConfigManage`, `ticketReassign`. Which convention does the team keep?

See also: [mvp.md](mvp.md) · [use_cases.md](use_cases.md) · [03-use-cases.md](03-use-cases.md) · [user_stories_report_log.md](user_stories_report_log.md) · [information_architecture_report_log.md](information_architecture_report_log.md)
