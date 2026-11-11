# User Stories & Acceptance Criteria — F03 Ticket Management

> Design status: TARGET V1
>
> Implementation status: NOT IMPLEMENTED — all criteria are acceptance targets
>
> Scope: US-TK-01 … US-TK-24 for UC-TK-01 … UC-TK-07 (the seven leaves of the **Ticket Management** branch of the mind map)

Use cases, transition table and `BR-TK-xx`: [02-use-cases.md](02-use-cases.md). Screens (`SCR-T-xx`, `DLG-T-xx`, `SEC-T-xx`, `EML-T-xx`): [information-architecture/](information-architecture/README.md). `BR-WF-xx` and `UC-WF-xx` come from [F05](../f05-workflow-automation/README.md); `UC-AI-xx` from [F04](../f04-ai-ticket-processing/README.md).

Authentication, scope (BR-TK-01), validation, audit (BR-TK-14), optimistic concurrency (BR-TK-21) and `ProblemDetails` errors (`400`, `403`, `404`, `409`, `429`) are **shared acceptance constraints** of every story, not separate features.

## 1. Backlog

MoSCoW: **Must** (required for V1), **Should** (important), **Could** (if time allows).

| ID | User story | Value | Dependencies | Priority | UC |
|---|---|---|---|---|---|
| US-TK-01 | As a **Requester**, I want to raise a ticket by sending an email to the IT support mailbox in my own words, so that I can ask for help the way I already do, without a portal or a form. | Most of the 8,000 tickets/day arrive by email | Mailbox connector, F05 routing | Must | UC-TK-01 |
| US-TK-02 | As a **Requester**, I want to raise a ticket on the portal by describing my problem in free text, without choosing a category or filling a category form, so that I cannot pick the wrong category or give up. | Less rigid than the ManageEngine category form | F01 login, F05 routing | Must | UC-TK-01 |
| US-TK-03 | As a **Requester**, I want to attach screenshots, logs and documents (paste a screenshot on the portal, attach files to the email), so that the AI and the technician understand the problem without asking again. | Fewer clarification rounds | US-TK-01, US-TK-02 | Must | UC-TK-01 |
| US-TK-04 | As a **Requester**, I want to raise a ticket on behalf of a colleague, so that a new employee or someone without access can still get help. | Covers onboarding requests | US-TK-02, BR-WF-16 | Should | UC-TK-01 |
| US-TK-05 | As a **Requester**, I want a *My tickets* list with plain-language statuses and what is expected from me, so that I always know where my requests stand. | Fewer "any update?" messages | US-TK-01, US-TK-02 | Must | UC-TK-02 |
| US-TK-06 | As a **Technician / Admin**, I want work queues (assigned to me, my sub-group queue, on hold, due soon, breached, escalated to me), so that I know what to work on next. | Faster pick-up, fewer SLA breaches | F05 SLA and assignment, F09 | Must | UC-TK-02 |
| US-TK-07 | As a **staff user**, I want to search by code or keyword and filter / sort by status, priority, source, support level, category, group, sub-group, owner, requester, date and SLA state, so that I can find any ticket in my scope among millions. | Findability | US-TK-06 | Must | UC-TK-02 |
| US-TK-08 | As a **staff user**, I want to save a filter set as a personal view, so that I can reopen my usual queue in one click. | Productivity | US-TK-07 | Could | UC-TK-02 |
| US-TK-09 | As any **user**, I want to open a ticket and see its conversation, fields, attachments and history appropriate to my role, so that I have the full context without seeing what I shouldn't. | Context + confidentiality | US-TK-01, US-TK-02 | Must | UC-TK-02 |
| US-TK-10 | As a **Technician**, I want a short summary at the top of every ticket handed to me (problem, impact, key details, what was tried, why it came to me), also in the handoff notification, so that I can start without reading the whole conversation. | Less reading time per ticket | F05 routing, AI Service | Must | UC-TK-03 |
| US-TK-11 | As a **Technician**, I want the summary to update when new information arrives and to regenerate it on demand, so that it never describes an outdated situation. | Current information | US-TK-10 | Should | UC-TK-03 |
| US-TK-12 | As an **Admin** deciding on an approval, I want the summary of the ticket with the requested action and why approval is needed, so that I can decide without reading the conversation. | Fast, informed approvals | US-TK-10, Q-TK-10 | Should | UC-TK-03 |
| US-TK-13 | As a **Technician / Admin**, I want to transfer a ticket to another sub-group or technician of the same group with a reason, so that it reaches the right person without leaving the division. | Fixes in-group misassignment | F09 groups, F05 UC-WF-05 | Must | UC-TK-04 |
| US-TK-14 | As a **Technician / Admin**, I want to escalate a ticket to a higher-tier sub-group of the same group, or to an Admin, with the work done so far, so that hard problems get expertise or management attention in time. | Fast resolution of complex issues | US-TK-13, support tiers (F09) | Must | UC-TK-04 |
| US-TK-15 | As a **Requester**, I want to add the missing information to an on-hold ticket by replying on the portal or to the email, and have the ticket resume by itself, so that nothing stays stuck waiting for me. | No stuck tickets | US-TK-01, F05 flow B | Must | UC-TK-05 |
| US-TK-16 | As a **Technician**, I want to put a ticket on hold with a question to the requester, and have the SLA pause until they answer, so that waiting time is fair. | Fair SLA | US-TK-15, BR-WF-07 | Must | UC-TK-05 |
| US-TK-17 | As a **Requester / Technician**, I want to exchange public replies with attachments in the ticket (portal or email), and be notified of new replies, so that all communication is in one traceable place. | No lost emails / chats | US-TK-09 | Must | UC-TK-05 |
| US-TK-18 | As a **Technician**, I want to write internal notes (and @mention colleagues) that the requester cannot see, so that I can document troubleshooting and ask for help freely. | Knowledge capture, confidentiality | US-TK-17 | Must | UC-TK-05 |
| US-TK-19 | As a **Technician**, I want to start work on a ticket and have the status shown to the requester, so that they know someone is on it. | Transparency | F05 assignment | Must | UC-TK-06 |
| US-TK-20 | As a **Technician**, I want to resolve a ticket with a resolution code, the Resolution and a reply to the requester drafted by the AI (or mark it as a duplicate), so that the requester can verify the fix and the knowledge can be reused. | Verifiable outcome, KB input | US-TK-19, F04 UC-AI-08 | Must | UC-TK-06 |
| US-TK-21 | As a **Technician**, I want to link related tickets and resolve them together, so that one problem affecting several people is handled once. | Less duplicate work | US-TK-20, F05 UC-WF-10 | Should | UC-TK-06 |
| US-TK-22 | As a **Requester**, I want to confirm that my ticket is solved and optionally rate the service, so that the ticket closes and the team gets feedback. | Closure + satisfaction data (F07) | US-TK-20 | Must | UC-TK-07 |
| US-TK-23 | As a **Requester**, I want to report "not fixed" or reopen a closed ticket within 7 days, so that I don't have to start over when the problem comes back. | Continuity | US-TK-22 | Must | UC-TK-07 |
| US-TK-24 | As a **Requester**, I want to cancel a request I no longer need, and as **staff** I want to cancel spam / test tickets with a reason, so that queues only contain real work. | Clean queues, accurate reports | US-TK-01, US-TK-02 | Must | UC-TK-07 |

