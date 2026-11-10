# User Stories & Acceptance Criteria — Dashboard Subsystem

This document outlines the Agile requirement specifications for the **Dashboard** subsystem. Stories are grouped into strategic **Epics**, prioritized via the **MoSCoW** framework, and defined with **BDD (Given - When - Then)** Acceptance Criteria to support automated testing and product verification.

---

## Epic & Story Summary

| Epic | Story ID | User Story Title | Priority | Primary Persona |
|---|---|---|---|---|
| **EPIC 1: Executive & Operational Visibility** | **US-DB-01** | Executive KPI Cards & Ticket Pipeline Breakdown | **Must Have** | IT Operations Manager |
| | **US-DB-02** | AI Automation & Resolution Rate Analytics | **Should Have** | IT Director, System Admin |
| **EPIC 2: SLA Governance & Bottleneck Mitigation** | **US-DB-03** | SLA Breach Early Warning Radar & Unassigned Backlog | **Must Have** | IT Manager, Team Lead |
| | **US-DB-04** | Deep Metric Drill-Down & Context Retention | **Must Have** | All IT Internal Staff |
| **EPIC 3: Workload Balancing & Resource Optimization** | **US-DB-05** | Engineer Capacity Matrix & Overload Warning | **Must Have** | Technical Team Lead |
| | **US-DB-06** | Intelligent Workload Redistribution Suggestions | **Could Have** | Technical Team Lead |
| **EPIC 4: Real-Time Situational Awareness & Fast Dispatch** | **US-DB-07** | Live Incident Stream & Critical P1 Broadcast | **Must Have** | On-Duty Engineer, IT Admin |
| | **US-DB-08** | Personalized Technician Daily Workbench | **Must Have** | Support Technician |

---

## EPIC 1: Executive & Operational Visibility

### US-DB-01: Executive KPI Cards & Ticket Pipeline Breakdown
* **Story ID:** `US-DB-01` | **Priority:** `Must Have`
* **User Story:**
  > **As an** IT Operations Manager,  
  > **I want to** view high-level KPI cards and priority distribution charts,  
  > **So that** I can grasp departmental workload and throughput at a glance without generating manual reports.

#### Acceptance Criteria (BDD):
* **Scenario 1: Default KPI Rendering**
  * **Given:** The IT Operations Manager is authenticated and navigates to the Dashboard.
  * **When:** The dashboard page finishes loading.
  * **Then:** The system displays 5 core KPI metric cards: Total Tickets, New (`OPEN`), In Progress (`IN_PROGRESS`), Pending Customer Input (`NEEDS_INFO`), and Completed (`RESOLVED`/`CLOSED`) over the past 24 hours.
  * **And:** A priority breakdown chart is displayed: P1 (Red), P2 (Orange), P3 (Yellow), P4 (Blue).

* **Scenario 2: Dynamic Time Range Filtering**
  * **Given:** The user is viewing data for "Today".
  * **When:** The user selects "Last 7 Days" from the date range dropdown.
  * **Then:** All KPI cards and charts refresh asynchronously with 7-day aggregated data without triggering a full page reload.

* **Scenario 3: Role-Based Access Enforcement**
  * **Given:** A user with the `REQUESTER` role attempts to access `/admin/dashboard`.
  * **When:** The HTTP request reaches the server.
  * **Then:** The server returns `403 Forbidden` and redirects the user to `/my-tickets`.

---

### US-DB-02: AI Automation & Resolution Rate Analytics
* **Story ID:** `US-DB-02` | **Priority:** `Should Have`
* **User Story:**
  > **As an** IT Director or Administrator,  
  > **I want to** track the percentage of tickets autonomously resolved by AI,  
  > **So that** I can measure ROI on AI investments and verify human workload offloading.

#### Acceptance Criteria:
* **Scenario 1: Autonomous Resolution Rate Calculation**
  * **Given:** 1,000 tickets are ingested during the day, with 350 resolved entirely by AI (e.g., automated permission grant, self-service password reset).
  * **When:** The administrator views the "AI Automation Rate" card.
  * **Then:** The card displays `35%` along with a period-over-period trend indicator (+X% vs last week).
  * **And:** Hovering over the card reveals a tooltip: *"350 automated resolutions / 650 routed to human engineers"*.

---

## EPIC 2: SLA Governance & Bottleneck Mitigation

### US-DB-03: SLA Breach Early Warning Radar & Unassigned Backlog
* **Story ID:** `US-DB-03` | **Priority:** `Must Have`
* **User Story:**
  > **As a** Technical Support Team Lead,  
  > **I want to** pinpoint unassigned tickets and tickets nearing or breaching SLA deadlines,  
  > **So that** I can intervene before service level commitments are compromised.

