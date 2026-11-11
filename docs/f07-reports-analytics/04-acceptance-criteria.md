# Acceptance Criteria — F07 Reports & Analytics

> Design status: TARGET V1
>
> Implementation status: NOT STARTED. Every criterion is an acceptance target.

Format: *Given* (context) · *When* (action) · *Then* (observable result). Criteria are grouped by use case (`AC-UC-BC-xx`, as in F04 and F05) and numbered by story: `AC-BC-04.3` is the third criterion of US-BC-04. The UI may satisfy a criterion visually, but permissions and rules **must** be enforced by the API.

Demo values: observation window **N = 7 days**, self-resolution target **50%**, satisfaction low-response threshold **20%**, export limit **50,000 rows**, maximum period **366 days**. Sample people (same as F04): technician **Hai** (IT Helpdesk group, not a lead), technician **Quang** (SAP group, lead of SAP), Admin **Thu**.

## Reference data set

| ID | Content | Use |
|---|---|---|
| RD-BC-01 | 1,000 seeded tickets over 8 weeks, with known expected figures for every metric: tickets in every status, every resolution code (`ANSWERED_BY_KB`, `SOLVED_BY_KB`, automation, technician), every `escalationClass`, `NEEDS_INFO` pauses, SLA breaches and warnings, reopened tickets, cancelled tickets, auto-closed tickets, ratings with low response, tickets moved across groups | Every criterion below is checked against the expected values of RD-BC-01; report, Dashboard (F02) and export must return identical numbers |

## AC-UC-BC-01 — Ticket volume and status

*(US-BC-01, US-BC-02)*

- **AC-BC-01.1** *Given* Thu (Admin), *when* she opens Tickets for the last 30 days, *then* she sees new, open, closed, cancelled, reopened and escalated counts by type, `domain/subcategory`, group, priority and channel, with a daily / weekly / monthly trend.
- **AC-BC-01.2** *Given* a ticket created on day 1 and closed on day 40, *when* the period is days 1–30, *then* it counts as new in that period.
- **AC-BC-01.3** *Given* a ticket in `TRIAGE`, *when* the report groups by group, *then* it appears under "Unassigned".
- **AC-BC-01.4** *Given* from > to or a period over 366 days, *when* the report is requested, *then* the API returns `422 INVALID_DATE_RANGE`.
- **AC-BC-02.1** *Given* Hai, *when* he opens Tickets, *then* every total equals the number of tickets he can open in the ticket list with the same filters.
- **AC-BC-02.2** *Given* Hai, *when* he filters on the SAP group, *then* the API returns `403`.
- **AC-BC-02.3** *Given* Hai was removed from IT Helpdesk, *when* he sends the next request, *then* IT Helpdesk tickets are no longer counted (BR-07).
- **AC-BC-02.4** *Given* a Requester or the `AI_AGENT` service account, *when* it calls any `/reports/*` endpoint, *then* the API returns `403`.

## AC-UC-BC-02 — Handling time and SLA

*(US-BC-03)*

- **AC-BC-03.1** *Given* the SLA report, *then* it shows M-01 (human and AI separately), M-02 (mean, median, P90) and M-03 by priority, type and group.
- **AC-BC-03.2** *Given* a ticket that spent 2 working hours in `NEEDS_INFO`, *then* M-02 excludes those 2 hours (BR-WF-07).
- **AC-BC-03.3** *Given* any ticket, *then* it counts as breached exactly when F05 set `sla_breached = true`; the report and F05 never disagree.
- **AC-BC-03.4** *Given* an open ticket with `SLA_WARNING` and no breach, *then* it is in the *At risk* tab.
- **AC-BC-03.5** *Given* F05 SLA data is unavailable, *then* M-03 shows "Unavailable", not `0%`, and M-01 / M-02 are still shown.

## AC-UC-BC-03 — AI self-resolution and quality

*(US-BC-04, US-BC-08, US-BC-09)*

