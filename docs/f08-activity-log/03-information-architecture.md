# Information architecture — Activity Log

Screens, role-based navigation, data and API of the module. Flows: [02-use-cases.md](02-use-cases.md); stories: [01-user-stories.md](01-user-stories.md); roles and `BR-xx`: [mvp.md](../mvp.md); Workflow data: [03-use-cases.md](../03-use-cases.md).

> Status: **draft for the team IA meeting.** Routes, menu placement and screens shared with other modules (Ticket detail, Workflow) are proposals until the team merges all IA documents. Proposed permissions are in section 4 of the use case document.

## 1. Sitemap

```
/admin/activity-log               SCR-NK-02 Activity log          (ADMIN)
/admin/workflow-runs              SCR-NK-04 Workflow runs         (ADMIN; TECHNICIAN via ticket)
  └─ /admin/workflow-runs/:id     SCR-NK-05 Workflow run detail
/tickets/:id                      Ticket detail                   (owned by the Ticket module)
  ├─ tab History                  SCR-NK-01 Ticket history        (every role within scope)
  └─ panel Approval               SCR-NK-03 Approval chain        (when the ticket has an access request)
```

## 2. Menu by role

| Menu item | ADMIN | TECHNICIAN | REQUESTER |
|---|---|---|---|
| Activity log | ✅ | ❌ | ❌ |
| Workflow runs | ✅ | ❌ (opens runs from the ticket instead) | ❌ |
| Ticket → History tab | ✅ | ✅ (tickets in scope) | ✅ (own tickets, allowlist only) |
| Ticket → Approval panel | ✅ | ✅ (tickets in scope) | ✅ (own tickets, summary) |

AI_AGENT has no UI. Menu items are a convenience; the backend still checks every permission and scope. Opening a route without permission shows the shared 403 page.

## 3. Screen content

| Screen | Main components |
|---|---|
| SCR-NK-01 Ticket history (tab) | Timeline newest first: icon by actor type (person / AI / system), action, before → after, time, evidence links (`decisionId`, `ruleId`, `runId`); filter by event group (status, assignment, replies, AI, workflow, SLA). REQUESTER: allowlisted events only, AI shown as "AI Assistant" |
| SCR-NK-02 Activity log | Filters: actor type, actor, action, target type, target id, evidence id, period; paginated table (time, actor, action, target, summary); side panel with `before` / `after` diff; deep links from user, service account, rule and SLA policy pages (`?actor=…`, `?target=…`) |
| SCR-NK-03 Approval chain (panel) | Vertical stepper: Request → AI understanding → Policy check (one row per condition, pass / fail) → Decision → Execution attempts → Verification → Requester confirmation → Revoke (if time-limited); banner "Approved — execution failed" with assignee. REQUESTER: one status line and short reason |
| SCR-NK-04 Workflow runs | Filters: status, error type, operation, connector, ticket, period; table (run id, ticket, operation, status, error type, attempts, started, duration); counters by status |
| SCR-NK-05 Workflow run detail | Run summary; attempts list (time, duration, result code, masked request / response); error message; link to ticket; actions Retry / Resolved manually belong to Workflow (UC-WF-09) and are shown only to the assigned technician or ADMIN |

**Empty and error states:** "No events", 403 page.

## 4. Data

The module **owns no new entity**: it reads the shared audit log and Workflow tables.

**Proposed extension of `AuditLog`** (needs the owner of `mvp.md`, Q-NK-04): add `ticket_id`, `decision_id`, `rule_id`, `sla_policy_version`, `run_id` next to the existing `suggestion_id`, with indexes on (`target_type`, `target_id`, `created_at`), (`actor_type`, `created_at`) and `ticket_id`. The application's database account has only `INSERT` and `SELECT` on this table (BR-NK-02).

**Read only (owned by other modules)**

| Entity | Owner | Read by |
|---|---|---|
| `AuditLog` | Shared (BR-09) | SCR-NK-01, 02 |
| `tickets`, replies, internal notes | Ticket | SCR-NK-01 |
| `ai_decisions` | Workflow | SCR-NK-03 |
| `workflow_runs`, `workflow_run_attempts`, `access_grants` | Workflow | SCR-NK-03, 04, 05 |
| `User`, `ServiceAccount` | User management | Actor names |

## 5. Main API

Prefix `/api/v1`. Errors use `{ "code": "...", "message": "..." }`. Every endpoint filters by the caller's scope in the query.

| Method | Path | Permission |
|---|---|---|
| GET | `/tickets/:id/history` | `ticket_history:read` + `ticket:read` on the ticket |
| GET | `/tickets/:id/approvals` | `approval:read` + `ticket:read` on the ticket |
| GET | `/approvals?outcome=&status=&from=&to=` | `approval:read` (ADMIN: all) |
| GET | `/audit-logs?actor_type=&actor_id=&action=&target_type=&target_id=&evidence_id=&from=&to=` | `audit:read` |
| GET | `/audit-logs/:id` | `audit:read` |
| GET | `/workflow-runs?status=&error=&operation=&connector=&ticket_id=&from=&to=` | `workflow_log:read` |
| GET | `/workflow-runs/:id` | `workflow_log:read` (detail for ADMIN and the assigned technician, summary otherwise) |

- No `POST`, `PUT`, `PATCH` or `DELETE` exists on `/audit-logs` or `/tickets/:id/history`; such calls return `405` (BR-NK-02).
- Invalid range → `422` `INVALID_DATE_RANGE`.
- `GET /workflow-runs/:id` overlaps with Workflow's `getWorkflowRun` (US-WF-20). Proposal: one endpoint, owned by Workflow; this module adds the list with filters.

## 6. Points to settle in the IA meeting

- **Ticket detail page:** owned by the Ticket module ([ticket-management](../ticket-management/)). This module only adds the History tab and the Approval panel; Workflow adds its Automation tab. The three should be placed together in one design.
- **Workflow runs:** Workflow's ticket tab *Automation* (SCR-T-02) and SCR-NK-05 show the same run. Proposal: one component, reused in both places.
- **Route and screen IDs:** this file uses path + `SCR-NK-xx` until the team chooses a convention (see the same point in [reporting-analytics](../reporting-analytics/03-information-architecture.md)).
