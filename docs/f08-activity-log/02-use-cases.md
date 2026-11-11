# Use cases — Activity Log

Roles, ticket visibility scope, permission matrix and `BR-xx` rules are reused from [mvp.md](../mvp.md). AI decisions and workflow runs are reused from the Workflow module ([03-use-cases.md](../f05-workflow-automation/02-use-cases.md), `BR-WF-xx` in [business-rules.md](../requirements/business-rules.md)). This file only adds `UC-NK-xx`, `BR-NK-xx` and **proposes** new permissions (section 4). The proposed permissions take effect only once the owner of `mvp.md` accepts them.

Related module: [Reporting & Analytics](../f07-reports-analytics/02-use-cases.md) (`UC-BC-xx`, `BR-BC-xx`) — report exports are written to this log.

This module only **reads** data owned by other modules; it never redefines or changes it:

| Data | Owner | Used for |
|---|---|---|
| Ticket statuses, replies, internal notes, assignment, escalation | Ticket / Workflow | Ticket history |
| `ai_decisions` (outcome `AUTO_APPROVED` / `NEEDS_INFO` / `ROUTE_TO_HUMAN`, immutable, BR-WF-17) | Workflow | Approval view |
| `workflow_runs`, `workflow_run_attempts` (payload already masked, BR-WF-15), `access_grants` | Workflow | Approval view, workflow errors |
| SLA `SLA_WARNING` / `SLA_BREACHED` events | Workflow | Ticket history |
| `AuditLog` (BR-09) | Shared | Every view |

## 1. Overview

| Code | Use case | Actor |
|---|---|---|
| UC-NK-01 | View ticket change history | ADMIN; TECHNICIAN, REQUESTER (within scope) |
| UC-NK-02 | Track actions by users and AI | ADMIN |
| UC-NK-03 | View approval requests and their outcome | ADMIN; TECHNICIAN, REQUESTER (within scope) |
| UC-NK-04 | Look up errors and workflow run history | ADMIN; TECHNICIAN (within scope) |
| UC-NK-05 | Write the log (append-only, sensitive data masked) | System |

The activity log uses the audit log exactly as defined in **BR-09**: who (`actor_type` = `USER` / `AI` / `SYSTEM`), what (`action`), when, on which target, **before and after values**, plus `suggestion_id` when present (BR-15). Workflow records automated actions with source `AUTOMATION` (BR-WF-05); this file treats `AUTOMATION` as `SYSTEM` until the team picks one name (Q-NK-03). Automated entries carry their evidence (`decisionId`, `ruleId`, `slaPolicyVersion` or `runId`, BR-WF-05), and the log shows it as links.

---

## 2. Use cases

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

## 3. New business rules

