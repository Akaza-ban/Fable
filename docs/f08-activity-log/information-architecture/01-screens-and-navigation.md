# Screens, Routes, and Navigation — F08 Activity Log

> Design status: TARGET V1
>
> Related: [IA overview](README.md) · [User flows](02-user-flows.md) · [UI states](03-ui-states-and-accessibility.md)

## 1. Host page and its F08 sections

| Host page | Owner | Route | F08 sections |
|---|---|---|---|
| SCR-T-02 Ticket detail | F03 | `/tickets/:code` | SEC-NK-01 (`#history`), SEC-NK-02 (approval chain, inside `#automation` for staff and as a status card for the requester) |

| ID | Section | Host | Content | Stories |
|---|---|---|---|---|
| SEC-NK-01 | Ticket history | SCR-T-02 `#history` | Timeline newest first: actor marker (person / AI + agent / Automation), action, before → after, time, evidence links (`decision_id` → SCR-WF-31 for Admin, `run_id` → SCR-NK-05, `analysis_id`); filter by event group (status, assignment, replies, AI, workflow, SLA). Requester: allowlisted events, AI shown as "IT assistant", workflow as one summary line | US-NK-01, 02, 07 |
| SEC-NK-02 | Approval chain | SCR-T-02 | Vertical stepper: Request → AI understanding → Policy check (one row per condition ✓ / ✗) → Decision → Execution attempts → Verification → Requester confirmation → Revoke (if time-limited); banner "Approved — execution failed" with assignee; *View AI decision* → SCR-WF-31 (Admin). Group technician: steps without payloads. Requester: one status line and the short reason | US-NK-05 |

## 2. F08 pages

| ID | Name | Type | Roles | Opened from | Route | Content | Stories |
|---|---|---|---|---|---|---|---|
| SCR-NK-02 | Activity log | Page | Admin | Sidebar *Audit*; *View log* on user, service account, rule, SLA policy pages | `/audit/activity-log` (`?actor=&actor_type=&action=&target=&evidence=&agent=&prompt_version=&from=&to=`) | Filters; paginated table (time, actor, action, target, summary); side panel with `before` / `after` diff and evidence links; no edit / delete control | US-NK-04, 10 |
| SCR-NK-04 | Workflow runs | Page | Admin | Sidebar *Audit*; SCR-91 (AI unavailable, run failed) | `/audit/workflow-runs` (`?status=&error=&operation=&connector=&ticket=&from=&to=`) | Counters by status; table (run id, ticket, operation, status, error type, attempts, started, duration) | US-NK-06, 07 |
| SCR-NK-05 | Workflow run detail | Page | Admin; assigned technician (also via `#automation`) | SCR-NK-04, SEC-NK-01 | `/audit/workflow-runs/:id` | Run summary; attempts (time, duration, result code, masked request / response); error message; link to SCR-T-02. *Retry* / *Resolved manually* are F05 modals (SCR-WF-20, 21), shown only to the assigned technician and Admin | US-NK-06 |

The ID `SCR-NK-01` and `SCR-NK-03` of earlier drafts are now the sections SEC-NK-01 and SEC-NK-02; the numbers are not reused.

## 3. Routes and permissions

The API enforces every row; hiding an element is only for usability. "Scope" means tickets assigned to the technician or one of their groups (F01).

| Item | Requester | Technician | Admin | Unauthorized result |
|---|---|---|---|---|
| SEC-NK-01 | Own tickets, allowlist | Scope, full | ✓ | `403`; out-of-allowlist entries absent from the response |
| SEC-NK-02 | Own tickets, status line | Assigned: full; group: summary | ✓ + link to SCR-WF-31 | Fields absent from the response |
| SCR-NK-02 | — | — | ✓ | SCR-90 (`/403`) |
| SCR-NK-04 | — | — | ✓ | SCR-90 |
| SCR-NK-05 | — | Assigned: full; group: summary | ✓ | SCR-90 |
| Any `PUT` / `PATCH` / `DELETE` on the log | `405` | `405` | `405` | — |

## 4. Navigation

| Role | Sidebar | What changes because of F08 |
|---|---|---|
| Requester | My tickets, New ticket, Profile | Nothing new; *History* tab and status card on their ticket |
| Technician | Work queues, Reports, Profile | Nothing new; *History* tab and approval chain on SCR-T-02 |
| Admin | Dashboard, Tickets, …, **Audit** | **Audit** group: Activity log (SCR-NK-02), Workflow runs (SCR-NK-04) |

Contextual navigation: *View log* buttons on user, service account, assignment rule (F05 SCR-WF-01) and SLA policy (SCR-WF-02) pages open SCR-NK-02 pre-filtered; a ticket in any entry opens SCR-T-02; notifications about failed runs (SCR-91) open SCR-NK-05.

## 5. Story-to-screen mapping

| Story | MoSCoW | Role | Screens / sections |
|---|---|---|---|
| US-NK-01 | Must | Technician | SEC-NK-01 |
| US-NK-02 | Must | Requester | SEC-NK-01 (allowlist) |
| US-NK-03 | Must | Admin | No screen: no edit / delete control (D-NK-IA-05); API `405` |
| US-NK-04 | Must | Admin | SCR-NK-02 |
| US-NK-05 | Must | Admin, Technician, Requester | SEC-NK-02, SCR-WF-31 |
| US-NK-06 | Must | Admin, Technician | SCR-NK-04, SCR-NK-05, `#automation` |
| US-NK-07 | Must | Requester | SEC-NK-01 (`TICKET_ROUTED_TO_HUMAN`) |
| US-NK-08 | Must | Admin | No screen: masked values everywhere |
| US-NK-09 | Must | Admin | SCR-NK-05 (`AUDIT_UNAVAILABLE`), SCR-91 alert |
| US-NK-10 | Should | Admin | SCR-NK-02 (filters `agent`, `prompt_version`) |

## 6. Screen-to-API mapping

Prefix `/api/v1`. Errors use `ProblemDetails`. Every endpoint filters by the caller's scope in the query.

| Screen | Method and path | Permission |
|---|---|---|
| SEC-NK-01 | `GET /tickets/:code/history` | `ticket_history:read` + `ticket:read` |
| SEC-NK-02 | `GET /tickets/:code/approvals` | `approval:read` + `ticket:read` |
| SCR-NK-02 | `GET /audit-logs`, `GET /audit-logs/:id` | `audit:read` |
| SCR-NK-04 | `GET /workflow-runs` | `workflow_log:read` |
| SCR-NK-05, `#automation` | `GET /workflow-runs/:id` (F05 `getWorkflowRun`, one endpoint) | `workflow_log:read` |

- No `POST`, `PUT`, `PATCH` or `DELETE` exists on `/audit-logs` or `/tickets/:code/history`; such calls return `405`.
- Period longer than the maximum → `422 INVALID_DATE_RANGE`.

## 7. Requests to other branches

| Owner | Request |
|---|---|
| F03 | Host SEC-NK-01 in `#history` and SEC-NK-02 on SCR-T-02 |
| F05 | Keep `getWorkflowRun` as the one run endpoint used by `#automation` and SCR-NK-05; write evidence ids on every automated entry (BR-WF-05); link from SCR-WF-31 back to the ticket history |
| F04 | Write `agent`, `model_version`, `prompt_version`, `analysis_id` on AI entries (BR-AI-02) |
| F01 | Accept the BR-09 extension, the evidence ids and the permissions (Q-NK-04); *View log* buttons on user and service account pages; decide `SYSTEM` vs `AUTOMATION` (Q-NK-03) |
