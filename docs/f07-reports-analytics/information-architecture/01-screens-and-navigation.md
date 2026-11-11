# Screens, Routes, and Navigation — F07 Reports & Analytics

> Design status: TARGET V1
>
> Implementation status: NOT STARTED
>
> Related: [IA overview](README.md) · [User flows](02-user-flows.md) · [UI states](03-ui-states-and-accessibility.md)

## 1. F07 pages, tabs and modals

| ID | Name | Type | Roles | Opened from | Route | Content | Stories |
|---|---|---|---|---|---|---|---|
| — | Shared filter bar | Part of every tab | As the tab | — | Query string `?from=&to=&type=&domain=&group=&priority=&channel=&handler=&granularity=` | Period (7 / 30 / 90 days, 12 weeks, custom), type, `domain/subcategory`, group (Technician: own groups only), priority, channel, handler (AI only / with technician); line "Parameters in use: Asia/Ho_Chi_Minh, N = 7 days, target 50%"; *Export* (Admin) | US-BC-01 … 06 |
| SCR-BC-01 | Tickets | Tab | Admin; Technician (scope) | Sidebar *Reports*, SEC-BC-01 | `/analytics/tickets` | Tiles: new, open (backlog), closed, cancelled, reopened, escalated; trend chart; priority distribution P1–P4; table type × group with a "Team queue" row for tickets not yet given to a technician | US-BC-01, 02 |
| SCR-BC-02 | SLA | Tab | Admin; Technician (scope) | Sidebar, SEC-BC-01 | `/analytics/sla` | Tiles: first response (human / AI), resolution time (mean, median, P90), breach rate by target; breakdown by priority / type / group; tabs **Breached** and **At risk** (number, type, group, priority, SLA policy version, target missed, overdue by); note "SLA data from Workflow (F05)" | US-BC-03 |
| SCR-BC-03 | AI | Tab | Admin | Sidebar, SEC-BC-02, EML-BC-01 | `/analytics/ai` | M-04 tile with "under observation"; weekly line vs target; split by path (automation, agent KB answer, self-help, after AI question) and by domain agent; route A / B / C (M-08) with ranked route C reasons; escalations by class (M-10); quality: M-05, M-06, M-11 with model / prompt version filter; lists: AI-closed tickets reopened, corrected classifications (AI → human), *Not solved* after agent answer | US-BC-04, 08, 09 |
| SCR-BC-04 | Satisfaction | Tab | Admin; Technician (scope) | Sidebar | `/analytics/satisfaction` | Average next to response rate; distribution 1–5; by type / group / handler; ratings 1–2 list; comments (Admin only); warning "Few ratings, not representative" | US-BC-05 |
| SCR-BC-05 | Report settings | Page | Admin | Sidebar *Settings* | `/analytics/settings` | N days (read-only, from the F03 reopen window), low-response threshold, self-resolution target, export row limit, maximum period, digest recipients, *Send test digest*; link "SLA targets are set in Workflow" → SCR-WF-02; last changed by / at | US-BC-07, 11 |
| SCR-BC-10 | Export | Modal | Admin | *Export* on any tab | on the tab route | Format CSV / XLSX; row count; note "Internal notes, comments and secrets are never exported"; errors `EXPORT_TOO_LARGE`, `AUDIT_UNAVAILABLE` | US-BC-06 |

## 2. Sections hosted by other branches

| ID | Section | Host | Content | Stories |
|---|---|---|---|---|
| SEC-BC-01 | KPI widgets | F02 Dashboard | Open, breached, M-04 this week, with a link to the matching tab; same API, short default period | US-BC-01, 03, 04 |
| SEC-BC-02 | AI cards | F05 SCR-WF-30 | Daily AI routing outcome (M-08: auto-approved, asked back, routed to a person); AI self-resolution rate (M-04) of the last full week past the N-day window, with the "under observation" count. Computed by `/reports/ai`; labels use the M-xx names; *View full report* → SCR-BC-03 | US-BC-04, 08 |

## 3. Emails

| ID | Email | Sent when | Links | Stories |
|---|---|---|---|---|
| EML-BC-01 | Weekly digest | Monday 08:00 | Each figure → its tab with last week's filter (sign-in required) | US-BC-11 |

