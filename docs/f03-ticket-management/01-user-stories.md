# User Stories & Acceptance Criteria — F03 Ticket Management

> Design status: TARGET V1
>
> Implementation status: NOT IMPLEMENTED — all criteria are acceptance targets
>
> Scope: US-TK-01 … US-TK-22 for UC-TK-01 … UC-TK-06

Use cases, transition table and `BR-TK-xx`: [02-use-cases.md](02-use-cases.md). Screens (`SCR-T-xx`, `DLG-T-xx`): [03-information-architecture.md](03-information-architecture.md). `BR-WF-xx` and `UC-WF-xx` come from F05 Workflow & Automation (branch `docs/workflow-requirements`).

Authentication, scope (BR-TK-01), validation, audit (BR-TK-14), optimistic concurrency (BR-TK-21) and `ProblemDetails` errors (`400`, `403`, `404`, `409`, `429`) are **shared acceptance constraints** of every story, not separate features.

## 1. Backlog

MoSCoW: **Must** (required for V1), **Should** (important), **Could** (if time allows).

| ID | User story | Value | Dependencies | Priority | UC |
|---|---|---|---|---|---|
| US-TK-01 | As a **Requester**, I want to create a ticket with a title and description, so that my IT problem is recorded and handled. | Single entry point for 8,000 requests/day | F01 login | Must | UC-TK-01 |
| US-TK-02 | As a **Requester**, I want to attach screenshots, logs and documents (including pasting a screenshot), so that the AI and the technician understand the problem without asking again. | Fewer clarification rounds | US-TK-01 | Must | UC-TK-01 |
| US-TK-03 | As a **Requester**, I want to create a ticket on behalf of a colleague, so that a new employee or someone without access can still get help. | Covers onboarding requests | US-TK-01, BR-WF-16 | Should | UC-TK-01 |
| US-TK-04 | As a **Requester**, I want to see similar open tickets and KB articles while I type, so that I can solve it myself or join an existing ticket instead of creating a duplicate. | Fewer duplicates | US-TK-01, F04, F06 | Could | UC-TK-01 |
| US-TK-05 | As a **Requester**, I want a *My tickets* list with plain-language statuses and what is expected from me, so that I always know where my requests stand. | Fewer "any update?" messages | US-TK-01 | Must | UC-TK-02 |
| US-TK-06 | As a **Technician / Team Lead**, I want work queues (assigned to me, my team unassigned, waiting for requester, due soon, triage), so that I know what to work on next. | Faster pick-up, fewer SLA breaches | F05 SLA, F09 teams | Must | UC-TK-02 |
| US-TK-07 | As a **staff user**, I want to search by code or keyword and filter / sort by status, priority, category, team, assignee, requester, date and SLA state, so that I can find any ticket in my scope among millions. | Findability | US-TK-06 | Must | UC-TK-02 |
| US-TK-08 | As a **staff user**, I want to save a filter set as a personal view, so that I can reopen my usual queue in one click. | Productivity | US-TK-07 | Could | UC-TK-02 |
| US-TK-09 | As any **user**, I want to open a ticket and see its conversation, details, attachments and history appropriate to my role, so that I have the full context without seeing what I shouldn't. | Context + confidentiality | US-TK-01 | Must | UC-TK-02 |
| US-TK-10 | As a **Team Lead / Admin**, I want to assign or reassign a ticket to a team member with a reason, seeing members' load and availability, so that work is balanced and nobody gets tickets while absent. | Balanced workload | F09, BR-WF-10 | Must | UC-TK-03 |
| US-TK-11 | As a **Technician**, I want to claim an unassigned ticket from my team queue, so that I can pick up work without waiting for my lead. | Faster pick-up | US-TK-06 | Should | UC-TK-03 |
| US-TK-12 | As a **Technician**, I want to transfer a ticket to another team with a reason, so that a misrouted ticket reaches the right team quickly (and the AI learns from it). | Fixes misrouting | F09 teams | Must | UC-TK-03 |
| US-TK-13 | As a **Technician**, I want to escalate a ticket to a higher support tier or to my team lead, with the work done so far, so that hard problems get expertise or management attention in time. | Fast resolution of complex issues | US-TK-12, support tiers (F09) | Must | UC-TK-03 |
| US-TK-14 | As a **Requester / Technician**, I want to exchange public replies with attachments in the ticket, and be notified of new replies, so that all communication is in one traceable place. | No lost e-mails / chats | US-TK-09 | Must | UC-TK-04 |
| US-TK-15 | As a **Technician**, I want to write internal notes (and @mention colleagues) that the requester cannot see, so that I can document troubleshooting and ask for help freely. | Knowledge capture, confidentiality | US-TK-14 | Must | UC-TK-04 |
| US-TK-16 | As a **Technician**, I want to ask the requester for missing information and have the SLA pause until they answer, and as a **Requester** I want my reply to resume the ticket automatically, so that waiting time is fair and nothing gets stuck. | Fair SLA, no stuck tickets | US-TK-14, BR-WF-07 | Must | UC-TK-04 |
| US-TK-17 | As a **Technician**, I want to start work on a ticket and have the status shown to the requester, so that they know someone is on it. | Transparency | US-TK-10 / US-TK-11 | Must | UC-TK-05 |
| US-TK-18 | As a **Technician**, I want to resolve a ticket with a resolution code and a note for the requester (or mark it as a duplicate), so that the requester can verify the fix and the data can be reused. | Verifiable outcome, KB input | US-TK-17 | Must | UC-TK-05 |
| US-TK-19 | As a **Technician / Team Lead**, I want to correct the category and priority with a reason, so that SLA, reports and future AI classification are accurate. | Data quality, AI learning | US-TK-17, F04 | Should | UC-TK-05 |
| US-TK-20 | As a **Requester**, I want to confirm that my ticket is solved and optionally rate the service, so that the ticket closes and the team gets feedback. | Closure + satisfaction data (F07) | US-TK-18 | Must | UC-TK-06 |
| US-TK-21 | As a **Requester**, I want to report "not fixed" or reopen a closed ticket within 7 days, so that I don't have to start over when the problem comes back. | Continuity | US-TK-20 | Must | UC-TK-06 |
| US-TK-22 | As a **Requester**, I want to cancel a request I no longer need, and as **staff** I want to cancel duplicates / spam with a reason, so that queues only contain real work. | Clean queues, accurate reports | US-TK-01 | Must | UC-TK-06 |

