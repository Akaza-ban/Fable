# Acceptance Criteria — Dashboard (V1)

> Design status: TARGET V1  
> Implementation status: NOT IMPLEMENTED — all criteria are acceptance targets  

These criteria specify verifiable system behaviors and technical constraints for the **Dashboard (F02)** subsystem. All authorization, scope boundaries, and metric calculations **must** be enforced by server APIs regardless of frontend client representation.

---

## AC-UC-DB-01 — Real-Time Operations Monitoring

*(US-DB-01, US-DB-02, US-DB-04)*

- Only users with the `ADMIN` role can access `/admin/dashboard`; requests from `REQUESTER` receive `403 Forbidden` and redirect to `/portal`.
- Default view loads real-time operational metrics for the past 24 hours (`date=today`).
- Exactly 5 core KPI status cards are rendered: Total Ingested Today, New (`OPEN`), In Progress (`IN_PROGRESS`), Waiting for Customer (`NEEDS_INFO`), and Completed (`RESOLVED`/`CLOSED`).
- The **AI Automation Rate Today** card displays the percentage:
  $$\text{AI Automation Rate} = \left(\frac{\text{Tickets resolved via AI today (Flow A)}}{\text{Total closed tickets today}}\right) \times 100$$
  formatted with 1 decimal place (e.g., `38.5%`), alongside an absolute count breakdown tooltip on hover.
- Priority breakdown displays four distinct slices matching standard colors: P1 Critical (Red `#b91c1c`), P2 High (Orange `#c2410c`), P3 Medium (Yellow `#a16207`), and P4 Low (Blue `#475569`).
- Aggregated dashboard summary payload is served from Redis cache (`TTL 60s`); cache miss queries complete within $\le 1.5$ seconds (P95).
- Clicking the manual refresh button (`🔄 Refresh`) purges the user's localized Redis cache key and requests fresh calculations from the database.
- Clicking any KPI metric card or chart slice navigates directly to `/tickets` preserving query parameter contexts.

---

## AC-UC-DB-02 — Live SLA Radar & Rapid Intervention

*(US-DB-03, US-DB-04)*

- The SLA radar scans active, non-closed tickets against their assigned priority SLA response and resolution targets governed by F05.
- **Two-tier warning thresholds:**
  - A ticket with remaining time $\le 20\%$ of target duration displays a **Yellow Warning** badge with an active countdown timer (`HH:mm:ss`).
  - A ticket that exceeds the deadline transitions immediately to a **Red Alert** badge displaying negative elapsed time (e.g., `-00:18:24`).
- Top 10 critical tickets list is sorted primarily by priority descending (P1 $\rightarrow$ P4) and secondarily by longest overdue duration descending.
- Clicking an at-risk ticket row opens the `SCR-DB-11` SLA Escalation Modal within $\le 300\text{ ms}$.
- The escalation modal mandates an escalation action: either "Nudge Assignee" (dispatches instant notification via `SCR-91`) or "Reassign" (requires selecting an engineer and entering a non-empty reason).
- Successful reassignment updates the ticket record, resets assignee counters, and writes an audit log entry with action `SLA_REASSIGNED` to F08 (`UC-NK-05`).
- When zero tickets are at risk or breached, the widget displays the positive status card: *"All active tickets are currently meeting SLA targets."*.

---

## AC-UC-DB-03 — Real-Time Incident Alerting & Event Feed

*(US-DB-07)*

- Dashboard maintains a persistent WebSocket connection to `/api/v1/dashboard/live-feed` when open, authenticated via Bearer JWT token.
- If WebSocket connection drops, UI displays `🔴 Reconnecting` and falls back to REST polling every 30 seconds after 15 seconds of disconnection.
- When a P1 (Critical) ticket is ingested, system broadcasts the event to all internal staff with sub-second latency ($\le 1.0\text{s}$, P95).
- Receiving a P1 event triggers a persistent flashing red banner across the top viewport and plays an audible alert chime (if enabled in user settings).
- The banner remains visible until acknowledged by authorized personnel (`POST /api/v1/dashboard/alerts/:id/ack`).
- Spike detection algorithm activates an alert when $> 5$ tickets sharing matching keywords/categories are created within 15 minutes.
- Automated workflow failure alert: When an automated API execution fails after maximum retries, the banner provides a direct link to F08 `/audit/workflow-runs?status=FAILED` to inspect technical payload and error trace.
- Clicking an alert banner opens slide-over drawer `SCR-DB-10` displaying AI root-cause hypothesis, correlated tickets, and quick-dispatch buttons.

---

## AC-UC-DB-04 — Technician Personal Workspace & Queue Operations

*(US-DB-08)*

- Users with the `TECHNICIAN` role accessing `/dashboard` are immediately redirected to `/technician/dashboard` (`SCR-DB-02`).
- "My Queue Today" displays active tickets with `assigned_to = current_user.id` and status $\in$ {`ASSIGNED`, `IN_PROGRESS`, `NEEDS_INFO`}, ordered by nearest SLA deadline.
- "Unassigned Team Pool" lists group tickets with `group_id IN (my_groups)` and `assigned_to IS NULL`.
- Clicking [Claim Ticket] invokes `POST /api/v1/dashboard/tickets/:id/claim`:
  - If already claimed by a peer, displays a non-blocking toast *"Ticket was just claimed by another technician"* and removes the row.
  - If successful, transitions status to `IN_PROGRESS`, sets `assigned_to = current_user.id`, logs audit entry to F08, and moves the card to "My Queue" within $\le 500\text{ ms}$.
- Each active task card features an isolated countdown timer synchronized with server time.

---

## AC-UC-DB-05 — Requester Self-Service Portal & Dashboard

*(US-DB-09)*

- Users with the `REQUESTER` role logging in are routed to `/portal` (`SCR-DB-03`).
- **Action-Needed Container:**
  - Queries tickets for the current requester in `NEEDS_INFO` status; displays yellow warning cards with [Answer Question] button. Clicking navigates to `/requests/:id#clarification`.
  - Queries tickets in `RESOLVED` status; displays green success cards with [Confirm Resolution] button. Clicking opens CSAT star-rating modal (1–5 stars) and transitions ticket to `CLOSED`.
- **My Requests Tracker:**
  - Renders the 5 most recent tickets of the user with visual milestone progress bars and target SLA completion times.
- **Smart Search & KB Deflection:**
  - Search input box invokes `GET /api/v1/kb/search` after a 300ms debounce, returning top 3 matching verified articles in a dropdown to deflect ticket submission.
- **IT Operational Announcements:**
  - Displays active broadcast notices published by Administrators (maintenance windows, system outages).

---

**Related:** [Features](01-features.md) · [Use Cases](03-use-cases.md) · [INVEST Requirements](05-invest-requirements.md)