Grouped by use case:

```text
US-TK-01–04   UC-TK-01  Raise ticket by email or free text, no category form
US-TK-05–09   UC-TK-02  View / search / filter tickets
US-TK-10–12   UC-TK-03  View ticket summary on handoff and approval
US-TK-13–14   UC-TK-04  Transfer / escalate within the same division
US-TK-15–18   UC-TK-05  Add missing information to on-hold tickets
US-TK-19–21   UC-TK-06  Update status and resolution
US-TK-22–24   UC-TK-07  Confirm / close / reopen / cancel ticket
```

Totals: **24 stories** — 19 Must, 4 Should, 1 Could.

## 2. Acceptance criteria

### EPIC 1 — Raise ticket by email or free text (UC-TK-01)

**US-TK-01 — Raise a ticket by email** · Must
- AC1: Given an email from a directory user to the support mailbox, then within 2 minutes (P95) a ticket exists in `NEW` with `source_channel = EMAIL`, title = subject, description = body, requester = sender, and the original email is kept for staff (NFR-TK-PERF-004).
- AC2: The sender receives the acknowledgement EML-T-01 with `[TK-YYYY-NNNNNN]` in the subject and a link to the ticket.
- AC3: A reply to that email (code in subject or matching `In-Reply-To`) is added to the same ticket and creates no new ticket (BR-TK-23).
- AC4: Reading the same `Message-ID` twice creates no duplicate; auto-replies and bounces create no ticket and get no acknowledgement.
- AC5: A sender not in the directory still gets a ticket; F05 receives it with `UNKNOWN_SENDER` and it is never automated.
- AC6: Audit `TICKET_CREATED` with `source = AUTOMATION` exists for every email ticket.

