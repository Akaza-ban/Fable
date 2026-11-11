# Feature Catalog — Dashboard (F02)

> Design status: TARGET V1  
> Specification status: **Specified (this document set)**  

System name: **AI-Powered IT Ticket Routing and Automated Resolution System**.

## System Feature Catalog Overview

| ID | Name | V1 | Specification Status |
|---|---|---|---|
| F01 | Authentication & Authorization | Yes | Specified |
| **F02** | **Dashboard** | **Yes** | **Specified (this document set)** |
| F03 | Ticket Management | Yes | In progress |
| F04 | AI Ticket Processing | Yes | Specified |
| F05 | Workflow & Automation | Yes | Specified |
| F06 | Knowledge Base — KB | Yes | Not specified |
| F07 | Reports & Analytics | Yes | Not specified |
| F08 | Activity Log | Yes | Not specified |
| F09 | User Management | Yes | Specified |
| F10 | Personal Profile | Yes | Specified |

---

## F02 Dashboard

**Context:** The enterprise receives approximately **8,000 tickets per day** across global business units. Managing this volume requires real-time operational visibility for IT leadership, early warning systems to prevent SLA breaches, and instant broadcasts of critical P1 incidents. Frontline support engineers require an uncluttered workspace focused on today's tasks ordered by SLA urgency. Crucially, enterprise requesters need a dedicated self-service portal to track personal requests, promptly provide answers to AI/technician clarification questions, and find instant answers from verified knowledge base solutions.

**V1 Goal:** Provide a Real-Time Operations Control Center for Administrators (`ADMIN`), a personalized Workbench for Support Engineers (`TECHNICIAN`), and a dedicated Self-Service Portal for Requesters (`REQUESTER`), complete with deep metric drill-down capabilities to Ticket Management (F03) and Activity Logs (F08).

---

### Feature Breakdown & Role Access

| Capability Area | Description | Target Role | Handled By |
|---|---|---|---|
| **Real-Time Operations** | High-level KPI summary, today's AI automation rate, hourly intake trends, priority distribution | System Administrator (`ADMIN`) | `SCR-DB-01` Executive Dashboard |
| **SLA & Bottleneck Watch** | Proactive radar identifying active at-risk ($\le 20\%$ time left) and breached tickets; rapid reminder/reassign actions | Administrators, Technicians | `SCR-DB-01` $\rightarrow$ `SCR-DB-11` Modal |
| **Live Incident Broadcast** | Instant WebSocket streaming of critical P1 tickets, outage surge clusters (spikes), and automated workflow failures | On-Duty Engineers, Administrators | Top Viewport Banner $\rightarrow$ `SCR-DB-10` Drawer |
| **Technician Workbench** | Priority-ordered personal task queue, unassigned team pool with 1-click ticket claim, personal SLA countdowns | Frontline Support Technicians | `SCR-DB-02` Technician Workbench |
| **Requester Self-Service Portal** | Action-needed cards (`NEEDS_INFO`, confirm `RESOLVED`), personal request progress tracker, instant KB deflection search, IT notices | Enterprise Requesters (`REQUESTER`) | `SCR-DB-03` Requester Portal |

---

### Sub-Features within F02

* **F02.1 Executive Real-Time KPI Metrics (Today / 24h):**
  * Real-time aggregation cards: Total Ingested Today, New (`OPEN`), In Progress (`IN_PROGRESS`), Waiting for Customer (`NEEDS_INFO`), and Completed (`RESOLVED`/`CLOSED`).
  * Dedicated **AI Automation Rate Today** KPI card tracking percentage and count of tickets resolved autonomously without human intervention.
* **F02.2 Intake Trends & Distribution Analytics:**
  * Priority breakdown visualization (P1 Critical, P2 High, P3 Medium, P4 Low).
  * Category distribution breakdown (Hardware, Network, Access & Permissions, Software, Security).
  * Hourly intake volume trend line chart across the current operating day to identify peak demand.
* **F02.3 SLA Breach Early Warning Radar & Bottleneck Watch:**
  * Two-tier SLA status indicators: Yellow Warning ($\le 20\%$ time left) and Red Overdue Alert (Breached).
  * Prioritized Top 10 critical active tickets list with live countdown timers and 1-click escalation/reassignment modal.
* **F02.4 Engineering Group Workload Overview:**
  * Real-time count of active open tickets grouped by engineering team to assist Administrators in bottleneck detection.
* **F02.5 Real-Time Incident Stream & Emergency P1 Broadcast:**
  * WebSocket channel (`WS /api/v1/dashboard/live-feed`) broadcasting critical events with sub-second latency.
  * Flashing red viewport banner for P1 incidents with mandatory acknowledgement (`ACK`).
  * Spike detection algorithm flagging $> 5$ matching tickets within 15 minutes as potential widespread outages.
  * Automated workflow API execution failure alert linking directly to F08 technical run logs.
* **F02.6 Context-Preserving Deep Metric Drill-Down:**
  * 1-click navigation from any aggregate chart or card directly to pre-filtered queries on the Ticket List (`/tickets?query=...`) or Activity Log (`/audit/*`).
* **F02.7 Technician Personal Workbench & 1-Click Ticket Claim:**
  * Filtered view showing "My Queue Today" sorted by nearest SLA deadline.
  * Unassigned team pool with 1-click "Claim Ticket" action immediately updating status to `IN_PROGRESS`.
  * Individual daily/weekly throughput metrics and CSAT satisfaction ratings.
* **F02.8 Requester Self-Service Portal & Request Hub:**
  * Action-needed section: Yellow cards for tickets awaiting requester response (`NEEDS_INFO`) with [Answer Question] button; green cards for resolved tickets awaiting confirmation and rating.
  * Request progress tracker: 5 most recent personal tickets with visual milestone stages and SLA target times.
  * Smart semantic search bar with instant KB solution deflection.
  * 1-click service shortcuts for frequent requests (Access Grant, Password Reset, Hardware, Software).
  * IT operational broadcast announcements (maintenance, outages).

---

### Outside F02 (Owned by Other Features)

* **Raw ticket CRUD, conversation messaging, attachments:** $\rightarrow$ F03 Ticket Management.
* **AI NLP extraction, confidence modeling, ticket summarization algorithms:** $\rightarrow$ F04 AI Ticket Processing.
* **Automated routing rule evaluation, SLA business calendar calculation engines, workload assignment algorithms:** $\rightarrow$ F05 Workflow & Automation.
* **Long-term historical trend reports, executive quarterly BI export, 11 metric calculations (M-01..M-11), scheduled weekly email digest:** $\rightarrow$ **F07 Reports & Analytics**.
* **Full immutable system audit logs (SOC-2 compliant), technical workflow API run logs:** $\rightarrow$ **F08 Activity Log**.
* **User accounts, skills matrix, team membership provisioning:** $\rightarrow$ F09 User Management.

**Related:** [Use Cases](03-use-cases.md) · [Acceptance Criteria](04-acceptance-criteria.md) · [User Stories](02-user-stories.md) · [Information Architecture](information-architecture/README.md)
