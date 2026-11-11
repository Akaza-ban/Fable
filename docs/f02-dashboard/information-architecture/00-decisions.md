# Decisions and Defaults — F02 Dashboard

> Design status: TARGET V1  
> Implementation status: NOT STARTED  

Information architecture IDs for the Dashboard subsystem carry the `IA-DB` prefix: decisions **D-IA-DB-01..D-IA-DB-10**, defaults **DF-IA-DB-01..DF-IA-DB-07**. These architectural proposals align with the system's business rules and user stories.

---

## Decisions Made in This Document Set

| ID | Decision | Rationale | Status |
|---|---|---|---|
| **D-IA-DB-01** | **Role-specific dedicated dashboard views for 3 roles:** `/dashboard` acts as a routing gateway, redirecting `ADMIN` to `/admin/dashboard` (Executive & Operations Control Center), `TECHNICIAN` to `/technician/dashboard` (Frontline Workbench), and `REQUESTER` to `/portal` (Self-Service Hub). | Caters directly to each user group: Leadership needs operational KPIs, engineers need today's active task queues, and requesters need self-service tracking and clarification resolution. | Decided |
| **D-IA-DB-02** | **One-Stop Self-Service Portal for Requesters (`SCR-DB-03`):** Requesters are routed to `/portal` to track personal requests, answer clarifying questions (`NEEDS_INFO`), confirm fixes (`RESOLVED`), search verified KB solutions, and read IT maintenance notices. Requesters are blocked only from internal IT telemetry (`/admin/dashboard` and `/technician/dashboard`). | Requesters represent ~90% of all system users. Empowering them with a dedicated self-service dashboard accelerates ticket progression and deflects routine inquiries. | Decided |
| **D-IA-DB-03** | **Standardized two-tier SLA breach warning indicators:**<br>• *Yellow Warning:* Remaining time $\le 20\%$ of SLA target duration.<br>• *Red Alert:* Overdue / breached SLA threshold. | Clear dual-threshold color coding enables team leads and engineers to intervene proactively before commitments are broken. | Decided |
| **D-IA-DB-04** | **Clear operational boundary between Dashboard (F02) and BI Reports (F07):** Dashboard F02 focuses exclusively on real-time operational data for today (Today / 24h) and active open tickets. Long-term trend analytics (weekly, monthly, quarterly) and 11 business intelligence metrics (M-01..M-11) belong to F07. | Eliminates duplication between real-time operational dashboards and analytical reporting. | Decided |
| **D-IA-DB-05** | **Hybrid telemetry streaming:** High-frequency event stream (P1 alerts, spike alerts, automation failure warnings) is pushed via WebSocket (`WS /api/v1/dashboard/live-feed`), while aggregated analytics use cached REST endpoints (`TTL 60s`). | Balances instant reactivity for mission-critical incidents with server resource conservation on heavy aggregation queries. | Decided |
| **D-IA-DB-06** | **Context-preserving Deep Drill-Down to F03 & F08:** Clicking any KPI card, chart slice, or SLA item preserves the active filter parameters and redirects to `/tickets?query=...` (F03) or links to F08 ticket timeline (`/tickets/:id#history`) and workflow run failure logs (`/audit/workflow-runs?status=FAILED`). | Eliminates redundant manual filtering when moving from analytical observation to tactical execution and forensic debugging. | Decided |
| **D-IA-DB-07** | **In-context action execution via Drawers & Modals:** Incident telemetry uses a right-hand slide-over drawer (`SCR-DB-10`); SLA interventions use focused modals (`SCR-DB-11`). | Maintains dashboard situational awareness without forcing full page transitions during high-stress triage. | Decided |
| **D-IA-DB-08** | **Explicit AI attribution styling:** All AI metrics (Autonomous Resolution Rate, spike detection alerts, KB deflection suggestions) are styled with distinct purple badges and labeled with text **AI**. | Preserves operational transparency so human supervisors easily distinguish deterministic data from AI-generated predictions. | Decided |
| **D-IA-DB-09** | **Redis metric caching with 60-second TTL:** Summary KPIs and intake charts are computed and cached in Redis with key structure `kpi:{date_range}:{group_id}:{role}`. | Prevents database table scans on active ticket tables during peak operational intake (8,000+ tickets/day). | Decided |
| **D-IA-DB-10** | **Persistent global incident banner for P1 issues:** Incoming P1 incidents display a persistent flashing red banner across all internal screens, dismissible only upon explicit acknowledgement (`ACK`). | Ensures critical infrastructure outages and high-severity incidents are not missed by on-duty personnel. | Decided |

---

## Defaults Pending Confirmation

| ID | Default | Status |
|---|---|---|
| **DF-IA-DB-01** | **Default time window:** "Today / Past 24 Hours". Long-term historical trends link directly to F07 Reports. | Decided |
| **DF-IA-DB-02** | **Auto-refresh polling fallback:** If the WebSocket connection drops, the dashboard automatically polls REST endpoints every 30 seconds. | Decided |
| **DF-IA-DB-03** | **Peer privacy rule:** Frontline technicians view their personal workload and unassigned team pool, but cannot browse individual ticket queues of peer engineers. | Decided |
| **DF-IA-DB-04** | **Audio notification chime:** Audio alert for P1 tickets is opt-in (default ON for on-duty engineers, toggleable in user preferences). | Decided |
| **DF-IA-DB-05** | **Number & date formatting:** Number separation with commas (`1,420`), percentages with 1 decimal place (`38.5%`), duration in minutes/hours (`42m`, `3h 15m`), timestamps in Vietnam Time (ICT, UTC+7). | Decided |
| **DF-IA-DB-06** | **Top 10 SLA display cap:** The SLA & Bottleneck radar limits visible items to the Top 10 most critical tickets to preserve readability; a "View All (N)" button links to the complete list. | Decided |
| **DF-IA-DB-07** | **One-click ticket claim:** Unassigned team tickets can be claimed directly from the workbench with status auto-transitioning to `IN_PROGRESS`. | Decided |

**Related:** [Sitemap](01-sitemap.md) · [Routes & Permissions](02-routes-and-permissions.md) · [Screen Inventory](04-screen-inventory.md)