**US-TK-02 — Raise a ticket in free text on the portal** · Must
- AC1: SCR-T-01 has a description box (10–5,000 chars), an optional subject (≤ 200 chars) and attachments — **no category, request type, urgency or priority field and no category form** (BR-TK-24).
- AC2: Submitting creates a ticket in `NEW` with `source_channel = PORTAL` and code `TK-YYYY-NNNNNN`; an empty subject is replaced by the first line of the description.
- AC3: The detail page shows the F05 outcome (e.g. *"The assistant needs more information"*, *"Assigned to IT Helpdesk — Printers"*) within 15 s (P95); the fields F05 filled are marked *"Filled by AI"* for staff.
- AC4: Empty or out-of-range description → `400` with a field message; input and attachments are kept.
- AC5: The same `Idempotency-Key` returns the same ticket; the 21st ticket within one hour → `429`.

**US-TK-03 — Attach evidence** · Must
- AC1: On the portal I can add files by drag-and-drop, file picker or pasting an image; each shows progress and scan status. Email attachments are imported with the same rules.
- AC2: Allowed types and limits only (BR-TK-03); a rejected file gives a clear message on the portal (`400` on the server) and is listed as *not accepted* in the email acknowledgement.
- AC3: A file detected as malware is quarantined, never linked or downloadable; I see *"This file was blocked for security reasons"*; audit `TICKET_ATTACHMENT_BLOCKED`.
- AC4: Downloading requires the ticket in scope; the signed URL expires after 5 minutes.

**US-TK-04 — Raise on behalf of a colleague** · Should
- AC1: Portal: ticking *"This request is for someone else"* lets me pick an active user; email: F05 extracts `requestedFor` from the text. The ticket stores `beneficiary_id`.
- AC2: The beneficiary sees the ticket labelled *"Requested by {name}"*, can read and reply, but cannot confirm, reopen or cancel (`403`).
- AC3: A beneficiary in another department is never auto-approved (BR-WF-16).
- AC4: Picking a disabled or non-existent user → `400`.

### EPIC 2 — View, search, filter (UC-TK-02)

**US-TK-05 — My tickets** · Must
- AC1: *My tickets* lists only tickets I raised or am beneficiary of, newest activity first, with code, title, plain-language status (BR-TK-18), last update and an *"Action needed"* badge when `NEEDS_INFO` or `RESOLVED`.
- AC2: Tabs: *Open*, *Needs my action*, *Resolved & closed*, *Cancelled*, with counts; tickets raised by email appear the same way.
- AC3: `ASSIGNED` appears as *"Waiting for a technician"*, `AUTOMATING` as *"Being processed automatically"*, a ticket owned by the AI agent as *"The AI Assistant is handling it"*.

**US-TK-06 — Work queues** · Must
- AC1: Technician default view is *Assigned to me*, sorted by SLA due soonest; views *My sub-group queue*, *On hold*, *Due soon* and *Breached* exist. Claiming from the queue is done on the F02 workbench.
- AC2: Admin additionally sees *Escalated to me* and *All open tickets*, across all groups. There is no triage view.
- AC3: Each row shows code, title, source, requester, priority (P1–P3), status, group / sub-group, owner (AI agent or technician), SLA badge and time left.
- AC4: Counts refresh at least every 60 s without reloading.
- AC5: A technician never sees a ticket that is neither assigned to them nor to one of their sub-groups (verified via API).

**US-TK-07 — Search, filter, sort** · Must
- AC1: Searching an exact ticket code returns that ticket (if in scope) as the only result.
- AC2: Keyword search matches title, description and **public** messages; internal notes are searchable only by staff.
- AC3: All filters in UC-TK-02 step 3 can be combined; the URL reflects filters, sort and page.
- AC4: Page size defaults to 25, max 100; 500 → `400`. Date range from > to, or > 366 days → `400`.
- AC5: First page in ≤ 1.5 s (P95) on 3 million tickets (NFR-TK-PERF-001).
- AC6: If full-text search is down, a banner explains it and filters still work.

