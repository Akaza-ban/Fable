# Routes and Page Permissions — F02 Dashboard

> Design status: TARGET V1  
> Implementation status: NOT STARTED  

## Routes & Query Parameters

Dashboard routes are logical application paths. Modals and drawers have no independent route; they mount over the active dashboard view while preserving context.

| Route | Screen / Component | Query Parameters & State Retention | Access Level |
|---|---|---|---|
| `/dashboard` | Routing Gateway | None (executes client/server redirect based on user role) | Authenticated |
| `/admin/dashboard` | `SCR-DB-01` Executive Dashboard | `?date=today`<br>`?group_id=<uuid>` (Filter by engineering team)<br>`?metric=<kpi_key>` | `ADMIN` only |
| `/technician/dashboard` | `SCR-DB-02` Technician Workbench | `?view=my_queue` / `unassigned`<br>`?sort=sla_asc` / `priority_desc` | `TECHNICIAN` |
| `/portal` | `SCR-DB-03` Requester Portal | `?tab=overview` / `action_needed` | `REQUESTER` (All roles) |
| `/403` | `SCR-90` Access Denied | Standard system unauthorized error view | Public / Authenticated |

### Drill-Down Query String Protocol (D-IA-DB-06)
When an administrator or technician clicks any aggregate component on the Dashboard, the browser navigates to the target screen with preserved filter contexts:
* **Clicking "P1 Critical" chart slice:** $\rightarrow$ `/tickets?priority=P1&date=today&group_id=<active_group>` (F03)
* **Clicking "08 Breached SLA" card:** $\rightarrow$ `/tickets?sla_status=BREACHED&date=today` (F03)
* **Clicking "Audit Timeline" on a critical ticket:** $\rightarrow$ `/tickets/:id#history` (F08 `UC-NK-01`)
* **Clicking "Workflow Execution Failure" banner:** $\rightarrow$ `/audit/workflow-runs?status=FAILED` (F08 `UC-NK-04`)
* **Clicking "Long-Term Analytics" link:** $\rightarrow$ `/reports/volume` (F07 `UC-BC-01`)

---

## Page Permission Matrix

| Screen / Component | Requester | Technician | Admin | AI Agent |
|---|---|---|---|---|
| `/dashboard` (Gateway) | Redirect `/portal` | Redirect `/technician/dashboard` | Redirect `/admin/dashboard` | N/A |
| `SCR-DB-01` Executive Dashboard | — (403) | — (403) | Full (Global Scope) | Read-only API |
| `SCR-DB-02` Technician Workbench | — (403) | Full (Self + Team Queue) | Optional View | N/A |
| `SCR-DB-03` Requester Portal | Full (Self Scope) | Full Access | Full Access | Read-only API |
| `SCR-DB-10` Live Incident Drawer | — (403) | Assigned / Team Incidents | All Incidents | Event Emitter |
| `SCR-DB-11` SLA Escalation Modal | — (403) | Assigned / Team Tickets | All Teams | N/A |
| `SCR-90` Access Denied | ✓ | ✓ | ✓ | N/A |

*Legend: `✓` = Full access; `—` = Forbidden (hidden from navigation; direct URL access triggers `SCR-90`).*

---

## Data Scope Enforcement Rules

1. **Server-Side Enforcement (BR-DB-01):** Authorization and query filtering are enforced at the database/API query level, never reliant on UI hiding:
   * `ADMIN`: Query conditions omit user/group scope filters (unrestricted enterprise data).
   * `TECHNICIAN`: Enforces `WHERE assigned_to = current_user.id OR (group_id IN (current_user.groups) AND assigned_to IS NULL)`.
   * `REQUESTER`: Enforces `WHERE requester_id = current_user.id` for personal queries under `/api/v1/dashboard/requester/*`. Direct requests to `/api/v1/dashboard/admin/*` or `/api/v1/dashboard/technician/*` terminate with HTTP `403 Forbidden`.
2. **Session Expiry & Deactivation:** If a session token expires or an account is deactivated while the user is viewing the dashboard:
   * Next REST poll or WebSocket ping returns `401 Unauthorized`.
   * Session state is purged and the application redirects to `/login`.

**Related:** [Sitemap](01-sitemap.md) · [Navigation](03-navigation.md) · [Screen Inventory](04-screen-inventory.md)
