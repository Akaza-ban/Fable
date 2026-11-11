# Screen Inventory — F02 Dashboard

> Design status: V1 — draft, awaiting user review  
> Implementation status: NOT STARTED  

The Dashboard subsystem inventory comprises **6 discrete UI surfaces: 2 primary full pages, 1 right-side drawer, and 3 contextual modals**.

---

## Screen Inventory Catalog

| Screen ID | Screen Name | Presentation Form | Permitted Roles | Opened From | Core Content & Interactive Controls | Related User Stories |
|---|---|---|---|---|---|---|
| `SCR-DB-01` | Executive Dashboard | Full Page | `ADMIN` (Global), `TECHNICIAN_LEAD` (Team view) | Sidebar (`/admin/dashboard`), Post-login redirect | • Global filter bar (date range, team dropdown, refresh, live status).<br>• Summary KPI cards (Total, Open, In Progress, Needs Info, Closed).<br>• AI Automation Rate card (percentage, trend indicator).<br>• Priority breakdown (P1-P4) & incident category charts.<br>• Hourly intake volume line chart.<br>• Top 10 SLA & Bottleneck watchlist with countdowns.<br>• Team workload matrix with capacity status indicators. | `US-DB-01`, `US-DB-02`, `US-DB-03`, `US-DB-04`, `US-DB-05` |
| `SCR-DB-02` | Technician Workbench | Full Page | `TECHNICIAN`, `TECHNICIAN_LEAD` | Sidebar (`/technician/dashboard`), Post-login redirect | • Personal KPI summary (assigned open tickets, due today, CSAT).<br>• "My Queue Today" table sorted by SLA urgency.<br>• "Unassigned Team Pool" with 1-click **Claim Ticket** buttons.<br>• Personal SLA countdown timers.<br>• Personal workload meter relative to team average. | `US-DB-08` |
| `SCR-DB-10` | Live Incident & P1 Drawer | Slide-over Drawer (Right) | `ADMIN`, `TECHNICIAN_LEAD`, on-duty `TECHNICIAN` | Flashing alert banner, Live Alert feed row | • Incident telemetry summary (title, severity, affected systems, start time).<br>• AI-generated incident summary and root cause hypothesis.<br>• List of grouped/correlated tickets.<br>• Quick actions: **Acknowledge (`ACK`)**, **Dispatch Team**, **Open War Room**. | `US-DB-07` |
| `SCR-DB-11` | SLA Escalation Modal | Modal | `ADMIN`, `TECHNICIAN_LEAD` | SLA Watch widget item, Breached badge click | • Breached/At-risk ticket overview (current assignee, time overdue).<br>• Team member selection dropdown with real-time capacity counters.<br>• Required escalation/reassignment reason input.<br>• Action buttons: **Send Nudge Notification**, **Reassign Now**. | `US-DB-03`, `US-DB-04` |
| `SCR-DB-12` | AI Workload Rebalance Modal | Modal | `ADMIN`, `TECHNICIAN_LEAD` | Overloaded engineer badge click, "Rebalance" button | • Overloaded engineer status (e.g., 9 active tickets, 2 P1s).<br>• AI proposed reallocation matrix: Move Ticket #TK-1082 to Engineer Bob (Load: 2/7).<br>• Confidence score and rebalancing rationale.<br>• Action buttons: **Apply Recommendation**, **Dismiss**. | `US-DB-05`, `US-DB-06` |
| `SCR-DB-13` | Custom Filter Modal | Modal | `ADMIN`, `TECHNICIAN_LEAD` | Date range dropdown $\rightarrow$ "Custom Range" | • Start date and end date pickers.<br>• Multi-select team selector.<br>• Priority filter checkboxes.<br>• Action buttons: **Apply Filters**, **Reset to Today**. | `US-DB-01` |

---

## Requirement-to-Screen Traceability Mapping

| User Story | Primary Screen / UI Surface | Action Path |
|---|---|---|
| **US-DB-01** (Executive KPI & Pipeline) | `SCR-DB-01` Executive Dashboard | Navigation $\rightarrow$ `/admin/dashboard` $\rightarrow$ inspect KPI cards and priority chart. |
| **US-DB-02** (AI Automation Analytics) | `SCR-DB-01` Executive Dashboard | View "AI Automation Rate" card $\rightarrow$ hover for breakdown tooltip. |
| **US-DB-03** (SLA Early Warning Radar) | `SCR-DB-01` $\rightarrow$ `SCR-DB-11` Modal | Review SLA Watch table $\rightarrow$ click breached item to open Escalation Modal. |
| **US-DB-04** (Deep Metric Drill-Down) | `SCR-DB-01` $\rightarrow$ `SCR-T-03` Ticket List | Click any metric card/chart slice $\rightarrow$ redirect to `/tickets?query=...`. |
| **US-DB-05** (Capacity Matrix & Overload) | `SCR-DB-01` Executive Dashboard | Inspect Workload Grid $\rightarrow$ identify red Overloaded badges. |
| **US-DB-06** (AI Workload Redistribution) | `SCR-DB-01` $\rightarrow$ `SCR-DB-12` Modal | Click "Rebalance Suggestion" $\rightarrow$ review AI proposal $\rightarrow$ Apply. |
| **US-DB-07** (Real-Time P1 Broadcast) | Persistent Banner $\rightarrow$ `SCR-DB-10` Drawer | P1 incident triggers banner $\rightarrow$ click banner $\rightarrow$ open drawer to ACK. |
| **US-DB-08** (Technician Workbench) | `SCR-DB-02` Technician Workbench | Navigation $\rightarrow$ `/technician/dashboard` $\rightarrow$ claim unassigned ticket. |

---

## Screen-to-API Mapping

Proposed REST and WebSocket operations powering the Dashboard UI surfaces:

| Screen / Component | API Operations | Protocols |
|---|---|---|
| `SCR-DB-01` Executive Dashboard | `getDashboardSummary`, `getSlaBreaches`, `getWorkloadMatrix`, `getLiveAlerts` | `GET /api/v1/dashboard/summary`<br>`GET /api/v1/dashboard/sla-breached`<br>`GET /api/v1/dashboard/workload`<br>`GET /api/v1/dashboard/alerts` |
| `SCR-DB-02` Technician Workbench | `getTechnicianQueue`, `claimTicket`, `getPersonalPerformance` | `GET /api/v1/dashboard/technician/my-queue`<br>`POST /api/v1/dashboard/tickets/:id/claim`<br>`GET /api/v1/dashboard/technician/metrics` |
| `SCR-DB-10` Live Incident Drawer | `getAlertDetail`, `acknowledgeAlert`, `dispatchIncidentTeam` | `GET /api/v1/dashboard/alerts/:id`<br>`POST /api/v1/dashboard/alerts/:id/ack`<br>`POST /api/v1/dashboard/alerts/:id/dispatch` |
| `SCR-DB-11` SLA Escalation Modal | `reassignTicket`, `sendSlaNudgeNotification` | `POST /api/v1/tickets/:id/reassign`<br>`POST /api/v1/tickets/:id/nudge` |
| `SCR-DB-12` AI Rebalance Modal | `getRebalanceSuggestions`, `applyRebalancePlan` | `GET /api/v1/dashboard/workload/suggestions`<br>`POST /api/v1/dashboard/workload/rebalance` |
| Live Telemetry Streaming | `dashboardEventStream` | `WS /api/v1/dashboard/live-feed` |

**Related:** [Navigation](03-navigation.md) · [User Flows](05-user-flows.md) · [UI States](06-ui-states-and-accessibility.md)