**US-TK-08 — Saved views** · Could
- AC1: I can save, rename and delete up to 20 personal views; the 21st → `400`.
- AC2: Saved views store filter parameters only; scope is re-applied every time.

**US-TK-09 — Ticket detail** · Must
- AC1: Opening a ticket in scope shows header (code, title, status, priority, SLA, source, owner), conversation, details panel and attachments.
- AC2: An out-of-scope ticket → `403` page; the response contains no ticket data.
- AC3: A requester never receives internal notes, the summary, AI confidence, policy-check results or routing internals — verified on the API response (BR-TK-10).
- AC4: Staff see the summary (SEC-T-01), the AI-filled fields with their marker, the routing history and, for `EMAIL` tickets, the original email.
- AC5: Only actions allowed for my role, the owner and the current status are displayed (BR-TK-04).

### EPIC 3 — Ticket summary on handoff and approval (UC-TK-03)

**US-TK-10 — Summary on handoff** · Must
- AC1: Given F05 assigns a ticket to me, or it is transferred, escalated, re-routed or reopened to me, when I open it, then a summary of at most 150 words is on top in the sections Problem, Impact, Key details, Already tried, Questions and answers, Why it came to you, Open points; an empty section shows "None".
- AC2: The handoff notification contains the summary, or *"Summary is being prepared"* if it is not ready; it appears on the ticket within 30 s.
- AC3: Clicking a fact scrolls to the message it comes from; no fact is absent from the ticket (manual review of 50 summaries: 0 invented facts, ≥ 90% rated accurate).
- AC4: When the AI agent escalated the ticket, *Already tried* lists what the agent did (KB reply, guidance steps, run result).
- AC5: A requester calling the summary API → `403`; requester ticket responses contain no summary.
- AC6: AI unavailable → *"Summary unavailable"* and the original description; the handoff still happens.

**US-TK-11 — Up-to-date summary** · Should
- AC1: Given summary version 1, when the requester adds a message, then within 30 s version 2 is shown marked *"Updated after a new message"*; version 1 can still be opened.
- AC2: *Regenerate* creates a new version, unless the last one is < 30 s old (then it is returned without a model call).
- AC3: A technician outside the scope calling *Regenerate* → `403`.
- AC4: I can mark a summary helpful / not helpful; the vote is stored for F04.8 and F07.

**US-TK-12 — Summary on approval** · Should
- AC1: Given a ticket waiting for a human approval decision, when the approver opens it or the approval request, then the summary includes **Requested action** (operation, system, entitlement, duration, beneficiary) and **Why approval is needed** (the failed policy condition).
- AC2: The approval request notification carries the summary.
- AC3: The approval decision and its outcome are recorded by their owner and visible in F08; F03 does not change them.

### EPIC 4 — Transfer / escalate within the same division (UC-TK-04)

**US-TK-13 — Transfer within the group** · Must
- AC1: The assignee or an Admin can transfer an `ASSIGNED` / `IN_PROGRESS` ticket owned by a technician to another sub-group or technician **of the same group**, with a reason code and a note (≥ 10 chars); missing → `400`.
- AC2: The target list shows only sub-groups and technicians of the ticket's group; technicians `OFF` or at maximum load are disabled; forcing one via API → `400 ASSIGNEE_UNAVAILABLE`; a target in another group → `400 OUTSIDE_DIVISION`.
- AC3: Transfer to a sub-group without a person → F05 picks a member within 10 s (P95) or the ticket waits in the sub-group queue.
- AC4: The new owner is notified with the summary; the previous assignee keeps read access through history but can no longer change the ticket (`403`).
- AC5: The dialog offers *Return for re-routing* (F05 SCR-WF-24) for a ticket that belongs to another group.
- AC6: The SLA clock is not reset; on `NEW`, `NEEDS_INFO`, `AUTOMATING`, a final status, or a ticket owned by the AI agent → `409`.

