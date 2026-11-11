# Use Case Specification — Dashboard Subsystem (F02)

This document specifies the Use Cases for the **Dashboard** subsystem of the AI-Powered Automated Ticket Dispatching & Resolution System.  
Focus: Detailed interactions between **Actors** and the **System**, processing logic, clear boundary separation from F07 (Reports & Analytics) and F08 (Activity Log), real-time operational monitoring, inclusion of the **Requester (`REQUESTER`)** self-service workspace, and alignment with the three standardized human roles (`ADMIN`, `TECHNICIAN`, `REQUESTER`).

---

## 1. Actors & Use Case Catalog

### 1.1. Participating Actors
* **System Administrator (`ADMIN`):** Oversees macro-level real-time operations, monitors today's ticket intake and AI automation rate, intervenes in active SLA bottlenecks, and tracks team capacity.
* **Support Technician (`TECHNICIAN`):** Operates on a personalized workbench, claims unassigned team tickets (1-click Claim), tracks individual SLA deadlines, and receives live critical incident alerts.
* **Requester (`REQUESTER`):** Tracks personal submitted tickets, acts on clarification requests (`NEEDS_INFO`), confirms resolution and rates CSAT (`RESOLVED`), searches knowledge base with instant deflection, and views IT maintenance announcements.
* **AI Service (`AI_AGENT`):** Automated background system providing spike detection, workflow execution failure alerts, and deflection matching.

### 1.2. Use Case Catalog

| UC ID | Use Case Name | Primary Actor | Business Objective | Boundary & Scope |
|---|---|---|---|---|
| **UC-DB-01** | Real-Time Operations Monitoring | `ADMIN` | Gain live visibility into today's ticket intake, status progression, and AI resolution rate. | Live operational data for today (24h); historical trends and long-term BI belong to F07 (`UC-BC-01`). |
| **UC-DB-02** | Live SLA Radar & Rapid Intervention | `ADMIN`, `TECHNICIAN` | Proactively identify active tickets at-risk or breached to dispatch immediate reminders or reassignments. | Active open tickets only; historical SLA compliance rates belong to F07 (`UC-BC-02`). |
| **UC-DB-03** | Real-Time Incident Alerting & Event Feed | `ADMIN`, `TECHNICIAN` | Receive immediate WebSocket notifications for critical P1 tickets, volume surges (spikes), and workflow failures. | Real-time live alerting; full immutable audit logging and API trace logs belong to F08 (`UC-NK-02`, `UC-NK-04`). |
| **UC-DB-04** | Technician Personal Workspace & Queue Operations | `TECHNICIAN` | Manage daily assigned tasks, review personal SLA countdowns, and claim unassigned group backlog tickets. | Daily engineer cockpit; conversation updates and ticket lifecycle transitions belong to F03. |
| **UC-DB-05** | Requester Self-Service Portal & Dashboard | `REQUESTER` | Track personal requests, answer clarifying questions, confirm resolutions, and search KB solutions. | One-stop self-service hub; ticket creation forms and deep KB authoring belong to F03 & F06. |

---

## 2. Detailed Use Case Specifications

### UC-DB-01: Real-Time Operations Monitoring

* **Goal:** Provide System Administrators with a comprehensive real-time view of ticket intake, lifecycle distribution, and AI automation performance for the current operating day (Today / Past 24 Hours).
* **Preconditions:** Authenticated with the `ADMIN` role.
* **Postconditions:** Visual telemetry is rendered; deep drill-down navigation to `/tickets` or F08 `/audit/activity-log` is enabled with preserved query parameters.