- **AC-BC-04.1** *Given* a ticket answered by the agent (`ANSWERED_BY_KB`), confirmed by the requester, closed 10 days ago and not reopened, *then* it is in the numerator of M-04 under the path "Agent KB answer".
- **AC-BC-04.2** *Given* a ticket where a technician retried a workflow run or added an internal note, *then* it is **not** in the numerator.
- **AC-BC-04.3** *Given* `CANCELLED` tickets, tickets auto-closed for no answer, and tickets closed 3 days ago, *then* they are in neither numerator nor denominator; the last group is shown as "Under observation".
- **AC-BC-04.4** *Given* a ticket the agent answered, the requester said *Not solved*, a technician fixed it, *then* it is not in the numerator and appears in the list "Not solved after agent answer".
- **AC-BC-04.5** *Given* the weekly view, *then* each week shows M-04 and the target line, and the report states "N = 7 days, target 50%".
- **AC-BC-04.6** *Given* a Technician, *when* they call `/reports/ai`, *then* `403`.
- **AC-BC-08.1** *Given* the AI report, *then* M-08 shows the share of `AUTO_APPROVED`, `NEEDS_INFO`, `ROUTE_TO_HUMAN` with route C reasons ranked.
- **AC-BC-08.2** *Given* escalated tickets, *then* M-10 splits them into `AGENT_CANNOT_RESOLVE` (with reason), `SYSTEM_BUG`, `SOFTWARE_REQUIREMENT`, and the three add up to all tickets handed to a technician.
- **AC-BC-09.1** *Given* corrections stored by F04, *when* the Admin filters prompt version v3 vs v4, *then* M-05 and M-06 are shown per version.
- **AC-BC-09.2** *Given* analyses with status `INVALID` or `FAILED`, *then* M-11 counts them and links to the ticket list.

## AC-UC-BC-04 — Requester satisfaction

*(US-BC-05)*

- **AC-BC-05.1** *Given* rated tickets, *then* the average, the count per score and the response rate are shown side by side.
- **AC-BC-05.2** *Given* a response rate of 12%, *then* the warning "Few ratings, not representative" is shown.
- **AC-BC-05.3** *Given* tickets rated 1–2, *then* they are listed with a link to the ticket.
- **AC-BC-05.4** *Given* Hai, *then* he sees no comments and no other technician's individual figures.

## AC-UC-BC-05 — Filter and export

*(US-BC-06)*

- **AC-BC-06.1** *Given* a report with filters, *when* Thu exports it, *then* the file has the same rows and figures as the screen.
- **AC-BC-06.2** *Given* any export, *then* it contains no internal notes, passwords, tokens, API keys, workflow payloads, AI extracted fields or rating comments (BR-BC-04).
- **AC-BC-06.3** *Given* a cell starting with `=`, `+`, `-` or `@`, *then* it is written with a leading `'`.
- **AC-BC-06.4** *Given* an export, *then* the audit log has `REPORT_EXPORTED` with report, filters, format and row count.
- **AC-BC-06.5** *Given* the audit log cannot be written, *then* the API returns `503 AUDIT_UNAVAILABLE` and no file.
- **AC-BC-06.6** *Given* 60,000 matching rows, *then* `422 EXPORT_TOO_LARGE`.
- **AC-BC-06.7** *Given* a Technician, *when* they call the export API, *then* `403`.

## AC-UC-BC-06 — Report parameters

*(US-BC-07)*

- **AC-BC-07.1** *Given* Thu changes N from 7 to 14, *then* `REPORT_CONFIG_UPDATED` stores before and after, and the next view of M-04 uses 14.
- **AC-BC-07.2** *Given* N = 0, a negative value, a limit of 0 or a target outside 1–100%, *then* `422`.
- **AC-BC-07.3** *Given* the settings page, *then* SLA targets are not editable there; a link opens F05 SCR-WF-02.
- **AC-BC-07.4** *Given* a non-Admin, *then* `403`.

## AC-UC-BC-07 — Team workload

*(US-BC-10)*

- **AC-BC-10.1** *Given* Quang (lead of SAP), *then* he sees one row per SAP technician with open, assigned, resolved and oldest open ticket.
- **AC-BC-10.2** *Given* Quang, *when* he requests the IT Helpdesk team, *then* `403`.
- **AC-BC-10.3** *Given* Hai (not a lead), *then* he sees only his own row and the group total.

## AC-UC-BC-08 — Weekly digest

*(US-BC-11)*

- **AC-BC-11.1** *Given* Monday 08:00, *then* recipients receive EML-BC-01 with last week's figures, identical to the AI and SLA reports for that week.
- **AC-BC-11.2** *Given* the digest is sent, *then* the audit log has `REPORT_DIGEST_SENT` with source `AUTOMATION`.
- **AC-BC-11.3** *Given* the email contains links, *when* a recipient without a session clicks one, *then* they sign in first and see only what their role allows.

**Related:** [User stories](03-user-stories-invest.md) · [Use cases](02-use-cases.md) · [Business rules](05-business-rules.md)