- **BR-NK-01**: Extend the logging scope of BR-09, keeping the record structure, to: ticket changes (created, type, group, assignee, priority, status, escalation, reply, internal note, resolved, closed, reopened, cancelled, rated); AI decisions; every workflow run step; routing to humans after AI failure; report exports and report parameter changes (BR-BC-05). AI actions use `actor_type = AI`; automated actions use `actor_type = SYSTEM` (Workflow's `AUTOMATION`) and carry their evidence id (BR-WF-05).
- **BR-NK-02**: The log is **append-only**. Nobody can edit or delete it, **including ADMIN**: there is no edit / delete API (`405`), and the application's database account only has `INSERT` and `SELECT` on the audit log table. Corrections are new records that reference the old one.
- **BR-NK-03**: Before writing, all `before` / `after` values, workflow payloads and API responses are **masked**: fields whose name contains `password`, `token`, `secret`, `api_key`, `authorization`, `cookie`, `otp` → `"***"`; strings matching `Bearer …`, JWT or API key patterns → `"***"`. `password_hash`, `totp_secret_enc`, `api_key_hash` and connector credentials are never written (BR-05, BR-13, BR-WF-15).
- **BR-NK-04**: A REQUESTER only sees events on the **allowlist** (UC-NK-01, UC-NK-03), filtered in the backend. A new event type not yet classified is **hidden** from REQUESTER by default.
- **BR-NK-05**: When the AI does not respond or returns an invalid result, the AI result is not applied and the ticket is routed to a human (UC-WF-04, UC-WF-07); the log records `TICKET_ROUTED_TO_HUMAN` with the error type.
- **BR-NK-06**: For operations that change an external system, a successful "execution started" log write is a precondition. If it fails → **the connector is not called** and the permission is not granted.

---

## 4. Proposed additions to the permission matrix

Proposals for the owner of [mvp.md](../mvp.md) to decide. This file does **not** edit `mvp.md`. Naming convention: see Q-BC-12 in [reporting-analytics](../f07-reports-analytics/02-use-cases.md).

| Proposed permission | ADMIN | TECHNICIAN | REQUESTER | AI_AGENT | Used in |
|---|---|---|---|---|---|
| `ticket_history:read` | all | assigned, group | own (allowlist, BR-NK-04) | all (read-only, to learn from corrections, mvp.md section 8) | UC-NK-01 |
| `audit:read` | ✅ (system-wide) | ❌ | ❌ | ❌ | UC-NK-02 |
| `approval:read` | all | assigned, group (Q-NK-09) | own (summary) | ❌ | UC-NK-03 |
| `workflow_log:read` | ✅ (detail) | assigned: detail; group: summary | ❌ | ❌ | UC-NK-04 |

No `audit:write`, `audit:update` or `audit:delete` permission is proposed for any role: only the backend writes the log (UC-NK-05), and nobody may edit or delete it (BR-NK-02).

---

## 5. Open questions (for the team to decide)

**Actors and approval**
- **Q-NK-01**: The mindmap lists "Approve access requests", but Workflow V1 has **no human approver**: the AI approves after a fixed policy check, and anything outside scope goes to a technician. Is a human approver (manager, resource owner) needed later? UC-NK-03 can show that step without changes if it is added.
- **Q-NK-02**: `mvp.md` defines `AI_AGENT` as a role that "may not grant permissions itself". Workflow defines "AI Service" as a system actor that **does** auto-approve grants within policy. The two owners need to agree on one wording; this file assumes AI auto-approval is allowed only within BR-WF-02.

**Logging**
- **Q-NK-03**: Automated actions are `actor_type = SYSTEM` in BR-09 but `source = AUTOMATION` in BR-WF-05. Pick one name; this file uses `SYSTEM` and maps `AUTOMATION` to it.
- **Q-NK-04**: Does the owner of `mvp.md` agree to extend BR-09 to tickets, AI decisions and workflows (BR-NK-01)? Should `AuditLog` get fields for the evidence ids (`ticket_id`, `decision_id`, `rule_id`, `sla_policy_version`, `run_id`), today only `suggestion_id` exists?
- **Q-NK-05**: UC-03 logs "every AI request", including reads. Do reads go into `AuditLog` or a separate access log? At ~8,000 tickets/day (Workflow NFR), logging every read would make UC-NK-02 very large.
- **Q-NK-06**: When `actor_type = AI`, does `actor_id` point to `ServiceAccount` or `User`?
- **Q-NK-07**: How long is the audit log kept? Does "nobody can delete it" (BR-NK-02) need an exception for automatic deletion after retention?

**Visibility**
- **Q-NK-08**: Can a REQUESTER see the name of the technician handling their ticket in the history?
- **Q-NK-09**: Workflow lets a technician see AI decisions only for tickets **assigned** to them; `mvp.md` scope is assigned **and group**. Which applies to `approval:read`?
- **Q-NK-10**: Is exporting the audit log (UC-NK-02) needed? If so, add `audit:export` and log the export.

See also: [01-user-stories.md](01-user-stories.md) · [03-information-architecture.md](03-information-architecture.md) · [Reporting & Analytics](../f07-reports-analytics/02-use-cases.md)