#### Acceptance Criteria:
* **Scenario 1: Two-Tier SLA Deadline Badges**
  * **Given:** Ticket #TK-992 has 25 minutes remaining before resolution deadline ($\le 20\%$ of total SLA duration).
  * **When:** The team lead reviews the SLA radar widget.
  * **Then:** The ticket displays a **Yellow Warning** badge with a live countdown `00:24:59`.
  * **And:** If the ticket exceeds the deadline, the badge turns **Red (Breached)** with the overdue duration `-00:15:20`.

* **Scenario 2: Unassigned Ticket Escalation**
  * **Given:** A newly created ticket has remained unassigned for over 30 minutes and AI could not auto-resolve it.
  * **When:** The periodic scan runs.
  * **Then:** The ticket is automatically highlighted in the "Requires Immediate Assignment" queue with an urgent indicator.

---

### US-DB-04: Deep Metric Drill-Down & Context Retention
* **Story ID:** `US-DB-04` | **Priority:** `Must Have`
* **User Story:**
  > **As a** Support Engineer or Administrator,  
  > **I want to** click directly on any metric card or chart section to navigate to the filtered ticket list,  
  > **So that** I can immediately work on those specific tickets without configuring manual search filters.

#### Acceptance Criteria:
* **Scenario 1: Drill-Down from Overdue SLA Card**
  * **Given:** The SLA card indicates `"08 Breached Tickets"`.
  * **When:** The user clicks on the card.
  * **Then:** The browser navigates to `/tickets?filter=sla_breached&scope=all`.
  * **And:** The table renders exactly the 8 corresponding breached tickets.

---

## EPIC 3: Workload Balancing & Resource Optimization

### US-DB-05: Engineer Capacity Matrix & Overload Warning
* **Story ID:** `US-DB-05` | **Priority:** `Must Have`
* **User Story:**
  > **As a** Technical Team Lead,  
  > **I want to** monitor the real-time active workload of each engineer on my team,  
  > **So that** I can identify bottlenecks and redistribute tasks before engineers become overwhelmed.

#### Acceptance Criteria:
* **Scenario 1: Threshold-Based Capacity Classification**
  * **Given:** Engineer Alice is actively working on 8 open tickets (`IN_PROGRESS`).
  * **When:** The team lead opens the "IT Support Workload" widget.
  * **Then:** Alice's progress bar is highlighted in Red with an `Overloaded (8/7)` indicator badge.
  * **And:** Engineer Bob with 2 open tickets is rendered in Green with a `Normal (2/7)` indicator badge.

* **Scenario 2: Peer Privacy Protection**
  * **Given:** A standard technician logs into the platform.
  * **When:** Viewing the workload section.
  * **Then:** They can only see their own workload metric compared against the team's average; individual ticket breakdowns of peers remain hidden.

---

## EPIC 4: Real-Time Situational Awareness & Fast Dispatch

### US-DB-07: Live Incident Stream & Critical P1 Broadcast
* **Story ID:** `US-DB-07` | **Priority:** `Must Have`
* **User Story:**
  > **As an** On-Duty Support Engineer,  
  > **I want to** receive instant real-time alerts when a critical P1 ticket is logged without refreshing the page,  
  > **So that** our team can respond immediately to severe outages (minimizing MTTR).

#### Acceptance Criteria:
* **Scenario 1: Audio & Visual Alert for P1 Tickets**
  * **Given:** The dashboard is open on the operations monitoring screen.
  * **When:** A new P1 (Critical) ticket is created in the system.
  * **Then:** A persistent red alert banner immediately renders at the top of the viewport, accompanied by an audible chime (if enabled).
  * **And:** The alert remains visible until an engineer clicks "Acknowledge Incident".

---

### US-DB-08: Personalized Technician Daily Workbench
* **Story ID:** `US-DB-08` | **Priority:** `Must Have`
* **User Story:**
  > **As a** Frontline Support Technician,  
  > **I want to** have a personalized workbench showing my assigned tasks and pending inquiries,  
  > **So that** I can focus exclusively on my daily duties without getting distracted by global enterprise statistics.

#### Acceptance Criteria:
* **Scenario 1: Priority-Sorted Personal Queue**
  * **Given:** A technician logs into the system.
  * **When:** Their default homepage loads.
  * **Then:** The "My Queue Today" list is automatically ordered by priority: P1 tickets first $\rightarrow$ Nearest SLA deadlines $\rightarrow$ Newly assigned items.
  * **And:** Quick-action buttons ("Start Working", "Request More Info") are accessible directly on each list item.