**US-TK-14 — Escalate** · Must
- AC1: *Higher tier* lists only sub-groups of the same group with a higher `support_tier` covering the ticket's subcategory; at the top tier the option is disabled with *"Escalate to Admin instead"*; API call → `400`.
- AC2: Escalation requires a reason and *work done so far* (≥ 20 chars, saved as an internal note); `escalation_level` increases by 1 and never decreases.
- AC3: *To Admin* keeps the owner, sets `is_escalated_to_admin = true`; Admins are notified within 1 minute with the summary and see the ticket in *Escalated to me*.
- AC4: Audit `TICKET_ESCALATED` records type, from/to sub-group, level and reason; F07 can separate manual (`USER`) from SLA (`AUTOMATION`) escalations.

### EPIC 5 — Add missing information to on-hold tickets (UC-TK-05)

**US-TK-15 — Requester adds the missing information** · Must
- AC1: An on-hold ticket (`NEEDS_INFO`) shows the question and *"Waiting for your reply"* on SCR-T-02 and in *Needs my action*; the question is also emailed (EML-T-02).
- AC2: Replying on the portal or to the email (text and/or files) stores the answer and resumes the ticket: `waiting_on = TECHNICIAN` → `IN_PROGRESS` with the same assignee and the SLA resumed; `waiting_on = AI` → passed to F05 (`answerClarification`) and re-routed.
- AC3: Pause and resume timestamps are stored; the owner is notified; the summary is refreshed.
- AC4: An email reply from an address that is not the requester, the beneficiary or staff in scope is held for staff review and does not resume the ticket.

**US-TK-16 — Technician puts a ticket on hold** · Must
- AC1: The assignee of an `IN_PROGRESS` ticket can *Ask requester* with a question (≥ 10 chars); the ticket becomes `NEEDS_INFO` with `waiting_on = TECHNICIAN`, the SLA pauses, the requester is notified.
- AC2: The assignee can *Resume work* without an answer only with a reason.
- AC3: Reminders on day 1 and N−1 and auto-close on day N follow UC-WF-06; the requester can reopen within 7 days.

**US-TK-17 — Public replies** · Must
- AC1: Requester, beneficiary and staff in scope can post a public reply (≤ 10,000 chars, ≤ 10 attachments) on a non-final ticket, on the portal or by email.
- AC2: Staff reply → requester and beneficiary notified (in-app + email); requester reply → owner notified; in-app delivery ≤ 10 s.
- AC3: The first public reply by a technician sets `first_responded_at`.
- AC4: Replying on `CLOSED` / `CANCELLED` on the portal → `409` with *Reopen* or *Create follow-up ticket*; by email see UC-TK-01 A2.
- AC5: Messages cannot be edited or deleted by their authors (BR-TK-13).

**US-TK-18 — Internal notes & mentions** · Must
- AC1: Staff can switch the composer to *Internal note*; notes are visually distinct (lock icon and label).
- AC2: A requester calling the API with `visibility = INTERNAL` → `403`; requester responses and emails never contain internal notes (contract test).
- AC3: `@mention` of a technician outside the sub-group notifies them and grants read access to that ticket; the grant is audited.
- AC4: Admin can redact a message with a reason; it shows *"[Removed by administrator: reason]"* and the original is purged.

### EPIC 6 — Update status and resolution (UC-TK-06)

**US-TK-19 — Start work** · Must
- AC1: Only the individual assignee sees *Start work* on an `ASSIGNED` ticket; it moves to `IN_PROGRESS` and sets `started_at`; the requester sees *"In progress"*.
- AC2: A ticket without an individual assignee cannot be started (`409`).
- AC3: Two users updating the same version → the second gets `409 STALE_VERSION`.

**US-TK-20 — Resolve with the Resolution** · Must
- AC1: *Resolve* opens with the Resolution and the reply drafted by F04 (marked *"Drafted by AI"*); the technician can edit both; a resolution code, the Resolution and a reply ≥ 20 chars are required (`400`).
- AC2: `DUPLICATE` requires an existing, non-cancelled original ticket code; a `DUPLICATE_OF` link is created.
- AC3: After resolving: `RESOLVED`, `resolved_at` / `resolved_by` set, Resolution stored, the reply sent with *Confirm / Not fixed* (EML-T-03), SLA resolution clock stopped.
- AC4: When F04 resolves a ticket, it uses the same operation with author `AI` and a code among `ANSWERED_BY_KB`, `SOLVED_BY_KB`, `GUIDED_BY_AI`, `ACCESS_GRANTED`; the same Resolution rules apply.
- AC5: *Resolve* is not available on `AUTOMATING` tickets (`409`).
- AC6: Ticking *Suggest a KB article* creates a draft suggestion in F06 linked to the ticket.