#### Main Success Scenario (Main Flow):
1. **Actor:** Navigates to `/admin/dashboard` via the primary navigation sidebar.
2. **System:**
   - Validates user role and data scope (`ADMIN` $\rightarrow$ global enterprise-wide scope).
   - Queries cached statistics for the current operating window (Today / Past 24 Hours).
   - Renders the dashboard layout:
     - **Summary KPI Cards:** Total tickets received today, New (`OPEN`), In Progress (`IN_PROGRESS`), Waiting for Customer (`NEEDS_INFO`), and Completed (`RESOLVED`/`CLOSED`).
     - **AI Automation Rate:** Percentage and count of tickets resolved end-to-end today by AI without human intervention.
     - **Distribution Charts:** Breakdown by priority (P1 Critical $\rightarrow$ P4 Low) and incident category.
     - **Hourly Intake Trend Line:** Intake volume distributed across hourly intervals to detect daily peak loads.
     - **Team Workload Overview:** Count of active open tickets per engineering group.
3. **Actor:** Clicks on a specific chart element or KPI card (e.g., "P1 - Critical" slice or "8 Breached Tickets").
4. **System:** Navigates to Ticket Management (`/tickets`) with preset query parameters: `priority=P1&date=today`.
5. **Actor:** Clicks the link "View Long-Term Analytical Reports".
6. **System:** Navigates to F07 Reports (`/reports/volume` - `UC-BC-01`).

#### Alternative & Exception Flows:
* **E1 - Aggregation Service Failure:** If the analytics cache is unreachable, the system displays an isolated error widget: *"Failed to load live metrics. Please click to Retry."* without blocking the page.

---

### UC-DB-02: Live SLA Radar & Rapid Intervention

* **Goal:** Locate active open tickets nearing or exceeding SLA response/resolution deadlines and enable rapid intervention.
* **Preconditions:** Authenticated as `ADMIN` or `TECHNICIAN`.
* **Postconditions:** Critical tickets are surfaced for immediate reminder dispatch or reassignment.

#### Main Success Scenario (Main Flow):
1. **Actor:** Views the **SLA & Bottleneck Watch** widget on the dashboard.
2. **System:**
   - Scans active open tickets against SLA policies (governed by F05):
     - **Yellow Warning (Near Breach):** Remaining time $\le 20\%$ of total SLA commitment.
     - **Red Alert (Breached):** Overdue response or resolution deadline (`sla_breached = true`).
   - Displays Top 10 critical active tickets sorted by priority descending (P1 $\rightarrow$ P4) and longest overdue time.
   - Renders real-time countdown timers or overdue badges.
3. **Actor:** Clicks "Quick View" on a breached ticket.
4. **System:** Opens summary modal `SCR-DB-11` displaying AI problem summary, current assigned engineer, and recent activity logs from F08.
5. **Actor:** Selects action: "Send Reminder" or "Reassign Ticket".
6. **System:** Updates ticket assignment, sends an instant notification, and writes an audit event `SLA_REASSIGNED` to F08 (`UC-NK-05`).

#### Alternative Flows:
* **A1 - 100% SLA Compliance:** If no tickets are nearing or exceeding deadlines, the widget displays: *"All active tickets currently meet SLA targets."*

---

### UC-DB-03: Real-Time Incident Alerting & Event Feed

* **Goal:** Instantly notify support staff of severe operational incidents, volume surges (spikes), and workflow automation failures with sub-second latency.
* **Preconditions:** Dashboard is open in the user's browser.
* **Postconditions:** High-priority incidents are acknowledged, dispatched, or investigated.

#### Main Success Scenario (Main Flow):
1. **System:** Maintains an active WebSocket connection (`WS /api/v1/dashboard/live-feed`) to the event streaming service.
2. **System:** Upon detecting an incoming critical event:
   - **P1 Ticket Ingestion:** Triggers optional chime and displays a persistent pulsing red banner across the top viewport with incident details and an [Acknowledge - ACK] button.
   - **Outage Surge / Spike Detection:** AI detects $> 5$ tickets matching identical keywords/categories within 15 minutes $\rightarrow$ Displays a clustered outage alert banner.
   - **Workflow Automation Failure:** AI auto-execution API fails after retries $\rightarrow$ Pushes an urgent intervention banner.
3. **Actor:** Clicks the alert banner.
4. **System:** Opens incident drawer `SCR-DB-10` containing telemetry, AI root-cause hypothesis, and quick-action buttons (Acknowledge, Dispatch Team, Open War Room).
5. **Actor:** Clicks "Inspect Failure Log" on an automation failure alert.
6. **System:** Navigates to F08 Technical Workflow Runs (`/audit/workflow-runs?status=FAILED` - `UC-NK-04`).