Grouped by use case:

```text
US-TK-01–04   UC-TK-01  Create ticket with evidence
US-TK-05–09   UC-TK-02  View, search, filter tickets
US-TK-10–13   UC-TK-03  Assign / transfer team / escalate
US-TK-14–16   UC-TK-04  Communicate & add information
US-TK-17–19   UC-TK-05  Update status & resolution
US-TK-20–22   UC-TK-06  Confirm / close / reopen / cancel
```

Totals: **22 stories** — 17 Must, 3 Should, 2 Could.

## 2. Acceptance criteria

### EPIC 1 — Create ticket with evidence (UC-TK-01)

**US-TK-01 — Create a ticket** · Must
- AC1: Given I am logged in, when I submit a title (5–200 chars) and description (10–5,000 chars), then a ticket is created in `NEW` with a code `TK-YYYY-NNNNNN`, I am its requester and beneficiary, and I am redirected to its detail page.
- AC2: Empty or out-of-range title / description → `400` with a message per field; the form keeps my input and attachments.
- AC3: The ticket is routed by Workflow (UC-WF-04) and the detail page shows the routing outcome (e.g. *"Approved automatically"*, *"The assistant needs more information"*, *"Assigned to Hardware team"*) within 15 s (P95, NFR-WF-PERF-001).
- AC4: Submitting twice with the same `Idempotency-Key` returns the same ticket and creates no duplicate.
- AC5: The 21st ticket within one hour from the same user → `429`.
- AC6: Audit `TICKET_CREATED` with `source = USER` exists for every ticket.

