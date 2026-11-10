# Information architecture — Reporting & Analytics, Activity Log

Screens, role-based navigation, data and API of the two modules. Flows: [use_cases_report_log.md](use_cases_report_log.md); stories: [user_stories_report_log.md](user_stories_report_log.md); roles and `BR-xx`: [mvp.md](mvp.md); Workflow data: [03-use-cases.md](03-use-cases.md).

> Status: **draft for the team IA meeting.** Routes, menu placement and screens shared with other modules (Ticket detail, Dashboard, Workflow) are proposals until the team merges all IA documents. Proposed permissions are the ones in section 6 of the use case document.

## 1. Sitemap

```
/reports                          Reports                        (ADMIN; TECHNICIAN within scope)
  ├─ /reports/tickets             SCR-BC-01 Ticket statistics
  ├─ /reports/sla                 SCR-BC-02 SLA & handling time
  ├─ /reports/ai                  SCR-BC-03 AI effectiveness      (ADMIN)
  └─ /reports/satisfaction        SCR-BC-04 Satisfaction
/admin/report-settings            SCR-BC-05 Report settings       (ADMIN)
/admin/activity-log               SCR-NK-02 Activity log          (ADMIN)
/admin/workflow-runs              SCR-NK-04 Workflow runs         (ADMIN; TECHNICIAN via ticket)
  └─ /admin/workflow-runs/:id     SCR-NK-05 Workflow run detail
/tickets/:id                      Ticket detail                   (owned by the Ticket module)
  ├─ tab History                  SCR-NK-01 Ticket history        (every role within scope)
  └─ panel Approval               SCR-NK-03 Approval chain        (when the ticket has an access request)
```

The four report screens are **tabs of one Reports page**: they share the filter bar, the export button and the URL query (`?from=…&to=…&type=…&group=…&priority=…`), so switching tabs keeps the filters.

## 2. Menu by role

| Menu item | ADMIN | TECHNICIAN | REQUESTER |
|---|---|---|---|
| Reports | ✅ all 4 tabs | ✅ Tickets, SLA, Satisfaction (own + group data) | ❌ |
| Activity log | ✅ | ❌ | ❌ |
| Workflow runs | ✅ | ❌ (opens runs from the ticket instead) | ❌ |
| Report settings (under Settings) | ✅ | ❌ | ❌ |
| Ticket → History tab | ✅ | ✅ (tickets in scope) | ✅ (own tickets, allowlist only) |
| Ticket → Approval panel | ✅ | ✅ (tickets in scope) | ✅ (own tickets, summary) |

AI_AGENT has no UI. Menu items are a convenience; the backend still checks every permission and scope (BR-BC-02). Opening a route without permission shows the shared 403 page.

## 3. Screen content

| Screen | Main components |
|---|---|
| **Shared filter bar** (SCR-BC-01…04) | Period (presets: 7 / 30 / 90 days, custom), ticket type, group (TECHNICIAN: only their groups), priority, handler (AI only / human), SLA status, AI routing outcome; "Parameters in use" line (time zone, N days); Export button (ADMIN) |
| SCR-BC-01 Ticket statistics | KPI tiles: new, open, closed, cancelled, reopened, escalated; trend chart by day / week / month; table by type × group; "Unassigned" row for `TRIAGE` |
| SCR-BC-02 SLA & handling time | Tiles: first response and resolution time (mean, median, P90), breach rate; breakdown by priority / type / group; tabs **Breached** and **At risk** (code, type, group, priority, SLA policy version, overdue by) with link to the ticket; note "SLA data from Workflow" |
| SCR-BC-03 AI effectiveness | Tiles: self-resolution rate with "under observation" count, classification correction rate, helpful KB rate; route A / B / C share; lists: AI-closed tickets reopened, corrected classifications (AI value → human value), top route C reasons; service account filter |
| SCR-BC-04 Satisfaction | Average score next to response rate; distribution 1–5; breakdown by type / group / handler; list of tickets rated 1–2; comments (ADMIN only); warning "few ratings, not representative" |
| Export dialog | Format CSV / XLSX, row count preview, note "internal notes and secrets are never exported"; errors `EXPORT_TOO_LARGE`, `AUDIT_UNAVAILABLE` |
| SCR-BC-05 Report settings | N days for AI self-resolution, satisfaction response-rate threshold, export row limit, maximum time range; link "SLA targets are set in Workflow" (UC-WF-02); last changed by / at |
| SCR-NK-01 Ticket history (tab) | Timeline newest first: icon by actor type (person / AI / system), action, before → after, time, evidence links (`decisionId`, `ruleId`, `runId`); filter by event group (status, assignment, replies, AI, workflow, SLA). REQUESTER: allowlisted events only, AI shown as "AI Assistant" |
| SCR-NK-02 Activity log | Filters: actor type, actor, action, target type, target id, evidence id, period; paginated table (time, actor, action, target, summary); side panel with `before` / `after` diff; deep links from user, service account, rule and SLA policy pages (`?actor=…`, `?target=…`) |
| SCR-NK-03 Approval chain (panel) | Vertical stepper: Request → AI understanding → Policy check (one row per condition, pass / fail) → Decision → Execution attempts → Verification → Requester confirmation → Revoke (if time-limited); banner "Approved — execution failed" with assignee. REQUESTER: one status line and short reason |
| SCR-NK-04 Workflow runs | Filters: status, error type, operation, connector, ticket, period; table (run id, ticket, operation, status, error type, attempts, started, duration); counters by status |
| SCR-NK-05 Workflow run detail | Run summary; attempts list (time, duration, result code, masked request / response); error message; link to ticket; actions Retry / Resolved manually belong to Workflow (UC-WF-09) and are shown only to the assigned technician or ADMIN |

