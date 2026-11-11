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

**Context:** The enterprise receives approximately **8,000 tickets per day** across global business units. Managing this volume requires real-time operational visibility for IT leadership, early warning systems to prevent SLA breaches, and balanced task distribution to avoid technician burnout. Frontline engineers require an uncluttered workspace focused on immediate tasks ordered by SLA urgency.

**V1 Goal:** Provide an Executive & Operations Control Center for IT Leadership (`ADMIN`, `TECHNICIAN_LEAD`) and a focused, real-time Personal Workbench for Support Engineers (`TECHNICIAN`), complete with deep metric drill-down capabilities, automated workload capacity tracking, and instant P1 incident broadcasts.

---

### Feature Breakdown & Role Access

| Capability Area | Description | Target Role | Handled By |
|---|---|---|---|
| **Executive Operations** | High-level KPI summary, AI automation rate, ticket intake volume trends, priority breakdown | IT Director, System Administrator | `SCR-DB-01` Executive Dashboard |
| **SLA & Bottleneck Watch** | Proactive radar identifying at-risk ($\le 20\%$ time left) and breached tickets; instant reassignment controls | Team Leads, Administrators | `SCR-DB-01` $\rightarrow$ `SCR-DB-11` Modal |
| **Workload Capacity Balancing** | Real-time capacity index tracking ($\text{Open} + \text{P1/P2} \times 2$), overload detection ($> 7$ tickets), AI redistribution suggestions | Team Leads, Administrators | `SCR-DB-01` $\rightarrow$ `SCR-DB-12` Modal |
| **Live Incident Broadcast** | Instant WebSocket streaming of critical P1 tickets, outage surge clusters, and automated workflow failures | On-Duty Engineers, Administrators | Top Viewport Banner $\rightarrow$ `SCR-DB-10` Drawer |
| **Technician Workbench** | Priority-ordered personal task queue, unassigned team pool with 1-click ticket claim, personal SLA countdowns | Frontline Support Technicians | `SCR-DB-02` Technician Workbench |

---

### Sub-Features within F02

* **F02.1 Executive KPI Metrics & Intake Pipeline:**
  * Real-time aggregation cards: Total Ingested Tickets, New (`OPEN`), In Progress (`IN_PROGRESS`), Waiting for Customer (`NEEDS_INFO`), and Completed (`RESOLVED`/`CLOSED`).
  * Dedicated **AI Automation Rate** KPI card tracking percentage and count of tickets resolved autonomously without human intervention.
* **F02.2 Intake Trends & Distribution Analytics:**
  * Priority breakdown visualization (P1 Critical, P2 High, P3 Medium, P4 Low).
  * Category distribution breakdown (Hardware, Network, Access & Permissions, Software, Security).
  * Hourly intake volume trend line chart across the selected observation window.
* **F02.3 SLA Breach Early Warning Radar & Bottleneck Watch:**
  * Two-tier SLA status indicators: Yellow Warning ($\le 20\%$ time left) and Red Overdue Alert (Breached).
  * Prioritized Top 10 critical tickets list with live countdown timers and 1-click escalation modal.
* **F02.4 Technician Workload Capacity Monitoring:**
  * Mathematical capacity index: $\text{Workload Index} = \text{Active Open Tickets} + (\text{Count of P1/P2} \times 2)$.
  * Capacity status categorization: Normal ($\le 4$, Green), Busy ($5 - 7$, Yellow), Overloaded ($> 7$ or $\ge 2$ P1/P2, Red).
  * Team-level aggregation and comparative capacity gauges.
* **F02.5 AI-Assisted Workload Rebalancing Suggestions:**
  * Automated detection of team capacity imbalances.
  * AI-generated task transfer proposals from overloaded to available engineers with confidence scores.
* **F02.6 Real-Time Incident Stream & Emergency P1 Broadcast:**
  * WebSocket channel (`WS /api/v1/dashboard/live-feed`) broadcasting critical events with sub-second latency.
  * Flashing red viewport banner for P1 incidents with mandatory acknowledgement (`ACK`).
  * Spike detection algorithm flagging $> 5$ matching tickets within 15 minutes as potential widespread outages.
* **F02.7 Context-Preserving Deep Metric Drill-Down:**
  * 1-click navigation from any aggregate chart or card directly to pre-filtered queries on the Ticket List (`/tickets?query=...`).
* **F02.8 Technician Personal Workbench & 1-Click Ticket Claim:**
  * Filtered view showing "My Queue Today" sorted by nearest SLA deadline.
  * Unassigned team pool with 1-click "Claim Ticket" action immediately updating status to `IN_PROGRESS`.
  * Individual daily/weekly throughput metrics and CSAT satisfaction ratings.
* **F02.9 Multi-Dimensional Filtering & Temporal Scoping:**
  * Preset time filters: "Today / Past 24h", "Last 7 Days", "Last 30 Days", and "Custom Date Range" (up to 365 days).
  * Engineering team filter for administrators and multi-team leads.

---

### Outside F02 (Owned by Other Features)

* **Raw ticket CRUD, conversation messaging, attachments:** $\rightarrow$ F03 Ticket Management.
* **AI NLP extraction, confidence modeling, ticket summarization algorithms:** $\rightarrow$ F04 AI Ticket Processing (F02 consumes models).
* **Automated routing rule evaluation, SLA business calendar calculation engines:** $\rightarrow$ F05 Workflow & Automation (F02 visualizes status).
* **Long-term historical trend reports, executive quarterly BI export:** $\rightarrow$ F07 Reports & Analytics.
* **Full immutable system audit logs:** $\rightarrow$ F08 Activity Log.
* **User accounts, skills matrix, team membership provisioning:** $\rightarrow$ F09 User Management.

**Related:** [Use Cases](02-use-cases.md) · [Acceptance Criteria](03-acceptance-criteria.md) · [INVEST Requirements](04-invest-requirements.md) · [Information Architecture](information-architecture/README.md)