**US-TK-02 — Attach evidence** · Must
- AC1: I can add files by drag-and-drop, file picker or pasting an image from the clipboard; each shows upload progress and scan status.
- AC2: Allowed types only (BR-TK-03); a file > 10 MB, an 11th file in one message or a ticket total > 50 MB → rejected with a clear message, on the client and with `400` on the server.
- AC3: A file detected as malware is quarantined, is not linked to the ticket, cannot be downloaded by anyone, and I see *"This file was blocked for security reasons"*; audit `TICKET_ATTACHMENT_BLOCKED`.
- AC4: Downloading an attachment requires the ticket in my scope; the signed URL expires after 5 minutes; images show an inline preview.
- AC5: A user without scope who obtains an attachment URL after expiry gets `403`.

**US-TK-03 — Create on behalf of a colleague** · Should
- AC1: Ticking *"This request is for someone else"* lets me pick an active user; the ticket stores `beneficiary_id`.
- AC2: The beneficiary sees the ticket in *My tickets* labelled *"Requested by <name>"*, can read and reply, but cannot confirm, reopen or cancel (`403`).
- AC3: If the beneficiary is in another department than me, Workflow never auto-approves it (BR-WF-16); it goes to a technician.
- AC4: Picking a disabled or non-existent user → `400`.

**US-TK-04 — Similar tickets & KB suggestions** · Could
- AC1: After I type ≥ 3 words in the title, up to 5 similar open tickets **within my own scope** and 3 KB articles are shown within 2 s; I never see other people's tickets.
- AC2: Choosing *"Add my information to this ticket"* on one of my own open tickets opens it with my draft pre-filled in the reply box; no new ticket is created.
- AC3: If the suggestion service fails, the form still works and shows no error to me.

### EPIC 2 — View, search, filter (UC-TK-02)

**US-TK-05 — My tickets** · Must
- AC1: *My tickets* lists only tickets I created or am beneficiary of, newest activity first, with code, title, plain-language status (BR-TK-18), last update and an *"Action needed"* badge when the ticket is `NEEDS_INFO` or `RESOLVED`.
- AC2: Tabs / filters: *Open*, *Needs my action*, *Resolved & closed*, *Cancelled*; counts are shown per tab.
- AC3: Empty state for a new user shows a *Create ticket* button.
- AC4: Internal statuses (`TRIAGE`, `ASSIGNED`) both appear as *"Waiting for a technician"*; `AUTOMATING` appears as *"Being processed automatically"*.

**US-TK-06 — Work queues** · Must
- AC1: Technician default view is *Assigned to me*, sorted by SLA due soonest; views *My team — unassigned*, *Waiting for requester*, *Due soon (at risk)* and *Breached* are available.
- AC2: Team Lead additionally sees *Triage* (tickets in `TRIAGE`) and *Escalated to me*; Admin sees all views across all teams.
- AC3: Each row shows code, title, requester, priority, status, team, assignee, SLA badge (`ON_TRACK` / `AT_RISK` / `BREACHED` / `PAUSED`) and time left.
- AC4: Counts per view refresh at least every 60 s without reloading the page.
- AC5: A technician never sees a ticket that is neither assigned to them nor to one of their teams (verified via API, not only UI).

**US-TK-07 — Search, filter, sort** · Must
- AC1: Searching an exact ticket code returns that ticket (if in scope) as the only result.
- AC2: Keyword search matches title, description and **public** messages; internal notes are searchable only by staff.
- AC3: All filters listed in UC-TK-02 step 3 can be combined (AND); the URL reflects filters, sort and page, and reopening the URL restores the same result.
- AC4: Page size defaults to 25, max 100; asking for 500 → `400`.
- AC5: Date range from > to, or longer than 366 days → `400`.
- AC6: First page returns in ≤ 1.5 s (P95) on 3 million tickets (NFR-TK-PERF-001).
- AC7: If full-text search is unavailable, a banner explains it and filters still work.

**US-TK-08 — Saved views** · Could
- AC1: I can save, rename and delete up to 20 personal views; the 21st → `400`.
- AC2: Saved views are private to me and only store filter parameters, never results; scope is re-applied every time.

