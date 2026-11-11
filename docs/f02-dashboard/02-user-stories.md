# User Stories & Acceptance Criteria — Dashboard Subsystem (F02)

This document outlines the Agile requirement specifications for the **Dashboard** subsystem. Stories are grouped into strategic **Epics**, prioritized via the **MoSCoW** framework, and defined with **BDD (Given - When - Then)** Acceptance Criteria to support automated testing and product verification.

---

## Epic & Story Summary

| Epic | Story ID | User Story Title | Priority | Primary Persona |
|---|---|---|---|---|
| **EPIC 1: Real-Time Operational Visibility** | **US-DB-01** | Executive KPI Cards & Ticket Pipeline Breakdown Today | **Must Have** | System Administrator (`ADMIN`) |
| | **US-DB-02** | Today's AI Automation & Resolution Rate Analytics | **Should Have** | System Administrator (`ADMIN`) |
| **EPIC 2: SLA Governance & Rapid Intervention** | **US-DB-03** | SLA Breach Early Warning Radar & Active Bottlenecks | **Must Have** | System Administrator, Technician |
| | **US-DB-04** | Deep Metric Drill-Down to F03 Tickets & F08 Audit Logs | **Must Have** | All IT Internal Staff |
| **EPIC 3: Engineering Capacity Overview** | **US-DB-05** | Engineering Group Workload Capacity Overview | **Should Have** | System Administrator (`ADMIN`) |
| **EPIC 4: Real-Time Situational Awareness** | **US-DB-07** | Live Incident Stream, P1 Broadcast & Automation Failure Alert | **Must Have** | On-Duty Engineer, IT Admin |
| **EPIC 5: Technician Personal Workbench** | **US-DB-08** | Personalized Technician Daily Workbench & 1-Click Claim | **Must Have** | Support Technician (`TECHNICIAN`) |
| **EPIC 6: Requester Self-Service Hub** | **US-DB-09** | Requester Self-Service Portal & Request Hub | **Must Have** | Requester (`REQUESTER`) |

---

## EPIC 1: Real-Time Operational Visibility

### US-DB-01: Executive KPI Cards & Ticket Pipeline Breakdown Today
* **Story ID:** `US-DB-01` | **Priority:** `Must Have`
* **User Story:**
  > **As a** System Administrator,  
  > **I want to** view real-time KPI cards and priority distribution charts for today,  
  > **So that** I can grasp daily operational workload and throughput without waiting for periodic reports.

#### Acceptance Criteria (BDD):
* **Scenario 1: Default KPI Rendering for Today**
  * **Given:** The Administrator is authenticated and navigates to `/admin/dashboard`.
  * **When:** The dashboard page finishes loading.
  * **Then:** The system displays 5 core KPI metric cards: Total Ingested Today, New (`OPEN`), In Progress (`IN_PROGRESS`), Waiting for Customer (`NEEDS_INFO`), and Completed (`RESOLVED`/`CLOSED`) over the past 24 hours.
  * **And:** A priority breakdown chart is displayed: P1 (Red), P2 (Orange), P3 (Yellow), P4 (Blue).

---

### US-DB-02: Today's AI Automation & Resolution Rate Analytics
* **Story ID:** `US-DB-02` | **Priority:** `Should Have`
* **User Story:**
  > **As a** System Administrator,  
  > **I want to** track the percentage of tickets autonomously resolved by AI today,  
  > **So that** I can monitor daily automation efficiency and human workload reduction.

#### Acceptance Criteria:
* **Scenario 1: Autonomous Resolution Rate Calculation**
  * **Given:** 1,000 tickets are ingested during the day, with 350 resolved entirely by AI without human intervention.
  * **When:** The administrator views the "AI Automation Rate" card.
  * **Then:** The card displays `35.0%` with absolute count `350 / 1,000 tickets`.

---

## EPIC 2: SLA Governance & Rapid Intervention

### US-DB-03: SLA Breach Early Warning Radar & Active Bottlenecks
* **Story ID:** `US-DB-03` | **Priority:** `Must Have`
* **User Story:**
  > **As an** Administrator or Support Technician,  
  > **I want to** see a live list of active open tickets approaching or exceeding SLA thresholds,  
  > **So that** I can take immediate mitigation actions before service level agreements are breached.