**US-TK-21 — Link related tickets** · Should
- AC1: Staff can link two tickets in scope as `RELATED_TO` and unlink them with a reason; F05 links (UC-WF-10) appear in the same panel.
- AC2: Resolving a main ticket offers *Resolve the related tickets too*; each related ticket gets its own reply and its own confirmation.
- AC3: A requester sees only the links to tickets in their own scope; other tickets show code and status only, never content (BR-AI-08).

### EPIC 7 — Confirm, close, reopen, cancel (UC-TK-07)

**US-TK-22 — Confirm & rate** · Must
- AC1: On a `RESOLVED` ticket, the requester sees *"Yes, it's solved"* and *"No, still a problem"*, on SCR-T-02 and as links in EML-T-03 (the link opens the dialog; nothing changes until I click).
- AC2: *Yes* → `CLOSED` with `close_reason = CONFIRMED`; an optional 1–5 rating and comment are stored; outside 1–5 → `400`.
- AC3: Without a response, reminders on day 1 and N−1 and closure on day N (default 5 business days) with `close_reason = AUTO_CONFIRMED`; `AUTOMATING` tickets are never auto-closed.
- AC4: The beneficiary or a technician calling confirm → `403`; Admin may confirm on behalf with a reason (`on_behalf_of`).

**US-TK-23 — Not fixed / reopen** · Must
- AC1: *Not fixed* on `RESOLVED`, or *Reopen* on `CLOSED` within 7 days, requires a reason (≥ 10 chars) and may include evidence.
- AC2: The ticket returns to `IN_PROGRESS` with the previous assignee if `ON`, otherwise `ASSIGNED` for F05 to pick a member of the same sub-group; a ticket resolved by the AI agent goes to a technician; `reopen_count` increases; the new owner gets the summary.
- AC3: A ticket auto-closed for no response reopens to `NEW` and is routed again.
- AC4: After 7 days → `409 REOPEN_WINDOW_EXPIRED`; the UI shows *Create follow-up ticket* with a `FOLLOW_UP_OF` link.
- AC5: The 3rd reopen escalates the ticket to Admin.

**US-TK-24 — Cancel** · Must
- AC1: The requester can cancel their own ticket in `NEW`, `NEEDS_INFO` or `ASSIGNED`; in `IN_PROGRESS` the button becomes *Ask to cancel*; a direct API cancel from `IN_PROGRESS` → `409`.
- AC2: Staff (assignee, Admin) can cancel with a reason code and note (≥ 10 chars); `DUPLICATE` needs a valid original ticket code.
- AC3: Cancelling an `AUTOMATING` ticket → `409` with an explanatory message.
- AC4: `CANCELLED` is final: replies, reopen and any status change → `409`.
- AC5: The other party is notified with the reason; F07 counts `CANCELLED` tickets in volume only.

## 3. INVEST review

`PASS` means the design gives enough information for planning and acceptance; it does **not** mean code or tests exist.