**US-TK-09 — Ticket detail** · Must
- AC1: Opening a ticket in scope shows header (code, title, status, priority, SLA), conversation, details panel and attachments.
- AC2: Opening an out-of-scope ticket by URL → `403` page; the response body contains no ticket data.
- AC3: A requester never receives internal notes, AI confidence, policy-check results, routing internals or technician-to-technician assignment events — verified on the API response, not only the UI (BR-TK-10).
- AC4: Staff see the AI summary, routing reason and flow (A/B/C), and the *Automation* tab when a workflow run exists.
- AC5: Only actions allowed for my role and the current status are displayed (BR-TK-04).

### EPIC 3 — Assign, transfer, escalate (UC-TK-03)

**US-TK-10 — Assign / reassign** · Must
- AC1: A Team Lead can assign a `TRIAGE` or unassigned ticket to their own team or one of its members; Admin to any team / member.
- AC2: The dialog shows each member's open ticket count and absence; disabled or absent members cannot be selected; forcing it via API → `400 ASSIGNEE_UNAVAILABLE`.
- AC3: Reassign without a reason → `400`; Team Lead reassigning outside their team → `403`.
- AC4: After reassignment the status is `ASSIGNED`, both old and new assignees are notified, and the audit record holds before/after assignee and the reason.
- AC5: The SLA deadline is unchanged by assign / reassign (BR-TK-06).
- AC6: Assign / reassign on an `AUTOMATING`, `RESOLVED`, `CLOSED` or `CANCELLED` ticket → `409`.

**US-TK-11 — Claim** · Should
- AC1: *Claim* is shown only on `ASSIGNED` tickets of my team with no individual assignee.
- AC2: When two technicians claim the same ticket simultaneously, exactly one succeeds; the other gets `409 ALREADY_ASSIGNED` and the list refreshes.
- AC3: Audit `TICKET_CLAIMED` with `source = USER`.

**US-TK-12 — Transfer to another team** · Must
- AC1: The assignee, the source team lead or Admin can transfer an `ASSIGNED` / `IN_PROGRESS` ticket with a reason code and a note (≥ 10 chars); missing → `400`.
- AC2: After transfer the ticket is `ASSIGNED` to the target team without an individual assignee; Workflow assigns a member by that team's strategy within 10 s (P95) or notifies the target lead.
- AC3: Transfer to the current team → `400`; the previous assignee can no longer change the ticket (`403`) but its history remains visible to them.
- AC4: Reason `WRONG_TEAM` stores a routing-correction record (AI category / team vs. human choice) consumable by F04 and F07.
- AC5: The SLA clock is not reset.

**US-TK-13 — Escalate** · Must
- AC1: *To higher support tier* lists only teams with a higher `support_tier` whose categories cover the ticket; at L3 this option is disabled with the hint *"Escalate to team lead instead"*; API call → `400`.
- AC2: Functional escalation requires reason and *work done so far* (≥ 20 chars, saved as an internal note); `escalation_level` increases by 1 and never decreases.
- AC3: *To team lead* keeps the current assignee, sets `is_escalated_to_lead = true`, and the lead receives a notification within 1 minute and sees the ticket in *Escalated to me*.
- AC4: Audit `TICKET_ESCALATED` records type, from/to team, level and reason; F07 can count escalations (metric M-09) separating manual (`USER`) from SLA (`AUTOMATION`) escalations.

### EPIC 4 — Communicate and add information (UC-TK-04)

**US-TK-14 — Public replies** · Must
- AC1: Requester, beneficiary and staff in scope can post a public reply (≤ 10,000 chars) with up to 10 attachments on a non-final ticket.
- AC2: Staff public reply → requester and beneficiary notified (in-app + e-mail); requester reply → assignee (or team) notified; delivery in-app ≤ 10 s.
- AC3: The first public reply by a technician sets `first_responded_at` if not set yet.
- AC4: Replying on a `CLOSED` / `CANCELLED` ticket → `409`; the UI offers *Reopen* (within window) or *Create follow-up ticket*.
- AC5: Messages cannot be edited or deleted by their authors (BR-TK-13).

