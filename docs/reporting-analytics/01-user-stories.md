# User stories — Reporting & Analytics

Each story has acceptance criteria (AC). Related use cases, metrics `M-xx`, `BR-BC-xx` and proposed permissions: [02-use-cases.md](02-use-cases.md). `BR-xx` rules: [mvp.md](../mvp.md); `BR-WF-xx`: [business-rules.md](../requirements/business-rules.md).

**US-BC-01 — Ticket statistics by type and group** (UC-BC-01)
As an Admin, I want to see how many tickets come in, stay open and get closed per ticket type and group, so that I can spot overloaded groups and recurring problems.
- AC1: Choosing a time range shows new, open, closed, cancelled, reopened and escalated counts per type and per group, with a daily / weekly / monthly trend.
- AC2: Counts assign tickets to the period by creation date; `CANCELLED` tickets appear in volume only.
- AC3: Tickets in `TRIAGE` appear under "Unassigned".
- AC4: Range with from > to, or longer than the maximum → `422` `INVALID_DATE_RANGE`.

**US-BC-02 — Statistics limited to my scope** (UC-BC-01, UC-BC-02, UC-BC-04)
As a technician, I want to see figures for tickets assigned to me and to my groups, so that I can follow my team's workload without seeing other teams' data.
- AC1: A TECHNICIAN's figures are computed only on tickets assigned to them or their groups; the totals equal the count of tickets they can open.
- AC2: Filtering by a group they do not belong to → `403`.
- AC3: After being removed from a group, the next request no longer includes that group's tickets (BR-07).
- AC4: REQUESTER or AI_AGENT calling a report API → `403`.
- AC5: A TECHNICIAN does not see other technicians' individual figures.

**US-BC-03 — Handling time and SLA breaches** (UC-BC-02)
As an Admin, I want to see response and resolution times and which tickets breached their SLA, so that I can fix bottlenecks before they become a habit.
- AC1: The report shows M-01 and M-02 (mean, median, P90) and the M-03 breach rate by priority, type and group.
- AC2: Time spent in `NEEDS_INFO` is not counted in M-02 (BR-WF-07).
- AC3: A ticket counts as breached exactly when Workflow set `sla_breached = true`; the report and Workflow never disagree on the same ticket.
- AC4: The at-risk list shows open tickets with an `SLA_WARNING` and no breach yet.
- AC5: If SLA data is unavailable, M-03 shows "unavailable", not 0%.

**US-BC-04 — AI self-resolution rate** (UC-BC-03)
As an Admin, I want to know how many tickets the AI really solved on its own, so that I can decide whether to widen or narrow what the AI is allowed to do.
- AC1: A ticket closed with no TECHNICIAN or ADMIN handling action and not reopened within N days counts in the numerator of M-04.
- AC2: A ticket where a technician retried or manually resolved a workflow run does **not** count as AI-resolved.
- AC3: `CANCELLED` tickets, tickets auto-closed for no response, and tickets closed less than N days ago are excluded from both numerator and denominator; the last group is shown as "under observation".
- AC4: The report also shows M-05 (AI classification corrected), M-06 (helpful KB suggestions) and M-08 (route A / B / C share with route C reasons).
- AC5: Each report shows the N in use; changing N in UC-BC-06 changes the figure on the next view.
- AC6: TECHNICIAN calling this report → `403`.

**US-BC-05 — Requester satisfaction** (UC-BC-04)
As an Admin, I want to see satisfaction scores together with how many people actually rated, so that I do not draw conclusions from a handful of ratings.
- AC1: The report shows the 1–5 average, the count per score and the response rate (M-07) side by side.
- AC2: Response rate below the threshold → warning "few ratings, not representative".
- AC3: Tickets rated 1–2 are listed with a link to the ticket.
- AC4: Rating comments are visible to ADMIN only.

**US-BC-06 — Filter and export a report** (UC-BC-05)
As an Admin, I want to export the report I am looking at to CSV or XLSX, so that I can share it with management.
- AC1: The exported file contains the same rows and figures as the preview with the same filters.
- AC2: The file contains no internal notes, passwords, tokens, API keys, workflow payloads or AI extracted fields (BR-BC-04).
- AC3: A cell starting with `=`, `+`, `-` or `@` is written with a leading `'`.
- AC4: Every export creates an audit log entry `REPORT_EXPORTED` with the report, filters, format and row count.
- AC5: If the audit log cannot be written → `503` `AUDIT_UNAVAILABLE`, no file is returned.
- AC6: More rows than the limit → `422` `EXPORT_TOO_LARGE`.
- AC7: TECHNICIAN calling the export API → `403`.

**US-BC-07 — Configure report parameters** (UC-BC-06)
As an Admin, I want to set the observation window N and the report thresholds, so that the figures match how our organization works.
- AC1: Saving creates an audit log entry `REPORT_CONFIG_UPDATED` with before and after values.
- AC2: Invalid values (negative, N = 0, limit = 0) → `422`.
- AC3: The screen does not edit SLA targets; it links to the Workflow SLA configuration (UC-WF-02).
- AC4: Non-ADMIN → `403`.
