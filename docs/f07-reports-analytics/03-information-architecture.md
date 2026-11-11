# Information architecture — Reporting & Analytics

Screens, role-based navigation, data and API of the module. Flows: [02-use-cases.md](02-use-cases.md); stories: [01-user-stories.md](01-user-stories.md); roles and `BR-xx`: [mvp.md](../mvp.md); Workflow data: [03-use-cases.md](../f05-workflow-automation/02-use-cases.md).

> Status: **draft for the team IA meeting.** Routes, menu placement and screens shared with other modules (Dashboard, Workflow) are proposals until the team merges all IA documents. Proposed permissions are in section 5 of the use case document.

## 1. Sitemap

```
/reports                          Reports                        (ADMIN; TECHNICIAN within scope)
  ├─ /reports/tickets             SCR-BC-01 Ticket statistics
  ├─ /reports/sla                 SCR-BC-02 SLA & handling time
  ├─ /reports/ai                  SCR-BC-03 AI effectiveness      (ADMIN)
  └─ /reports/satisfaction        SCR-BC-04 Satisfaction
/admin/report-settings            SCR-BC-05 Report settings       (ADMIN)
```

The four report screens are **tabs of one Reports page**: they share the filter bar, the export button and the URL query (`?from=…&to=…&type=…&group=…&priority=…`), so switching tabs keeps the filters.

## 2. Menu by role

| Menu item | ADMIN | TECHNICIAN | REQUESTER |
|---|---|---|---|
| Reports | ✅ all 4 tabs | ✅ Tickets, SLA, Satisfaction (own + group data) | ❌ |
| Report settings (under Settings) | ✅ | ❌ | ❌ |

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

**Empty and error states:** "Not enough data" (no ticket closed for N days), "Under observation", "SLA data unavailable", 403 page. A metric with no data is never shown as 0%.

## 4. Data

The module **owns almost no data**: it reads other modules' tables and the shared audit log.

**Owned (new)**

| Entity | Main fields |
|---|---|
| `ReportConfig` | key (`ai_resolution_window_days`, `csat_min_response_rate`, `export_row_limit`, `max_range_days`), value, updated_by, updated_at — every change is logged `REPORT_CONFIG_UPDATED` |
| `ReportDailyStat` *(production only, Q-BC-09)* | date, ticket_type, group_id, priority, counts and time sums per metric; rebuilt nightly from source data, never edited by hand |

**Read only (owned by other modules)**

| Entity | Owner | Read by |
|---|---|---|
| `tickets` (status, priority, type, group, timestamps, `sla_breached`) | Ticket / Workflow | All report screens |
| SLA policy versions, `SLA_WARNING` / `SLA_BREACHED` events | Workflow | SCR-BC-02 |
| `ai_decisions` | Workflow | SCR-BC-03 |
| `AuditLog` | Shared (BR-09) | M-04, M-05 |
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

- Invalid range → `422` `INVALID_DATE_RANGE`; export too large → `422` `EXPORT_TOO_LARGE`; audit log unavailable on export → `503` `AUDIT_UNAVAILABLE`.

## 6. Points to settle in the IA meeting

- **Dashboard vs Reports:** the Dashboard ([dashboard](../f02-dashboard/README.md)) shows open / overdue counts too. Proposal: the Dashboard calls the same `/reports/*` endpoints with a short default period, so the numbers always match (BR-BC-01).
- **Route and screen IDs:** `feature/author` uses plain paths (`/admin/users`), Workflow uses `SCR-WF-xx` IDs and hash routes (`#/403`). This file uses both (path + `SCR-BC-xx`) until the team chooses.
- **Team lead view:** if `is_lead` is accepted (Q-BC-01), leads get per-technician columns on SCR-BC-02 and SCR-BC-04 for their group.