**US-TK-15 — Internal notes & mentions** · Must
- AC1: Staff can switch the composer to *Internal note*; internal notes are visually distinct (e.g. yellow background, lock icon).
- AC2: A requester calling the API with `visibility = INTERNAL` → `403`; requester API responses never contain internal notes (contract test).
- AC3: `@mention` of a technician outside the ticket's team notifies them and grants read access to that ticket; the grant is audited.
- AC4: Admin can redact a message with a reason; the content becomes *"[Removed by administrator: reason]"* for everyone and the original is purged.

**US-TK-16 — Ask requester / resume** · Must
- AC1: The assignee of an `IN_PROGRESS` ticket can *Ask requester* with a question (≥ 10 chars); the ticket becomes `NEEDS_INFO` with `waiting_on = TECHNICIAN`, the SLA pauses, and the requester is notified.
- AC2: The requester's next public reply (or attachment) automatically moves the ticket back to `IN_PROGRESS` with the same assignee and resumes the SLA; pause and resume timestamps are stored.
- AC3: If `waiting_on = AI`, the requester's reply is treated as a clarification answer and re-routed (UC-WF-04).
- AC4: The assignee can *Resume work* without an answer only with a reason.
- AC5: Reminders on day 1 and N−1 and auto-close on day N follow UC-WF-06; the requester can reopen within 7 days.

### EPIC 5 — Update status and resolution (UC-TK-05)

**US-TK-17 — Start work** · Must
- AC1: Only the individual assignee sees *Start work* on an `ASSIGNED` ticket; it moves to `IN_PROGRESS` and sets `started_at`; requester sees *"In progress"*.
- AC2: A ticket without an individual assignee cannot be started (`409`) — claim or assign first.
- AC3: Two users updating the same ticket version → the second gets `409 STALE_VERSION` and sees the latest state.

**US-TK-18 — Resolve** · Must
- AC1: *Resolve* requires a resolution code and a requester-visible note (≥ 20 chars); missing → `400`.
- AC2: `DUPLICATE` requires an existing, non-cancelled original ticket code; a `DUPLICATE_OF` link is created and the requester sees it.
- AC3: After resolving: status `RESOLVED`, `resolved_at` / `resolved_by` set, requester notified with the note and *Confirm / Not fixed* buttons, SLA resolution clock stopped.
- AC4: *Resolve* is not available on `AUTOMATING` tickets (`409`); those are resolved by Workflow verification or *Resolved manually* (UC-WF-09).
- AC5: Ticking *Suggest a KB article* creates a draft suggestion in F06 linked to the ticket.

**US-TK-19 — Correct category and priority** · Should
- AC1: Changing category or priority without a reason → `400`.
- AC2: A technician can change the category of an assigned ticket; priority changes by a technician are *proposals* that the team lead approves; Team Lead / Admin change priority directly.
- AC3: A priority change re-binds the ticket to the SLA policy version in force for the new priority (BR-WF-08) and recalculates the deadline; raising to P1 notifies the team lead.
- AC4: The AI-suggested and the human-chosen values are both kept (`category_source`, audit before/after) for F04 learning and F07 reports.

### EPIC 6 — Confirm, close, reopen, cancel (UC-TK-06)

**US-TK-20 — Confirm & rate** · Must
- AC1: On a `RESOLVED` ticket, the requester sees *"Yes, it's solved"* and *"No, still a problem"*.
- AC2: *Yes* → `CLOSED` with `close_reason = CONFIRMED`; an optional 1–5 rating and comment are stored; rating outside 1–5 → `400`.
- AC3: Without a response, reminders are sent on day 1 and N−1 and the ticket closes on day N (default 5 business days) with `close_reason = AUTO_CONFIRMED`, `source = AUTOMATION`; `AUTOMATING` tickets are never auto-closed.
- AC4: The beneficiary or a technician calling confirm → `403`; Admin may confirm on behalf with a reason (audit shows `on_behalf_of`).

**US-TK-21 — Not fixed / reopen** · Must
- AC1: *Not fixed* on `RESOLVED`, or *Reopen* on `CLOSED` within 7 days, requires a reason (≥ 10 chars) and may include evidence.
- AC2: The ticket returns to `IN_PROGRESS` with the previous assignee if available, otherwise to `ASSIGNED` for the owning team; `reopen_count` increases; the assignee is notified.
- AC3: A ticket auto-closed for no response reopens to `NEW` and is routed again.
- AC4: Reopen after 7 days → `409 REOPEN_WINDOW_EXPIRED`; the UI shows *Create follow-up ticket*, which pre-fills a new ticket with a `FOLLOW_UP_OF` link.
- AC5: The 3rd reopen automatically escalates the ticket to the team lead.

