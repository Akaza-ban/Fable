# Acceptance Criteria — Dashboard (V1)

> Design status: TARGET V1  
> Implementation status: NOT IMPLEMENTED — all criteria are acceptance targets  

These criteria specify verifiable system behaviors and technical constraints for the **Dashboard (F02)** subsystem. All authorization, scope boundaries, and metric calculations **must** be enforced by server APIs regardless of frontend client representation.

---

## AC-UC-DB-01 — Monitor Overall Operations & Ticket Distribution

*(US-DB-01, US-DB-02, US-DB-04)*

- Only users with `ADMIN` or `TECHNICIAN_LEAD` roles can access `/admin/dashboard`; requests from `REQUESTER` receive `403 Forbidden` and redirect to `/my-tickets`.
- Default view loads metrics for the past 24 hours (`range=today`).
- Exactly 5 core KPI status cards are rendered: Total Ingested Tickets, New (`OPEN`), In Progress (`IN_PROGRESS`), Waiting for Customer (`NEEDS_INFO`), and Completed (`RESOLVED`/`CLOSED`).
- The **AI Automation Rate** card displays the percentage:
  $$\text{AI Automation Rate} = \left(\frac{\text{Tickets resolved via AI (Flow A)}}{\text{Total closed tickets}}\right) \times 100$$
  formatted with 1 decimal place (e.g., `38.5%`), alongside an absolute count breakdown tooltip on hover.
- Priority breakdown displays four distinct slices matching standard colors: P1 Critical (Red `#b91c1c`), P2 High (Orange `#c2410c`), P3 Medium (Yellow `#a16207`), and P4 Low (Blue `#475569`).
- Aggregated dashboard summary payload is served from Redis cache (`TTL 60s`); cache miss queries complete within $\le 1.5$ seconds (P95).
- Clicking the manual refresh button (`🔄 Refresh`) purges the user's localized Redis cache key and requests fresh calculations from the database.

---

## AC-UC-DB-02 — Track Bottlenecks & Manage SLA Breaches

*(US-DB-03, US-DB-04)*

- The SLA radar scans all active, non-closed tickets against their assigned priority SLA response and resolution targets.
- **Two-tier warning thresholds:**
  - A ticket with remaining time $\le 20\%$ of target duration displays a **Yellow Warning** badge with an active countdown timer (`HH:mm:ss`).
  - A ticket that exceeds the deadline transitions immediately to a **Red Alert** badge displaying negative elapsed time (e.g., `-00:18:24`).
- Top 10 critical tickets list is sorted primarily by priority descending (P1 $\rightarrow$ P4) and secondarily by longest overdue duration descending.
- Clicking an at-risk ticket row opens the `SCR-DB-11` SLA Escalation Modal within $\le 300\text{ ms}$.
- The escalation modal mandates an escalation action: either "Nudge Assignee" (dispatches instant high-priority notification via `SCR-91`) or "Reassign" (requires selecting an engineer and entering a non-empty reason).
- Successful reassignment updates the ticket record, resets assignee counters, and writes an audit log entry with action `AUDIT_SLA_REASSIGN`.
- When zero tickets are at risk or breached, the widget displays the positive status card: *"All active tickets are currently meeting SLA targets."*.

---

## AC-UC-DB-03 — Monitor & Rebalance Technician Workload

*(US-DB-05, US-DB-06)*

- Only `ADMIN` and `TECHNICIAN_LEAD` can view the Team Workload Capacity Matrix (`/api/v1/dashboard/workload`). Standard `TECHNICIAN` role receives `403` on team-wide workload queries.
- Individual workload score is calculated in real time using the standardized formula:
  $$\text{Workload Index} = \text{Active Open Tickets} + (\text{Count of P1/P2 Tickets} \times 2)$$
- Capacity status is determined by strict thresholds:
  - `NORMAL` (Green): $\text{Workload Index} \le 4$.
  - `BUSY` (Yellow): $\text{Workload Index} \in [5, 7]$.
  - `OVERLOADED` (Red): $\text{Workload Index} > 7$ OR holding $\ge 2$ unresolved P1/P2 tickets.