#### Acceptance Criteria:
* **Scenario 1: Two-Tier SLA Warning Indication**
  * **Given:** Ticket #TK-981 has $\le 20\%$ time remaining and ticket #TK-980 is 15 minutes overdue.
  * **When:** The user inspects the SLA Watch widget.
  * **Then:** #TK-981 displays a Yellow Warning badge with live countdown; #TK-980 displays a Red Alert badge with `-15m overdue`.

---

### US-DB-04: Deep Metric Drill-Down to F03 Tickets & F08 Audit Logs
* **Story ID:** `US-DB-04` | **Priority:** `Must Have`
* **User Story:**
  > **As an** IT Internal Staff member,  
  > **I want to** click directly on any metric card, chart element, or alert to navigate to the underlying ticket list or audit log,  
  > **So that** I can inspect root causes immediately without re-entering filter criteria.

#### Acceptance Criteria:
* **Scenario 1: Drill-down to Ticket List**
  * **Given:** The KPI card indicates "8 Breached SLA Tickets".
  * **When:** The user clicks on the card.
  * **Then:** The browser navigates to `/tickets?sla_status=BREACHED&date=today` (F03).
* **Scenario 2: Drill-down to Technical Workflow Error Logs**
  * **Given:** An alert indicates an automated permission granting failure.
  * **When:** The user clicks "Inspect Failure Log".
  * **Then:** The browser navigates to `/audit/workflow-runs?status=FAILED` (F08).

---

## EPIC 3: Engineering Capacity Overview

### US-DB-05: Engineering Group Workload Capacity Overview
* **Story ID:** `US-DB-05` | **Priority:** `Should Have`
* **User Story:**
  > **As a** System Administrator,  
  > **I want to** monitor active open ticket volume across technical teams,  
  > **So that** I can detect department-level backlog bottlenecks and reallocate team shifts.

---

## EPIC 4: Real-Time Situational Awareness

### US-DB-07: Live Incident Stream, P1 Broadcast & Automation Failure Alert
* **Story ID:** `US-DB-07` | **Priority:** `Must Have`
* **User Story:**
  > **As an** On-Duty Engineer or Administrator,  
  > **I want** the Dashboard to automatically push critical P1 incidents and automation failures in real time via WebSocket,  
  > **So that** we achieve sub-second response times for major service outages.

---

## EPIC 5: Technician Personal Workbench

### US-DB-08: Personalized Technician Daily Workbench & 1-Click Claim
* **Story ID:** `US-DB-08` | **Priority:** `Must Have`
* **User Story:**
  > **As a** Support Technician,  
  > **I want** a dedicated home workspace displaying my prioritized daily tasks and an unassigned team pool with 1-click claim,  
  > **So that** I can maximize resolution throughput without distraction from company-wide metrics.

#### Acceptance Criteria:
* **Scenario 1: 1-Click Unassigned Ticket Claim**
  * **Given:** Ticket #TK-1011 is unassigned in the team pool.
  * **When:** The technician clicks [Claim Ticket].
  * **Then:** The ticket is assigned to the technician, transitioned to `IN_PROGRESS`, recorded in F08 audit logs, and moved to "My Queue Today" within 500ms.

---

## EPIC 6: Requester Self-Service Hub

### US-DB-09: Requester Self-Service Portal & Request Hub
* **Story ID:** `US-DB-09` | **Priority:** `Must Have`
* **User Story:**
  > **As a** Requester,  
  > **I want** a personal self-service dashboard highlighting actions awaiting my input, progress on open tickets, instant KB solution search, and IT broadcast announcements,  
  > **So that** I can resolve issues independently or unblock pending requests without contacting IT support.

#### Acceptance Criteria (BDD):
* **Scenario 1: Pinned Action-Needed Notifications**
  * **Given:** The requester has 1 ticket awaiting clarification (`NEEDS_INFO`) and 1 resolved ticket awaiting confirmation (`RESOLVED`).
  * **When:** The requester navigates to `/portal`.
  * **Then:** An "Action Needed" container is prominently displayed with [Answer Question] and [Confirm Resolution] action buttons.
* **Scenario 2: Instant Knowledge Base Deflection Search**
  * **Given:** The requester inputs a query into the portal search box.
  * **When:** Keystroke pauses for 300ms.
  * **Then:** Up to 3 relevant verified KB articles are rendered in a dropdown to deflect ticket submission.
* **Scenario 3: IT Operational Maintenance Notices**
  * **Given:** IT has scheduled system maintenance.
  * **When:** The requester views the portal.
  * **Then:** An informational announcement banner indicates maintenance schedules and affected services.
