# Decisions and Defaults — F02 Dashboard

> Design status: V1 — draft, awaiting user review  
> Implementation status: NOT STARTED  

Information architecture IDs for the Dashboard subsystem carry the `IA-DB` prefix: decisions **D-IA-DB-01..D-IA-DB-10**, defaults **DF-IA-DB-01..DF-IA-DB-08**. These architectural proposals align with the system's business rules and user stories.

---

## Decisions Made in This Document Set

| ID | Decision | Rationale | Status |
|---|---|---|---|
| **D-IA-DB-01** | **Role-specific dedicated dashboard views:** `/dashboard` acts as a routing gateway, redirecting `ADMIN` to `/admin/dashboard` (Executive & Operations Control Center) and `TECHNICIAN`/`TECHNICIAN_LEAD` to `/technician/dashboard` (Frontline Workbench). | Executive management requires high-level KPI aggregation, whereas frontline engineers need immediate operational task queues. Splitting prevents visual clutter and permission leakage. | Proposed |
| **D-IA-DB-02** | **Requester role is strictly excluded from Dashboard (`403 Forbidden`):** Users with the `REQUESTER` role attempting to access `/dashboard` or `/admin/dashboard` are redirected to `/my-tickets`. | Requesters track only their own submitted requests; enterprise workload, SLA breach radars, and team utilization are internal IT operational telemetry. | Proposed |
| **D-IA-DB-03** | **Standardized two-tier SLA breach warning indicators:**<br>• *Yellow Warning:* Remaining time $\le 20\%$ of SLA target duration.<br>• *Red Alert:* Overdue / breached SLA threshold. | Clear dual-threshold color coding enables team leads to intervene proactively before commitments are broken. | Proposed |
| **D-IA-DB-04** | **Standardized Workload Capacity Index formula:**<br>$$\text{Workload Index} = \text{Active Open Tickets} + (\text{Count of P1/P2 Tickets} \times 2)$$<br>Categorized into: Normal (Green, $\le 4$), Busy (Yellow, $5 - 7$), Overloaded (Red, $> 7$ or $\ge 2$ P1/P2). | Weighting high-severity issues (P1/P2) prevents false-normal readings when an engineer holds fewer tickets that require intensive investigation. | Proposed |
| **D-IA-DB-05** | **Hybrid telemetry streaming:** High-frequency event stream (P1 alerts, SLA threshold triggers) is pushed via WebSocket (`WS /api/v1/dashboard/live-feed`), while aggregated analytics use cached REST endpoints (`TTL 60s`). | Balances instant reactivity for mission-critical incidents with server resource conservation on heavy aggregation queries. | Proposed |
| **D-IA-DB-06** | **Context-preserving Deep Drill-Down:** Clicking any KPI card, chart slice, or SLA item preserves the active filter parameters (date range, department, team) and redirects to `/tickets?query=...`. | Eliminates redundant manual filtering when moving from analytical observation to tactical execution. | Proposed |
| **D-IA-DB-07** | **In-context action execution via Drawers & Modals:** Incident telemetry uses a right-hand slide-over drawer (`SCR-DB-10`); reassignments use focused modals (`SCR-DB-11`). | Maintains dashboard situational awareness without forcing full page transitions during high-stress triage. | Proposed |
| **D-IA-DB-08** | **Explicit AI attribution styling:** All AI metrics (Autonomous Resolution Rate, AI triage suggestions, clustering alerts) are styled with distinct purple badges and labeled with text **AI**. | Preserves operational transparency so human supervisors easily distinguish deterministic data from AI-generated predictions. | Proposed |
| **D-IA-DB-09** | **Redis metric caching with 60-second TTL:** Summary KPIs and intake charts are computed and cached in Redis with key structure `kpi:{date_range}:{group_id}:{role}`. | Prevents database table scans on active ticket tables during peak operational intake (8,000+ tickets/day). | Proposed |
| **D-IA-DB-10** | **Persistent global incident banner for P1 issues:** Incoming P1 incidents display a persistent flashing red banner across all internal screens, dismissible only upon explicit acknowledgement (`ACK`). | Ensures critical infrastructure outages and high-severity incidents are not missed by on-duty personnel. | Proposed |

---

## Defaults Pending Confirmation

| ID | Default | Status |
|---|---|---|
| **DF-IA-DB-01** | **Default time window:** "Today / Past 24 Hours". The user may toggle to "Last 7 Days", "Last 30 Days", or "Custom Range". | Proposed |
| **DF-IA-DB-02** | **Overload thresholds:** Individual capacity maximum is 7 active tickets or 2 P1/P2 tickets. Team leads can override thresholds per team policy. | Proposed |
| **DF-IA-DB-03** | **Peer privacy rule:** Frontline technicians view their personal workload metric against the team's average, but cannot browse individual ticket queues of peer engineers. | Proposed |
| **DF-IA-DB-04** | **Auto-refresh polling fallback:** If the WebSocket connection drops, the dashboard automatically polls REST endpoints every 30 seconds. | Proposed |
| **DF-IA-DB-05** | **Audio notification chime:** Audio alert for P1 tickets is opt-in (default ON for on-duty engineers, toggleable in user preferences). | Proposed |
| **DF-IA-DB-06** | **Number & date formatting:** Number separation with commas (`1,420`), percentages with 1 decimal place (`38.5%`), duration in minutes/hours (`42m`, `3h 15m`), timestamps in Vietnam Time (ICT, UTC+7). | Proposed |
| **DF-IA-DB-07** | **Top 10 SLA display cap:** The SLA & Bottleneck radar limits visible items to the Top 10 most critical tickets to preserve readability; a "View All (N)" button links to the complete list. | Proposed |
| **DF-IA-DB-08** | **One-click ticket claim:** Unassigned team tickets can be claimed directly from the workbench with status auto-transitioning to `IN_PROGRESS`. | Proposed |

---

## Differences from or Additions to Root Requirements

| Topic | What Earlier Documents Said | What F02 Information Architecture Specifies and Why |
|---|---|---|
| Admin Home Page | Root IA temporarily routed Admin to `SCR-WF-30` (AI Decisions). | Replaces temporary route with `/admin/dashboard` (`SCR-DB-01`) as the official Executive Control Center. |
| Technician View | Earlier documents routed Technician to Ticket List `SCR-T-03`. | Routes Technician to `/technician/dashboard` (`SCR-DB-02`), providing a personal daily workbench, queue countdowns, and 1-click claim pool. |
| Workload Telemetry | General requirements mentioned workload monitoring. | Explicitly defines the mathematical Workload Index, capacity state thresholds (Green/Yellow/Red), and team balancing mechanics. |
| Real-time Alerts | Requirements described basic notifications. | Introduces dedicated WebSocket channel (`WS /live-feed`) and persistent emergency banner for P1 outages. |

**Related:** [Sitemap](01-sitemap.md) · [Routes and Permissions](02-routes-and-permissions.md) · [Screen Inventory](04-screen-inventory.md)