- Frontline technicians can only see their own workload score compared against team average; individual queue breakdowns of colleagues are strictly redacted (`DF-IA-DB-03`).
- When an engineer is flagged as `OVERLOADED`, clicking their status pill opens `SCR-DB-12` AI Rebalance Modal.
- The modal displays AI-recommended candidate tickets for reassignment, target recipient engineers currently categorized as `NORMAL`, and an AI confidence score ($\ge 0.80$).
- Applying the recommendation moves the ticket to the recipient engineer and emits real-time updates to both engineers' queues within $\le 1$ second.

---

## AC-UC-DB-04 — Real-Time Incident Alerting & Event Feed

*(US-DB-07)*

- The dashboard establishes a persistent WebSocket connection to `/api/v1/dashboard/live-feed` upon page mount using Bearer JWT authentication.
- If the WebSocket connection drops, the client displays `🔴 Reconnecting` and automatically falls back to REST polling every 30 seconds after 15 seconds of disconnection.
- Ingestion of a P1 (Critical) ticket broadcasts an event payload to all active internal staff within $\le 1.0$ second (P95).
- Upon receiving a P1 event, the dashboard mounts a persistent flashing red banner at the top of the viewport and triggers an audible chime (if enabled in preferences).
- The banner remains visible across all screens until an authorized user clicks "Acknowledge Incident" (`POST /api/v1/dashboard/alerts/:id/ack`).
- Outage spike detection triggers when $> 5$ tickets with matching keywords or category are ingested within a rolling 15-minute window, rendering a clustered incident warning banner.
- Clicking any alert banner opens the right-hand `SCR-DB-10` Live Incident Drawer containing AI root cause hypothesis, correlated ticket list, and team dispatch triggers.

---

## AC-UC-DB-05 — Technician Personal Workbench & Queue Operations

*(US-DB-08)*

- Authenticated users with the `TECHNICIAN` role accessing `/dashboard` are redirected immediately to `/technician/dashboard`.
- "My Queue Today" lists all tickets where `assigned_to = current_user.id` and status $\in$ {`ASSIGNED`, `IN_PROGRESS`, `NEEDS_INFO`}, ordered by nearest SLA deadline ascending.
- "Unassigned Team Pool" lists tickets where `group_id IN (current_user.groups)` and `assigned_to IS NULL`.
- Clicking "Claim Ticket" executes `POST /api/v1/dashboard/tickets/:id/claim`:
  - Validates ticket is still unassigned; if already claimed by a colleague, displays non-blocking toast *"Ticket was already claimed"* and removes item from list.
  - If successful, transitions ticket status to `IN_PROGRESS`, sets `assigned_to = current_user.id`, and moves the card to "My Active Queue" within $\le 500\text{ ms}$.
- Each active ticket card displays an isolated live countdown timer synchronized with server time.

---

## AC-UC-DB-06 — Multi-Dimensional Filtering & Deep Drill-Down Validation

*(US-DB-01, US-DB-04)*

- Custom date range validation:
  - Start date after End date $\rightarrow$ HTTP `400 Bad Request` with error code `INVALID_DATE_RANGE`.
  - Date interval exceeding 365 days $\rightarrow$ HTTP `400 Bad Request` with error code `DATE_RANGE_EXCEEDS_LIMIT`.
- Clicking any aggregate metric element transitions the browser to `/tickets` with exact corresponding query parameters:
  - Priority chart slice P1 $\rightarrow$ `/tickets?priority=P1&range=<current_range>&group_id=<current_group>`.
  - Breached SLA card $\rightarrow$ `/tickets?sla_status=BREACHED&range=<current_range>`.
  - In Progress KPI card $\rightarrow$ `/tickets?status=IN_PROGRESS&range=<current_range>`.
- The destination ticket list view renders exactly the dataset reflected in the dashboard metric counter.
- Pressing browser "Back" from `/tickets` returns to `/admin/dashboard` with previous filter state and scroll position preserved.

**Related:** [Feature Catalog](01-features.md) · [Use Cases](02-use-cases.md) · [INVEST Requirements](04-invest-requirements.md)