## 4. Routes and permissions

The API enforces every row; hiding an element is only for usability. "Scope" means tickets assigned to the technician or one of their groups (F01).

| Item | Requester | Technician | Admin | Unauthorized result |
|---|---|---|---|---|
| SCR-BC-01, 02, 04 | — | Scope | ✓ | SCR-90 (`/403`) |
| SCR-BC-03 | — | — | ✓ | SCR-90 |
| SCR-BC-05 | — | — | ✓ | SCR-90 |
| SCR-BC-10 | — | — | ✓ | `403`, button not shown |
| SEC-BC-01 | — | Scope | ✓ | Widget not rendered |
| SEC-BC-02 | — | — | ✓ (host is Admin only) | — |
| Comments on SCR-BC-04 | — | ✗ | ✓ | Field absent from the response |

## 5. Navigation

| Role | Sidebar | What changes because of F07 |
|---|---|---|
| Requester | My tickets, New ticket, Profile | Nothing |
| Technician | Work queues, **Reports**, Profile | *Reports* opens SCR-BC-01; tabs AI hidden |
| Admin | Dashboard, Tickets, …, **Reports**, Settings › **Report settings** | All tabs; *Export* on each tab |

Contextual navigation: every count, bar and list row opens SCR-T-04 with the same filters (D-BC-IA-04); a ticket number opens SCR-T-02 (checked against `ticket:read`); the AI tab links to F05 SCR-WF-30 and F04 learning examples (SCR-AI-01) for follow-up.

## 6. Story-to-screen mapping

| Story | MoSCoW | Role | Screens / sections / emails |
|---|---|---|---|
| US-BC-01 | Must | Admin | SCR-BC-01, SEC-BC-01 |
| US-BC-02 | Must | Technician | SCR-BC-01 (scoped), filter bar |
| US-BC-03 | Must | Admin, Technician | SCR-BC-02 |
| US-BC-04 | Must | Admin | SCR-BC-03, SEC-BC-02 |
| US-BC-05 | Should | Admin, Technician | SCR-BC-04 |
| US-BC-06 | Must | Admin | SCR-BC-10 |
| US-BC-07 | Must | Admin | SCR-BC-05 |
| US-BC-08 | Must | Admin | SCR-BC-03 |
| US-BC-09 | Should | Admin | SCR-BC-03 (quality section) |
| US-BC-11 | Could | Admin | EML-BC-01, SCR-BC-05 |

## 7. Screen-to-API mapping

Prefix `/api/v1`. Errors use `ProblemDetails`. Every endpoint filters by the caller's scope in the query (BR-BC-02).

| Screen | Method and path | Permission |
|---|---|---|
| SCR-BC-01, SEC-BC-01 | `GET /reports/tickets` | `report:read` |
| SCR-BC-02 | `GET /reports/sla`, `GET /reports/sla/tickets?state=breached\|at_risk` | `report:read` |
| SCR-BC-03, SEC-BC-02 | `GET /reports/ai` | `report:read_ai` |
| SCR-BC-04 | `GET /reports/satisfaction` | `report:read` |
| SCR-BC-10 | `POST /reports/export` (report, filters, format) | `report:export`; writes `REPORT_EXPORTED` first |
| SCR-BC-05 | `GET` / `PATCH /report-config`, `POST /report-config/test-digest` | `report:read` / `report:configure` |

## 8. Requests to other branches

| Owner | Request |
|---|---|
| F02 | Dashboard widgets call `/reports/*` (SEC-BC-01) instead of their own counts |
| F05 | SCR-WF-30 cards call `/reports/ai` (SEC-BC-02); label the daily cards as M-08 (auto-approved / clarification / human-routing), and show self-resolution (M-04) only weekly, never per day; confirm Q-BC-05 (target in `SLA_BREACHED`) |
| F04 | Keep corrections and votes countable by model and prompt version (AC-AI-14.3); expose `agent` and `escalationClass` on the current analysis |
| F03 | Store the resolution codes `ANSWERED_BY_KB` and `SOLVED_BY_KB` and the 1–5 rating; ticket list SCR-T-04 accepts the report filters in its query string |
| F01 | Accept the `report:*` permissions |