**US-TK-22 — Cancel** · Must
- AC1: The requester can cancel their own ticket in `NEW`, `NEEDS_INFO`, `TRIAGE` or `ASSIGNED`; in `IN_PROGRESS` the button becomes *Ask to cancel* (posts a public message); a direct API cancel from `IN_PROGRESS` → `409`.
- AC2: Staff (assignee, team lead, Admin) can cancel with a reason code and note (≥ 10 chars); `DUPLICATE` needs a valid original ticket code.
- AC3: Cancelling an `AUTOMATING` ticket → `409` with an explanatory message.
- AC4: `CANCELLED` is final: replies, reopen and any status change → `409`.
- AC5: The requester (when staff cancels) or the assignee (when the requester cancels) is notified with the reason; F07 counts `CANCELLED` tickets in volume only.

## 3. INVEST review

`PASS` means the design gives enough information for planning and acceptance; it does **not** mean code or tests exist.

| Story | I | N | V | E | S | T | Notes |
|---|---|---|---|---|---|---|---|
| US-TK-01 | PASS | PASS | PASS | PASS | PASS | PASS | Routing outcome comes from F05; demoable with a stub router. |
| US-TK-02 | PASS | PASS | PASS | PASS | PASS | PASS | Scanner can be a stub (EICAR test file) in V1 demo. |
| US-TK-03 | PASS | PASS | PASS | PASS | PASS | PASS | Interacts with BR-WF-16. |
| US-TK-04 | NEEDS DEP | PASS | PASS | NEEDS DEP | PASS | PASS | Depends on F04 similarity and F06 search APIs. |
| US-TK-05 | PASS | PASS | PASS | PASS | PASS | PASS | Label mapping BR-TK-18. |
| US-TK-06 | PASS | PASS | PASS | PASS | PASS | PASS | SLA state read from F05 timers. |
| US-TK-07 | PASS | PASS | PASS | PASS | NEEDS SPLIT | PASS | Can split: filters (DB) / full-text search. |
| US-TK-08 | PASS | PASS | PASS | PASS | PASS | PASS | Stores parameters only. |
| US-TK-09 | PASS | PASS | PASS | PASS | PASS | PASS | Visibility verified on API DTOs. |
| US-TK-10 | PASS | PASS | PASS | PASS | PASS | PASS | Reuses BR-WF-10 availability. |
| US-TK-11 | PASS | PASS | PASS | PASS | PASS | PASS | Race handled with version check. |
| US-TK-12 | PASS | PASS | PASS | PASS | PASS | PASS | Auto-assignment in target team via UC-WF-05. |
| US-TK-13 | PASS | NEEDS CONFIRM | PASS | PASS | PASS | PASS | Support tiers pending Q-TK-03. |
| US-TK-14 | PASS | PASS | PASS | PASS | PASS | PASS | |
| US-TK-15 | PASS | PASS | PASS | PASS | PASS | PASS | Mention grants read-only access. |
| US-TK-16 | PASS | NEEDS CONFIRM | PASS | PASS | PASS | PASS | `waiting_on` pending Q-TK-02. |
| US-TK-17 | PASS | PASS | PASS | PASS | PASS | PASS | |
| US-TK-18 | PASS | PASS | PASS | PASS | PASS | PASS | Resolution codes listed in UC-TK-05. |
| US-TK-19 | PASS | PASS | PASS | PASS | PASS | PASS | Priority approval flow for technicians. |
| US-TK-20 | PASS | NEEDS CONFIRM | PASS | PASS | PASS | PASS | Auto-confirm pending Q-TK-01 (BR-WF-11). |
| US-TK-21 | PASS | PASS | PASS | PASS | PASS | PASS | 7-day window shared with F05 / M-04. |
| US-TK-22 | PASS | NEEDS CONFIRM | PASS | PASS | PASS | PASS | Cancel from `IN_PROGRESS` pending Q-TK-04. |