**Empty and error states** (shared): "Not enough data" (no ticket closed for N days), "Under observation", "SLA data unavailable", "No events", 403 page. A metric with no data is never shown as 0%.

## 4. Data

These modules **own almost no data**: they read other modules' tables and the shared audit log.

**Owned (new)**

| Entity | Main fields |
|---|---|
| `ReportConfig` | key (`ai_resolution_window_days`, `csat_min_response_rate`, `export_row_limit`, `max_range_days`), value, updated_by, updated_at — every change is logged `REPORT_CONFIG_UPDATED` |
| `ReportDailyStat` *(production only, Q-18)* | date, ticket_type, group_id, priority, counts and time sums per metric; rebuilt nightly from source data, never edited by hand |

**Proposed extension of `AuditLog`** (needs the owner of `mvp.md`, Q-07): add `ticket_id`, `decision_id`, `rule_id`, `sla_policy_version`, `run_id` next to the existing `suggestion_id`, with indexes on (`target_type`, `target_id`, `created_at`), (`actor_type`, `created_at`) and `ticket_id`.

**Read only (owned by other modules)**

| Entity | Owner | Read by |
|---|---|---|
| `AuditLog` | Shared (BR-09) | SCR-NK-01, 02; M-04, M-05 |
| `tickets` (status, priority, type, group, timestamps, `sla_breached`) | Ticket / Workflow | All report screens |
| SLA policy versions, `SLA_WARNING` / `SLA_BREACHED` events | Workflow | SCR-BC-02 |
| `ai_decisions` | Workflow | SCR-BC-03, SCR-NK-03 |
| `workflow_runs`, `workflow_run_attempts`, `access_grants` | Workflow | SCR-NK-03, 04, 05 |
| Satisfaction ratings | Ticket | SCR-BC-04 |
| KB suggestions and ratings | Knowledge Base | SCR-BC-03 |
| `User`, `TechnicianGroup`, `GroupMember`, `ServiceAccount` | User management | Names, scope filtering |

## 5. Main API

Prefix `/api/v1`. Errors use `{ "code": "...", "message": "..." }`. Every endpoint filters by the caller's scope in the query (BR-BC-02).

| Method | Path | Permission |
|---|---|---|
| GET | `/reports/tickets?from=&to=&type=&group=&priority=&granularity=` | `report:read` |
| GET | `/reports/sla` | `report:read` |
| GET | `/reports/sla/tickets?state=breached\|at_risk` | `report:read` |
| GET | `/reports/ai` | `report:read` (ADMIN only) |
| GET | `/reports/satisfaction` | `report:read` |
| POST | `/reports/export` (body: report, filters, format) | `report:export` — writes `REPORT_EXPORTED` before returning the file |
| GET / PATCH | `/report-config` | `report:read` / `report:configure` |
| GET | `/tickets/:id/history` | `ticket_history:read` + `ticket:read` on the ticket |
| GET | `/tickets/:id/approvals` | `approval:read` + `ticket:read` on the ticket |
| GET | `/approvals?outcome=&status=&from=&to=` | `approval:read` (ADMIN: all) |
| GET | `/audit-logs?actor_type=&actor_id=&action=&target_type=&target_id=&evidence_id=&from=&to=` | `audit:read` |
| GET | `/audit-logs/:id` | `audit:read` |
| GET | `/workflow-runs?status=&error=&operation=&connector=&ticket_id=&from=&to=` | `workflow_log:read` |
| GET | `/workflow-runs/:id` | `workflow_log:read` (detail for ADMIN and the assigned technician, summary otherwise) |

- No `POST`, `PUT`, `PATCH` or `DELETE` exists on `/audit-logs` or `/tickets/:id/history`; such calls return `405` (BR-NK-02).
- Invalid range → `422` `INVALID_DATE_RANGE`; export too large → `422` `EXPORT_TOO_LARGE`; audit log unavailable on export → `503` `AUDIT_UNAVAILABLE`.
- `GET /workflow-runs/:id` overlaps with Workflow's `getWorkflowRun` (US-WF-20). Proposal: one endpoint, owned by Workflow; this module adds the list with filters.

## 6. Points to settle in the IA meeting

- **Dashboard vs Reports:** the Dashboard (module 4) shows open / overdue counts too. Proposal: the Dashboard calls the same `/reports/*` endpoints with a short default period, so the numbers always match (BR-BC-01).
- **Ticket detail page:** owned by the Ticket module. This module only adds the History tab and the Approval panel; Workflow adds its Automation tab. The three should be placed together in one design.
- **Workflow runs:** Workflow's ticket tab *Automation* (SCR-T-02) and SCR-NK-05 show the same run. Proposal: one component, reused in both places.
- **Route and screen IDs:** `feature/author` uses plain paths (`/admin/users`), Workflow uses `SCR-WF-xx` IDs and hash routes (`#/403`). This file uses both (path + `SCR-BC-xx` / `SCR-NK-xx`) until the team chooses.
- **Team lead view:** if `is_lead` is accepted (Q-01), leads get per-technician columns on SCR-BC-02 and SCR-BC-04 for their group.
