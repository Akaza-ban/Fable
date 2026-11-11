# Screen Inventory — F02 Dashboard

> Design status: TARGET V1  
> Implementation status: NOT STARTED  

The Dashboard subsystem inventory comprises **5 discrete UI surfaces: 3 primary full pages (for Admin, Technician, Requester), 1 right-side drawer, and 1 contextual modal**.

---

## Screen Inventory Catalog

| Screen ID | Screen Name | Presentation Form | Permitted Roles | Opened From | Core Content & Interactive Controls | Related User Stories |
|---|---|---|---|---|---|---|
| `SCR-DB-01` | Executive Dashboard | Full Page | `ADMIN` (Global scope) | Sidebar (`/admin/dashboard`), Post-login redirect | • Real-time operational summary cards (Ingested Today, Open, In Progress, Needs Info, Closed).<br>• AI Automation Rate card for today.<br>• Priority breakdown (P1-P4) & incident category charts.<br>• Hourly intake volume trend line chart.<br>• Top 10 SLA & Bottleneck watchlist with countdowns.<br>• Engineering team workload overview. | `US-DB-01`, `US-DB-02`, `US-DB-03`, `US-DB-04`, `US-DB-05` |
| `SCR-DB-02` | Technician Workbench | Full Page | `TECHNICIAN` | Sidebar (`/technician/dashboard`), Post-login redirect | • Personal KPI summary (assigned open tickets, due today, CSAT).<br>• "My Queue Today" table sorted by SLA urgency.<br>• "Unassigned Team Pool" with 1-click **[Claim Ticket]** buttons.<br>• Personal SLA countdown timers.<br>• Daily task throughput and productivity statistics. | `US-DB-08` |
| `SCR-DB-03` | Requester Portal | Full Page | `REQUESTER` (All roles) | Sidebar (`/portal`), Post-login redirect | • Action-needed section: Warning cards for tickets in `NEEDS_INFO` with [Answer Question], cards for `RESOLVED` tickets with [Confirm & Rate].<br>• My Requests Tracker: 5 recent personal tickets with visual milestone stages.<br>• Smart search bar with real-time suggested KB solutions (Deflection).<br>• 1-click service shortcuts (Access, Password Reset, Hardware, Software).<br>• IT broadcast announcements (maintenance, wide-area outages). | `US-DB-09` |
| `SCR-DB-10` | Live Incident & P1 Drawer | Slide-over Drawer (Right) | `ADMIN`, on-duty `TECHNICIAN` | Flashing alert banner, Live Alert feed row | • Incident telemetry summary (title, severity, affected systems, start time).<br>• AI-generated incident summary and root cause hypothesis.<br>• List of grouped/correlated tickets (Spike cluster).<br>• Quick actions: **Acknowledge (`ACK`)**, **Open War Room**, or **Inspect Technical Failure Logs in F08**. | `US-DB-07` |
| `SCR-DB-11` | SLA Escalation Modal | Modal | `ADMIN`, `TECHNICIAN` | SLA Watch widget item, Breached badge click | • Breached/At-risk ticket overview (current assignee, time overdue).<br>• Team member selection dropdown with real-time capacity counters.<br>• Required escalation/reassignment reason input.<br>• Action buttons: **Send Nudge Notification**, **Reassign Now**. | `US-DB-03`, `US-DB-04` |

---

## Requirement-to-Screen Traceability Mapping

| User Story | Primary Screen / UI Surface | Action Path |
|---|---|---|
| **US-DB-01** (Executive KPI & Pipeline Today) | `SCR-DB-01` Executive Dashboard | Navigation $\rightarrow$ `/admin/dashboard` $\rightarrow$ inspect KPI cards and priority chart. |
| **US-DB-02** (AI Automation Analytics Today) | `SCR-DB-01` Executive Dashboard | View "AI Automation Rate" card $\rightarrow$ hover for breakdown tooltip. |
| **US-DB-03** (SLA Early Warning Radar) | `SCR-DB-01` $\rightarrow$ `SCR-DB-11` Modal | Review SLA Watch table $\rightarrow$ click breached item to open Escalation Modal. |
| **US-DB-04** (Deep Metric Drill-Down) | `SCR-DB-01` $\rightarrow$ `SCR-T-03`, `SCR-NK-01`, `SCR-NK-04` | Click any metric card/chart slice $\rightarrow$ redirect to `/tickets?query=...` or F08 audit views. |
| **US-DB-05** (Engineering Group Workload Overview) | `SCR-DB-01` Executive Dashboard | Inspect Workload Grid $\rightarrow$ identify teams with high active ticket backlogs. |
| **US-DB-07** (Real-Time P1 & Automation Alert Broadcast) | Persistent Banner $\rightarrow$ `SCR-DB-10` Drawer | Critical incident triggers banner $\rightarrow$ click banner $\rightarrow$ open drawer to ACK or inspect F08 log. |
| **US-DB-08** (Technician Workbench) | `SCR-DB-02` Technician Workbench | Navigation $\rightarrow$ `/technician/dashboard` $\rightarrow$ claim unassigned ticket. |
| **US-DB-09** (Requester Self-Service Portal) | `SCR-DB-03` Requester Portal | Navigation $\rightarrow$ `/portal` $\rightarrow$ act on clarification requests and search KB solutions. |

---

## Screen-to-API Mapping

Proposed REST and WebSocket operations powering the Dashboard UI surfaces:

| Screen / Component | API Operations | Protocols |
|---|---|---|
| `SCR-DB-01` Executive Dashboard | `getDashboardSummary`, `getSlaBreaches`, `getWorkloadOverview`, `getLiveAlerts` | `GET /api/v1/dashboard/summary`<br>`GET /api/v1/dashboard/sla-breached`<br>`GET /api/v1/dashboard/workload`<br>`GET /api/v1/dashboard/alerts` |
| `SCR-DB-02` Technician Workbench | `getTechnicianQueue`, `claimTicket`, `getPersonalPerformance` | `GET /api/v1/dashboard/technician/my-queue`<br>`POST /api/v1/dashboard/tickets/:id/claim`<br>`GET /api/v1/dashboard/technician/metrics` |
| `SCR-DB-03` Requester Portal | `getRequesterActionNeeded`, `getMyRequests`, `getAnnouncements`, `kbSearch` | `GET /api/v1/dashboard/requester/action-needed`<br>`GET /api/v1/dashboard/requester/my-requests`<br>`GET /api/v1/dashboard/requester/announcements`<br>`GET /api/v1/kb/search` |
| `SCR-DB-10` Live Incident Drawer | `getAlertDetail`, `acknowledgeAlert`, `dispatchIncidentTeam` | `GET /api/v1/dashboard/alerts/:id`<br>`POST /api/v1/dashboard/alerts/:id/ack`<br>`POST /api/v1/dashboard/alerts/:id/dispatch` |
| `SCR-DB-11` SLA Escalation Modal | `reassignTicket`, `sendSlaNudgeNotification` | `POST /api/v1/tickets/:id/reassign`<br>`POST /api/v1/tickets/:id/nudge` |
| Live Telemetry Streaming | `dashboardEventStream` | `WS /api/v1/dashboard/live-feed` |

**Related:** [Navigation](03-navigation.md) · [User Flows](05-user-flows.md) · [UI States](06-ui-states-and-accessibility.md)