## 4. Traceability

| Story | UC | Screen / dialog | API operationId (proposed) | Tables | Rules |
|---|---|---|---|---|---|
| US-TK-01 | UC-TK-01 | SCR-T-01 | `createTicket` | `tickets`, `audit_logs` | BR-TK-02, 19 |
| US-TK-02 | UC-TK-01 | SCR-T-01, SCR-T-02 | `uploadAttachment`, `getAttachmentDownloadUrl` | `ticket_attachments` | BR-TK-03 |
| US-TK-03 | UC-TK-01 | SCR-T-01 | `createTicket` (`beneficiaryId`), `searchUsers` (F09) | `tickets` | BR-TK-22, BR-WF-16 |
| US-TK-04 | UC-TK-01 | SCR-T-01 | `suggestSimilarTickets` (F04), `searchKbArticles` (F06) | — | BR-TK-01 |
| US-TK-05 | UC-TK-02 | SCR-T-03 | `listTickets` (`scope=mine`) | `tickets` | BR-TK-01, 18 |
| US-TK-06 | UC-TK-02 | SCR-T-04 | `listTickets` (`view=…`), `getTicketViewCounts` | `tickets`, `ticket_sla_timers` | BR-TK-01 |
| US-TK-07 | UC-TK-02 | SCR-T-04 | `listTickets` (`q`, filters, sort) | `tickets`, search index | BR-TK-01, 10 |
| US-TK-08 | UC-TK-02 | SCR-T-04 | `listSavedViews`, `createSavedView`, `updateSavedView`, `deleteSavedView` | `saved_views` | — |
| US-TK-09 | UC-TK-02 | SCR-T-02 | `getTicket`, `listTicketMessages`, `listTicketHistory` (F08) | `tickets`, `ticket_messages` | BR-TK-01, 10, 18 |
| US-TK-10 | UC-TK-03 | DLG-T-01, SCR-WF-10 | `assignTicket`, `reassignTicket`, `listAssignableMembers` | `ticket_assignments` | BR-TK-05, 06, 15 |
| US-TK-11 | UC-TK-03 | SCR-T-04, SCR-T-02 | `claimTicket` | `ticket_assignments` | BR-TK-21 |
| US-TK-12 | UC-TK-03 | DLG-T-02 | `transferTicket` | `ticket_assignments`, `routing_corrections` | BR-TK-05, 06 |
| US-TK-13 | UC-TK-03 | DLG-T-03 | `escalateTicket` | `ticket_escalations` | BR-TK-16 |
| US-TK-14 | UC-TK-04 | SCR-T-02 | `addTicketMessage` (`PUBLIC`) | `ticket_messages`, `notifications` | BR-TK-13 |
| US-TK-15 | UC-TK-04 | SCR-T-02 | `addTicketMessage` (`INTERNAL`), `redactTicketMessage` | `ticket_messages`, `ticket_access_grants` | BR-TK-10, 13 |
| US-TK-16 | UC-TK-04 | DLG-T-04 | `requestTicketInfo`, `resumeTicketWork`, `answerClarification` (F05) | `tickets`, `sla_events` | BR-TK-06 |
| US-TK-17 | UC-TK-05 | SCR-T-02 | `startTicketWork` | `tickets`, `ticket_status_history` | BR-TK-09, 21 |
| US-TK-18 | UC-TK-05 | DLG-T-05 | `resolveTicket` | `tickets`, `ticket_links` | BR-TK-07 |
| US-TK-19 | UC-TK-05 | DLG-T-09 | `updateTicketClassification` | `tickets`, `routing_corrections` | BR-TK-05, BR-WF-08 |
| US-TK-20 | UC-TK-06 | DLG-T-08 | `confirmTicketResolution` | `tickets`, `ticket_ratings` | BR-TK-12, 22 |
| US-TK-21 | UC-TK-06 | DLG-T-07 | `reopenTicket` | `tickets` | BR-TK-11 |
| US-TK-22 | UC-TK-06 | DLG-T-06 | `cancelTicket` | `tickets` | BR-TK-08, 17 |