---

### UC-DB-04: Technician Personal Workspace & Queue Operations

* **Goal:** Provide frontline engineers with a focused personal cockpit displaying daily assigned tasks, personal SLA countdowns, and 1-click group ticket claiming.
* **Preconditions:** Authenticated with the `TECHNICIAN` role.
* **Postconditions:** Engineer prioritizes tasks and claims unassigned work smoothly.

#### Main Success Scenario (Main Flow):
1. **Actor:** Logs in and is routed to `/technician/dashboard` (`SCR-DB-02`).
2. **System:**
   - Applies default scope: `assigned_to = me` OR `group_id IN (my_groups)`.
   - Renders personalized operational modules:
     - **My Queue Today:** Assigned tickets pending action, ordered by nearest SLA deadline with live countdown timers.
     - **Unassigned Team Pool:** Open group tickets awaiting an engineer, with a **[Claim Ticket]** 1-click button.
     - **Personal Productivity KPI:** Tickets resolved today/this week, customer satisfaction rating (CSAT).
3. **Actor:** Clicks [Claim Ticket] on an unassigned ticket.
4. **System:** Assigns ticket to Actor, transitions status to `IN_PROGRESS`, emits an audit log event to F08, and immediately refreshes queue counters.
5. **Actor:** Clicks on a task in "My Queue Today".
6. **System:** Opens Ticket Details (`/tickets/:id` in F03).

---

### UC-DB-05: Requester Self-Service Portal & Dashboard

* **Goal:** Provide enterprise requesters with a one-stop self-service hub to track personal requests, act promptly on clarification requests (`NEEDS_INFO`), confirm completed fixes (`RESOLVED`), search knowledge base solutions with instant deflection, and view IT announcements.
* **Preconditions:** Authenticated with the `REQUESTER` role.
* **Postconditions:** Requester resolves inquiries faster or unblocks stagnant tickets awaiting their feedback.

#### Main Success Scenario (Main Flow):
1. **Actor:** Logs in and is directed to `/portal` (`SCR-DB-03`).
2. **System:**
   - Validates `REQUESTER` role and scopes queries to `requester_id = current_user.id`.
   - Renders 4 primary operational sections:
     - **Action Needed (Pinned at top):**
       - Yellow warning cards for tickets in `NEEDS_INFO` status with [Answer Question] button.
       - Green success cards for tickets in `RESOLVED` status with [Confirm & Rate] button.
     - **My Requests Tracker:**
       - Summary count of active open tickets and completed tickets this month.
       - Recent 5 tickets with visual progress indicators (Submitted $\rightarrow$ AI Classified $\rightarrow$ In Progress $\rightarrow$ Resolved).
     - **Smart Search & KB Deflection:**
       - Natural language search bar with real-time auto-suggested verified KB solutions (Deflection).
       - 1-click service shortcuts (Access Request, Password Reset, Software Bug, Equipment).
     - **IT Operational Announcements:**
       - Broadcast maintenance schedules and known wide-area outage notices.
3. **Actor:** Clicks [Answer Question] on a ticket requiring clarification.
4. **System:** Navigates directly to `/requests/:id#clarification`, highlighting the pending question from AI or support staff.

#### Alternative & Exception Flows:
* **A1 - Zero Open Requests:** The ticket tracker displays a friendly empty state card: *"You have no open IT requests"*, accompanied by a primary [Submit New Request] button.
* **A2 - Resolution Confirmation:** On a `RESOLVED` ticket card, Actor clicks [Confirm Resolution] $\rightarrow$ System displays CSAT star-rating modal (1–5 stars) $\rightarrow$ Updates ticket status to `CLOSED`.

---

**Related Specifications:** [Features](01-features.md) · [Acceptance Criteria](04-acceptance-criteria.md) · [User Stories](02-user-stories.md) · [Information Architecture](information-architecture/README.md)