| Story | I | N | V | E | S | T | Notes |
|---|---|---|---|---|---|---|---|
| US-TK-01 | NEEDS DEP | PASS | PASS | PASS | NEEDS SPLIT | PASS | Mailbox connector; can split: intake / threading / acknowledgement. |
| US-TK-02 | PASS | PASS | PASS | PASS | PASS | PASS | Routing outcome from F05; demoable with a stub router. |
| US-TK-03 | PASS | PASS | PASS | PASS | PASS | PASS | Scanner can be a stub (EICAR test file) in the V1 demo. |
| US-TK-04 | PASS | PASS | PASS | PASS | PASS | PASS | Interacts with BR-WF-16. |
| US-TK-05 | PASS | PASS | PASS | PASS | PASS | PASS | Label mapping BR-TK-18. |
| US-TK-06 | PASS | PASS | PASS | PASS | PASS | PASS | SLA state and assignment from F05; claim on F02. |
| US-TK-07 | PASS | PASS | PASS | PASS | NEEDS SPLIT | PASS | Can split: filters (DB) / full-text search. |
| US-TK-08 | PASS | PASS | PASS | PASS | PASS | PASS | Stores parameters only. |
| US-TK-09 | PASS | PASS | PASS | PASS | PASS | PASS | Visibility verified on API DTOs. |
| US-TK-10 | NEEDS DEP | PASS | PASS | PASS | PASS | PARTIAL | AI Service summary model; "no invented facts" checked by manual review of 50 summaries. |
| US-TK-11 | PASS | PASS | PASS | PASS | PASS | PASS | Reuses the generator of US-TK-10. |
| US-TK-12 | NEEDS DEP | NEEDS CONFIRM | PASS | PASS | PASS | PASS | Who approves is pending Q-TK-10 / Q-AI-16. |
| US-TK-13 | PASS | PASS | PASS | PASS | PASS | PASS | Member pick by F05 UC-WF-05. |
| US-TK-14 | PASS | NEEDS CONFIRM | PASS | PASS | PASS | PASS | Support tiers pending Q-TK-03. |
| US-TK-15 | PASS | PASS | PASS | PASS | PASS | PASS | Email answer threaded by BR-TK-23. |
| US-TK-16 | PASS | NEEDS CONFIRM | PASS | PASS | PASS | PASS | `waiting_on` pending Q-TK-02. |
| US-TK-17 | PASS | PASS | PASS | PASS | PASS | PASS | |
| US-TK-18 | PASS | PASS | PASS | PASS | PASS | PASS | Mention grants read-only access. |
| US-TK-19 | PASS | PASS | PASS | PASS | PASS | PASS | |
| US-TK-20 | NEEDS DEP | PASS | PASS | PASS | PASS | PASS | Drafts from F04 UC-AI-08; works without them (empty fields). |
| US-TK-21 | PASS | PASS | PASS | PASS | PASS | PASS | Shares `ticket_links` with F05. |
| US-TK-22 | PASS | NEEDS CONFIRM | PASS | PASS | PASS | PASS | Auto-confirm pending Q-TK-01 (BR-WF-11). |
| US-TK-23 | PASS | PASS | PASS | PASS | PASS | PASS | 7-day window shared with F05 / M-04. |
| US-TK-24 | PASS | NEEDS CONFIRM | PASS | PASS | PASS | PASS | Cancel from `IN_PROGRESS` pending Q-TK-04. |

## 4. Traceability

