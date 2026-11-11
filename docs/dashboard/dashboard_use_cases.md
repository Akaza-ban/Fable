# Use Case Specification — Dashboard Subsystem

This document specifies the Use Cases for the **Dashboard** subsystem of the AI-Powered Automated Ticket Dispatching & Resolution System.  
Focus: Detailed interactions between **Actors** and the **System**, processing logic, preconditions, postconditions, alternative flows, and exception scenarios.

---

## 1. Actors & Use Case Catalog

### 1.1. Participating Actors
* **System Administrator (`ADMIN`):** Oversees end-to-end system operations, ticket allocation, engineering team workloads, and overall platform KPIs.
* **Technical Team Lead (`TECHNICIAN_LEAD`):** Monitors team performance, unassigned ticket queues, and workload capacity of team members.
* **Support Technician (`TECHNICIAN`):** Tracks personal assigned tickets, approaching SLA deadlines, and team queue status.
* **AI Service (`AI_AGENT`):** Serves as an automated system actor delivering classification, confidence scores, and workload rebalancing recommendations.

### 1.2. Use Case Catalog

| UC ID | Use Case Name | Primary Actor | Business Objective |
|---|---|---|---|
| **UC-DB-01** | Monitor Overall Operations & Ticket Distribution | `ADMIN` | Gain macro-level visibility into ticket intake, status progression, and AI resolution rates. |
| **UC-DB-02** | Track Bottlenecks & Manage SLA Breaches | `ADMIN`, `TECHNICIAN_LEAD` | Proactively identify at-risk and breached tickets to take timely mitigation actions. |
| **UC-DB-03** | Monitor & Rebalance Technician Workload | `ADMIN`, `TECHNICIAN_LEAD` | Assess real-time engineer capacity, detect workload bottlenecks, and redistribute tasks. |
| **UC-DB-04** | Real-Time Incident Alerting & Event Feed | `ADMIN`, `TECHNICIAN` | Receive immediate real-time notifications for critical P1 tickets, volume spikes, and workflow failures. |
| **UC-DB-05** | Technician Personal Workspace & Queue Operations | `TECHNICIAN` | Manage personal daily work queues, review personal SLA countdowns, and claim unassigned group tickets. |

---

## 2. Detailed Use Case Specifications

### UC-DB-01: Monitor Overall Operations & Ticket Distribution

* **Goal:** Provide System Administrators with a comprehensive overview of ticket volume, lifecycle distribution, and AI automation performance.
* **Preconditions:** The user is authenticated with the `ADMIN` role.
* **Postconditions:** Visual analytics and KPI metrics are rendered; deep drill-down navigation to `/tickets` is enabled.

#### Main Success Scenario (Main Flow):
1. **Actor:** Navigates to the `Dashboard` via the primary navigation sidebar.
2. **System:**
   - Validates user role and data scope (`ADMIN` -> global enterprise-wide scope).
   - Queries cached statistics for the default time window (Today / Past 24 Hours).
   - Renders the dashboard layout:
     - **Summary KPI Cards:** Total tickets received, New (`OPEN`), In Progress (`IN_PROGRESS`), Waiting for Customer (`NEEDS_INFO`), and Closed (`CLOSED`).
     - **AI Automation Rate:** Percentage and count of tickets resolved end-to-end by AI without manual engineer intervention.
     - **Distribution Charts:** Breakdown by priority (P1 Critical -> P4 Low) and incident category (Hardware, Network, Access & Permissions, Software, etc.).
     - **Hourly Trend Line Chart:** Ticket intake volume distributed across hourly intervals.
3. **Actor:** Modifies the time range filter (e.g., selects "Last 7 Days", "This Month", or "Custom Date Range").
4. **System:** Validates the date parameters and asynchronously reloads all KPI cards and charts.
5. **Actor:** Clicks on a specific chart element (e.g., the "P1 - Critical" slice on the breakdown chart).
6. **System:** Navigates to the Ticket Management screen (`/tickets`) with preset URL query parameters: `priority=P1&date_range=...`.

#### Alternative & Exception Flows:
* **A1 - Invalid Custom Date Range:** At step 3, if the start date is after the end date or the interval exceeds 365 days, the system displays an inline validation error: *"Invalid date range. Range cannot exceed 1 year."* and retains the current data view.
* **E1 - Aggregation Service Failure:** If the analytics database or cache is unreachable, the system displays an isolated error widget: *"Failed to load metric data. Please click to Retry."* without blocking the rest of the application.

---

### UC-DB-02: Track Bottlenecks & Manage SLA Breaches

* **Goal:** Locate stagnant tickets and identify tickets nearing or exceeding Service Level Agreement (SLA) response/resolution deadlines.
* **Preconditions:** The user is logged in as `ADMIN` or `TECHNICIAN_LEAD`.
* **Postconditions:** At-risk tickets are surfaced for immediate escalation, reminder dispatch, or re-assignment.

