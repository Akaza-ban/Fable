# Routes and Page Permissions — F02 Dashboard

> Design status: V1 — draft, awaiting user review  
> Implementation status: NOT STARTED  

## Routes & Query Parameters

Dashboard routes are logical application paths. Modals and drawers have no independent route; they mount over the active dashboard view while preserving context.

| Route | Screen / Component | Query Parameters & State Retention | Access Level |
|---|---|---|---|
| `/dashboard` | Routing Gateway | None (executes client/server redirect based on user role) | Authenticated |
| `/admin/dashboard` | `SCR-DB-01` Executive Dashboard | `?range=today` / `7d` / `30d` / `custom`<br>`?start_date=YYYY-MM-DD&end_date=YYYY-MM-DD`<br>`?group_id=<uuid>` (Filter by engineering team)<br>`?metric=<kpi_key>` | `ADMIN` only |
| `/technician/dashboard` | `SCR-DB-02` Technician Workbench | `?view=my_queue` / `unassigned`<br>`?sort=sla_asc` / `priority_desc` | `TECHNICIAN`, `TECHNICIAN_LEAD` |
| `/403` | `SCR-90` Access Denied | Standard system unauthorized error view | Public / Authenticated |

### Drill-Down Query String Protocol (D-IA-DB-06)
When an administrator or lead clicks any aggregate component on `/admin/dashboard`, the browser navigates to the ticket inventory screen (`SCR-T-03`) with preserved filter contexts:
* **Clicking "P1 Critical" chart slice:** $\rightarrow$ `/tickets?priority=P1&range=today&group_id=<active_group>`
* **Clicking "08 Breached SLA" card:** $\rightarrow$ `/tickets?sla_status=BREACHED&range=today`
* **Clicking "Unassigned Tickets" widget:** $\rightarrow$ `/tickets?status=OPEN&assignment=unassigned`

---

## Page Permission Matrix

| Screen / Component | Requester | Technician | Technician Lead | Admin | AI Agent |
|---|---|---|---|---|---|
| `/dashboard` (Gateway) | Redirect `/my-tickets` | Redirect `/technician/dashboard` | Redirect `/technician/dashboard` | Redirect `/admin/dashboard` | N/A |
| `SCR-DB-01` Executive Dashboard | — (403) | — (403) | View Only (Team Scope) | Full (Global Scope) | Read-only API |
| `SCR-DB-02` Technician Workbench | — (403) | Full (Self Scope) | Full (Self + Team Lead Scope) | Optional View | N/A |
| `SCR-DB-10` Live Incident Drawer | — (403) | Assigned Incidents | Team Incidents | All Incidents | Event Emitter |
| `SCR-DB-11` SLA Escalation Modal | — (403) | — (403) | Within Own Team | All Teams | N/A |
| `SCR-DB-12` AI Rebalance Modal | — (403) | — (403) | Within Own Team | All Teams | Proposal Generator |
| `SCR-90` Access Denied | ✓ | ✓ | ✓ | ✓ | N/A |

*Legend: `✓` = Full access; `—` = Forbidden (hidden from navigation; direct URL access triggers `SCR-90`).*

---

## Data Scope Enforcement Rules

1. **Server-Side Enforcement (BR-DB-01):** Authorization and query filtering are enforced at the database/API query level, never reliant on UI hiding:
   * `ADMIN`: Query conditions omit user/group scope filters (unrestricted enterprise data).
   * `TECHNICIAN_LEAD`: Enforces `WHERE group_id = current_user.led_group_id`.
   * `TECHNICIAN`: Enforces `WHERE assigned_to = current_user.id OR (group_id IN (current_user.groups) AND assigned_to IS NULL)`.
   * `REQUESTER`: Any request to `/api/v1/dashboard/*` terminates with HTTP `403 Forbidden`.
2. **Peer Privacy Guarantee (DF-IA-DB-03):** Technicians can view their personal workload gauge and compare it to their team's average workload score. They cannot inspect individual active ticket queues of other technicians.
3. **Session Expiry & Deactivation:** If a session token expires or an account is deactivated while the user is viewing the dashboard:
   * Next REST poll or WebSocket ping returns `401 Unauthorized`.
   * Session state is purged and the application redirects to `/login`.

**Related:** [Sitemap](01-sitemap.md) · [Navigation](03-navigation.md) · [Screen Inventory](04-screen-inventory.md)