| Story | UC | Screen / dialog | API operationId (proposed) | Tables | Rules |
|---|---|---|---|---|---|
| US-TK-01 | UC-TK-01 | EML-T-01 | mailbox intake (internal), `listTicketMessages` | `tickets`, `ticket_emails`, `audit_logs` | BR-TK-02, 19, 23, 24 |
| US-TK-02 | UC-TK-01 | SCR-T-01 | `createTicket` | `tickets`, `audit_logs` | BR-TK-02, 19, 24 |
| US-TK-03 | UC-TK-01 | SCR-T-01, SCR-T-02, EML-T-01 | `uploadAttachment`, `getAttachmentDownloadUrl` | `ticket_attachments` | BR-TK-03 |
| US-TK-04 | UC-TK-01 | SCR-T-01 | `createTicket` (`beneficiaryId`), `searchUsers` (F09) | `tickets` | BR-TK-22, BR-WF-16 |
| US-TK-05 | UC-TK-02 | SCR-T-03 | `listTickets` (`scope=mine`) | `tickets` | BR-TK-01, 18 |
| US-TK-06 | UC-TK-02 | SCR-T-04 | `listTickets` (`view=…`), `getTicketViewCounts` | `tickets`, `ticket_sla_timers` | BR-TK-01 |
| US-TK-07 | UC-TK-02 | SCR-T-04 | `listTickets` (`q`, filters, sort) | `tickets`, search index | BR-TK-01, 10 |
| US-TK-08 | UC-TK-02 | SCR-T-04 | `listSavedViews`, `createSavedView`, `updateSavedView`, `deleteSavedView` | `saved_views` | — |
| US-TK-09 | UC-TK-02 | SCR-T-02 | `getTicket`, `listTicketMessages`, `getTicketEmail`, `listTicketHistory` (F08) | `tickets`, `ticket_messages`, `ticket_emails` | BR-TK-01, 10, 18 |
| US-TK-10 | UC-TK-03 | SEC-T-01, SCR-91 | `getTicketSummary` | `ticket_summaries` | BR-TK-10, 26 |
| US-TK-11 | UC-TK-03 | SEC-T-01 | `regenerateTicketSummary`, `listTicketSummaryVersions`, `rateTicketSummary` | `ticket_summaries` | BR-TK-26 |
| US-TK-12 | UC-TK-03 | SEC-T-01 | `getTicketSummary` (`purpose=APPROVAL`) | `ticket_summaries` | BR-TK-26 |
| US-TK-13 | UC-TK-04 | DLG-T-02 | `transferTicket`, `listTransferTargets` | `ticket_assignments` | BR-TK-05, 06, 15, 25 |
| US-TK-14 | UC-TK-04 | DLG-T-03 | `escalateTicket` | `ticket_escalations` | BR-TK-16, 25 |
| US-TK-15 | UC-TK-05 | SCR-T-02, EML-T-02 | `addTicketMessage`, mailbox intake, `answerClarification` (F05) | `ticket_messages`, `ticket_status_history` | BR-TK-06, 23 |
| US-TK-16 | UC-TK-05 | DLG-T-04 | `requestTicketInfo`, `resumeTicketWork` | `tickets`, `sla_events` | BR-TK-06 |
| US-TK-17 | UC-TK-05 | SCR-T-02 | `addTicketMessage` (`PUBLIC`) | `ticket_messages`, `notifications` | BR-TK-13 |
| US-TK-18 | UC-TK-05 | SCR-T-02 | `addTicketMessage` (`INTERNAL`), `redactTicketMessage` | `ticket_messages`, `ticket_access_grants` | BR-TK-10, 13 |
| US-TK-19 | UC-TK-06 | SCR-T-02 | `startTicketWork` | `tickets`, `ticket_status_history` | BR-TK-09, 21 |
| US-TK-20 | UC-TK-06 | DLG-T-05 (+ F04 SEC-AI-13), EML-T-03 | `resolveTicket` | `tickets`, `ticket_links` | BR-TK-07 |
| US-TK-21 | UC-TK-06 | SCR-T-02 related panel | `linkTickets`, `unlinkTickets` (F05) | `ticket_links` | BR-TK-05 |
| US-TK-22 | UC-TK-07 | DLG-T-08, EML-T-03 | `confirmTicketResolution` | `tickets`, `ticket_ratings` | BR-TK-12, 22 |
| US-TK-23 | UC-TK-07 | DLG-T-07 | `reopenTicket` | `tickets` | BR-TK-11 |
| US-TK-24 | UC-TK-07 | DLG-T-06 | `cancelTicket` | `tickets` | BR-TK-08, 17 |

## 5. Mapping from the previous version

The backlog was re-cut to the seven leaves of the final mind map. Old story IDs are not reused for a different meaning outside this table.

| Previous | Now | Change |
|---|---|---|
| US-TK-01, 02 (create, attach) | US-TK-01, 02, 03 | Split by channel (email, portal free text); the category hint and urgency fields were removed |
| US-TK-03 (on behalf) | US-TK-04 | — |
| US-TK-04 (similar tickets and KB while typing) | — | Moved: duplicate warning to F05 (UC-WF-10), KB suggestions to F04 (SEC-AI-01) |
| US-TK-05 … 09 | US-TK-05 … 09 | Triage view removed; source, support level, owner filters added |
| — | US-TK-10, 11, 12 | New leaf *View ticket summary on handoff and approval* (taken over from F04 US-AI-07, US-AI-08) |
| US-TK-10 (assign / reassign), US-TK-11 (claim) | — | Removed: F05 assigns automatically (UC-WF-05, Admin reassign as last resort); claim is on the F02 workbench |
| US-TK-12 (transfer to another team), US-TK-13 (escalate) | US-TK-13, 14 | Limited to the same division; another group → F05 *Return for re-routing* |
| US-TK-14, 15, 16 | US-TK-17, 18, 16 | Re-grouped under *Add missing information to on-hold tickets*; new US-TK-15 for the requester's answer (portal or email) |
| US-TK-17, 18 | US-TK-19, 20 | Resolution drafted by F04; AI resolution codes added |
| US-TK-19 (category / priority) | — | Moved to F05 (`correctTicketField`, `changeTicketPriority`) |
| — | US-TK-21 | Link related tickets (used by F05 UC-WF-10) |
| US-TK-20, 21, 22 | US-TK-22, 23, 24 | — |