#### Main Success Scenario (Main Flow):
1. **Actor:** Views the **SLA & Bottleneck Watch** widget on the dashboard.
2. **System:**
   - Scans all active tickets and matches them against active SLA policies:
     - **Yellow Warning (Near Breach):** Remaining time $\le 20\%$ of the total SLA commitment.
     - **Red Alert (Breached):** Actual time exceeds initial response deadline or resolution deadline.
   - Displays a prioritized list of the Top 10 critical tickets sorted by priority descending (P1 $\rightarrow$ P4) and longest overdue time.
   - Renders real-time countdown timers or overdue badges beside each ticket.
3. **Actor:** Clicks "Quick View" on a breached ticket.
4. **System:** Opens a summary modal containing: AI-generated problem summary, current assigned technician, and recent activity logs.
5. **Actor:** Selects an action: "Send Reminder to Assignee" or "Reassign Ticket".
6. **System:** Dispatches the notification/reassignment, updates the ticket state, and logs the action in the `AuditLog`.

#### Alternative Flows:
* **A1 - 100% SLA Compliance:** If no tickets are nearing or exceeding deadlines, the widget displays a positive state card: *"All active tickets are currently meeting SLA targets."*

---

### UC-DB-03: Monitor & Rebalance Technician Workload

* **Goal:** Provide team leads and admins with visibility into individual engineer workload to prevent burnout and ensure balanced distribution.
* **Preconditions:** The user holds the `ADMIN` or `TECHNICIAN_LEAD` role.
* **Postconditions:** Overloaded personnel are flagged, enabling rapid manual or AI-assisted task rebalancing.

#### Main Success Scenario (Main Flow):
1. **Actor:** Accesses the **Technician Workload Monitor** widget.
2. **System:**
   - Retrieves active technicians within the actor's authorized scope.
   - Computes the workload index for each technician:
     $$\text{Workload Index} = \text{Active Open Tickets} + (\text{Count of P1/P2 Tickets} \times 2)$$
   - Categorizes capacity status:
     - **Normal (Green):** $\le 4$ active tickets.
     - **Busy (Yellow):** $5 - 7$ active tickets.
     - **Overloaded (Red):** $> 7$ active tickets OR $\ge 2$ unresolved P1/P2 tickets.
   - Displays comparative horizontal bar charts showing individual load versus team capacity.
3. **Actor:** Filters the view by a specific engineering team (e.g., "IT Support Team", "Cloud Infrastructure Team").
4. **System:** Updates the list and recalculates the team's average workload metric.
5. **Actor:** Clicks on an "Overloaded" engineer to inspect their ticket backlog.
6. **System:** Renders their active tickets along with an AI recommendation: *"Technician B currently has Low capacity (1 ticket). Suggest reassigning #TK-1082"*.

---

### UC-DB-04: Real-Time Incident Alerting & Event Feed

* **Goal:** Instantly notify support staff of severe operational incidents, volume surges, and automation failures as they happen.
* **Preconditions:** The dashboard page is open in the user's browser.
* **Postconditions:** High-priority incidents are acknowledged or acted upon with minimal latency.

#### Main Success Scenario (Main Flow):
1. **System:** Maintains an active WebSocket connection to the event streaming service.
2. **System:** Upon detection of an incoming high-severity event:
   - **New P1 Ticket Ingestion:** Triggers an optional audio chime and displays a persistent pulsing red banner at the top of the viewport with incident details.
   - **Ticket Surge / Spike Detection:** AI detects $> 5$ tickets matching identical keywords/categories within a 15-minute window (potential outage) $\rightarrow$ Displays a clustered outage alert banner.
   - **Automation / API Failure:** AI workflow execution fails (e.g., permission granting API timeout) $\rightarrow$ Pushes an urgent fallback intervention alert.
3. **Actor:** Clicks on the alert banner.
4. **System:** Slides open a detailed incident drawer containing context telemetry and quick-action buttons ("Acknowledge Alert", "Dispatch Dedicated Incident Team").

---

### UC-DB-05: Technician Personal Workspace & Queue Operations

* **Goal:** Provide frontline technicians with a focused personal cockpit displaying only tasks directly relevant to their duties.
* **Preconditions:** The user is authenticated with the `TECHNICIAN` role.
* **Postconditions:** The technician prioritizes and claims work items efficiently.

#### Main Success Scenario (Main Flow):
1. **Actor:** Logs in and is directed to `/technician/dashboard`.
2. **System:**
   - Applies the default scope: `assigned_to = me` OR `group_id IN (my_groups)`.
   - Renders personalized operational modules:
     - **My Queue Today:** Assigned tickets pending action, ordered by nearest SLA deadline.
     - **Personal SLA Watch:** Real-time countdowns for the actor's own tickets.
     - **Personal Productivity KPI:** Tickets resolved today/this week, customer satisfaction rating.
     - **Unassigned Team Queue:** Open group tickets awaiting an engineer.
3. **Actor:** Clicks "Claim Ticket" on an unassigned ticket belonging to their group.
4. **System:** Assigns the ticket to the actor, transitions status to `IN_PROGRESS`, and immediately refreshes the queue counter.
