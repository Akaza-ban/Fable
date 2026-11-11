# Screen Content — F03 Ticket Management

> Design status: TARGET V1
>
> Related: [IA overview](README.md) · [Screens and navigation](01-screens-and-navigation.md) · [User flows](03-user-flows.md) · [UI states](04-ui-states-and-accessibility.md)

Zones of each page, the F03 sections, dialog fields, emails, which actions each role sees in each status, and low-fidelity wireframes. Sections of other branches are named where they sit (`SEC-AI-xx` from [F04](../../f04-ai-ticket-processing/information-architecture/01-screens-and-navigation.md), `SCR-WF-xx` from [F05](../../f05-workflow-automation/information-architecture/04-screen-inventory.md)).

## 1. Pages

### 1.1 SCR-T-01 New request

| Zone | Content |
|---|---|
| Header | "New request"; hint *"Describe what you need in your own words — no category to pick. You can also email it-support@…"* |
| Form | Description* (multi-line, counter 0/5,000, placeholder: *what you need or what happened, since when, the error message*); Subject (optional, counter 0/200, *"Leave empty to use the first line"*); ☐ *This request is for someone else* → user picker (name / email, active users). **No category, request type, urgency or priority field, and no category form** (BR-TK-24). |
| Evidence | Drop zone *"Drag files here, browse, or paste a screenshot (Ctrl+V)"*; limits text; per-file row: thumbnail / icon, name, size, progress, scan status (Scanning… / ✔ Clean / ⛔ Blocked), remove |
| KB suggestions | F04 SEC-AI-01 (up to 3 articles while typing; *This solved my problem*) |
| Footer | *Submit* (disabled while uploads / scans are pending or the description is invalid); *Cancel* (confirm discard if dirty) |
| States | Field errors inline; `429` banner; draft restored banner |

### 1.2 SCR-T-02 Ticket detail

| Zone | Requester / beneficiary | Staff (Technician, Admin) |
|---|---|---|
| Header | Code, title, **plain-language status** (BR-TK-18), created date, *"Requested by …"* when beneficiary; owner shown as *"AI Assistant"* or *"IT Helpdesk team"* | Code, title, **internal status**, source chip (`EMAIL` / `PORTAL`, created by), support level, priority badge, SLA badge + countdown (or *Paused*), group › sub-group, owner (*AI agent — {domain}* or technician), escalation level, reopen count |
| Action bar | Status-dependent (§4): *Confirm / Not fixed*, *Reopen*, *Cancel request*, *Ask to cancel*, *Create follow-up ticket* | Status- and owner-dependent: *Start work*, *Ask requester*, *Resume work*, *Resolve*, *Transfer*, *Escalate*, *Link*, *Cancel*; F05: *Return for re-routing* (SCR-WF-24) |
| Banner | SEC-T-02 on hold: the question and *"Waiting for your reply"*; *"Resolved — is it working now?"* with buttons; F05 merge banner (*Not the same problem*) | **SEC-T-01 summary** (first, expanded); F04 escalation banner (SEC-AI-05); SEC-T-02 *"On hold since …"*; *"Automation running — actions locked"* when `AUTOMATING` |
| Tab *Conversation* | Public messages, AI Assistant messages (SEC-AI-03), messages received by email (✉ icon); composer (public only) + attachments | All messages; internal notes highlighted (🔒 + label); composer with toggle **Public reply / Internal note**, `@mention`, attachments; *Redact* (Admin) |
| Details panel (right) | Submitted on, source, team name (no individual names unless they replied), Resolution reply when resolved; SEC-T-03 related tickets (scope-filtered) | Requester (name, department, contact), beneficiary, fields filled by F05 with the *"Filled by AI"* marker and the F05 *Correct* action (request type, category, subcategory, support level, business unit, site, priority + reason), assignee, timestamps (created, first response, started, on hold from–to, resolved, closed), SLA policy version; SEC-T-03; F04 SEC-AI-07 domain form |
| Tab *Attachments* | Attachments on public messages and on the original email; preview images; download | + attachments on internal notes; blocked files listed (name only, *Blocked*) |
| Tab *Email* | — | `EMAIL` tickets: original email (from, to, cc, date, subject, body, attachments list), read-only |
| Tab *History* (F08) | Allowlisted events only | Full timeline with actor, source (`USER` / `AI` / `AUTOMATION`), before/after, reason; routing history (F05) |
| Tab *Automation* (F04 SEC-AI-16) | — (one status line in the banner) | Runs, attempts, error codes; *Retry* / *Resolved manually* |
| Tab *AI* (F05 / F04) | — | AI decision, confidence, policy-check results, agent results (SEC-AI-18) |

### 1.3 SEC-T-01 Ticket summary

| Element | Content |
|---|---|
| Title line | *"Summary"* · version *v3* · *"Updated after a new message"* (when regenerated) · generated time · *Regenerate* · *Older versions* |
| Sections | **Problem** · **Impact** · **Key details** · **Already tried** (incl. what the AI agent did) · **Questions and answers** · **Why it came to you** (F05 route reason, escalation reason, transfer note) · **Open points**; empty → "None" |
| Approval variant | + **Requested action** (operation, system, entitlement, duration, beneficiary) · **Why approval is needed** (failed policy condition) |
| Facts | Each fact is a link to its message (scrolls and highlights it) |
| Feedback | 👍 helpful / 👎 not helpful (optional comment) |
| States | *"Summary is being prepared"* (≤ 30 s), *"Summary unavailable"* + original description, collapsed by the user (remembered per user) |

### 1.4 SCR-T-03 My tickets

| Zone | Content |
|---|---|
| Tabs | *Open* · *Needs my action* (badge) · *Resolved & closed* · *Cancelled* |
| Toolbar | Search (code / keyword), date filter, *+ New request* |
| Card / row | Code, title, source icon (✉ / 🌐), plain status chip, *Action needed* badge, last activity (*"Technician replied 2 h ago"*), *Requested by …* for beneficiary tickets |
| Empty state | *"You have no requests yet"* + *New request* + *"or email it-support@…"* |
| Mobile | Cards stacked, sticky *+ New request* button |

### 1.5 SCR-T-04 Work queues

| Zone | Content |
|---|---|
| Left nav | Views listed in [navigation](01-screens-and-navigation.md#4-navigation) with live counts; *Saved views* |
| Toolbar | Search box (code or keyword); filter chips: status, priority (P1–P3), source, support level, request type, category, group, sub-group, owner (AI agent / technician / in queue), assignee, requester, department, created / updated range, SLA state, flow A/B/C, waiting on, escalated, reopened, linked, has attachments; sort (SLA due ↑ default, priority, created, updated); *Save view*; column picker |
| Table | Code, title (+ ✉ / attachment icons), requester, priority, status, group › sub-group, owner, SLA badge + time left, updated; F04 SEC-AI-14 columns; row click → SCR-T-02 |
| States | Loading skeleton; empty view (*"Nothing here — nice work"*); search-down banner; `400` filter error inline |
| Paging | 25 / 50 / 100 per page; total count (approximate above 10,000) |

## 2. Dialog fields

| Dialog | Fields | Validation |
|---|---|---|
| DLG-T-02 Transfer within the group | Header *"Within {group}"*; target: sub-group (with its subcategories and tier) **or** technician of the group (status `ON` / `OFF`, workload score; unavailable disabled); reason code (`NEEDS_OTHER_SKILL` / `WORKLOAD` / `ABSENCE` / `OTHER`); note ≥ 10 chars; link *"Belongs to another group? Return for re-routing"* → SCR-WF-24 | BR-TK-15, BR-TK-25 |
| DLG-T-03 Escalate | Type: *Higher tier in {group}* (eligible sub-groups only) or *Admin*; reason; work done so far ≥ 20 chars | BR-TK-16 |
| DLG-T-04 Ask requester | Question ≥ 10 chars (sent publicly and by email); info: *"The ticket goes on hold and the SLA pauses until the requester replies"* | — |
| DLG-T-05 Resolve | Resolution code; **Resolution** (root cause, steps) and **reply to the requester** ≥ 20 chars, both pre-filled by F04 SEC-AI-13 (*"Drafted by AI"*) and editable; original ticket code (when `DUPLICATE`); ☐ *Resolve the related tickets too*; ☐ *Suggest a KB article* | BR-TK-07 |
| DLG-T-06 Cancel | Requester: optional note. Staff: reason code (`DUPLICATE` / `SPAM_OR_TEST` / `REQUESTER_WITHDREW` / `OUT_OF_SCOPE`) + note ≥ 10 chars (+ original code if `DUPLICATE`) | BR-TK-08 |
| DLG-T-07 Not fixed / reopen | Reason ≥ 10 chars, attachments; remaining reopen window | BR-TK-11 |
| DLG-T-08 Confirm & rate | 1–5 stars (optional), comment ≤ 1,000 chars | — |

## 3. Emails

Plain, short emails in the requester's language; the subject always starts with `[TK-YYYY-NNNNNN]`; the body starts with *"Reply above this line to add information"* so replies are threaded (BR-TK-23). Emails never contain internal notes, the summary or routing details (BR-TK-10).

| Email | Subject | Body |
|---|---|---|
| EML-T-01 Acknowledgement | `[TK-…] We received your request: {subject}` | What we received (first lines), *"Our assistant is reviewing it — you don't need to choose a category"*, files not accepted (if any), F04 KB suggestions (if any), *View ticket* |
| EML-T-02 Information needed | `[TK-…] We need a little more information` | The question, *"Just reply to this email"*, what happens if there is no answer (reminder dates, auto-close date), *View ticket* |
| EML-T-03 Resolved — please confirm | `[TK-…] Resolved: {subject}` | The reply (from the technician or *AI Assistant*), *Yes, it's solved* / *No, still a problem* links (open DLG-T-08 / DLG-T-07), auto-close date |
| EML-T-04 Ticket update | `[TK-…] {event}` | The new public reply, or the cancel / reopen reason, *View ticket* |

## 4. Action availability (status × role)

`R` = requester (owner), `A` = individual assignee (technician), `T` = sub-group member (not assignee), `AD` = Admin. Technician actions require the owner to be a technician (not the AI agent). Anything not listed → hidden in UI and rejected by API (D-TK-IA-04, BR-TK-04).

| Action | `NEW` | `NEEDS_INFO` | `ASSIGNED` | `IN_PROGRESS` | `AUTOMATING` | `RESOLVED` | `CLOSED` | `CANCELLED` |
|---|---|---|---|---|---|---|---|---|
| Public reply / add information | R, AD | R, A, AD | R, A, T, AD | R, A, T, AD | R, A, AD | R, A, AD | — (email: UC-TK-01 A2) | — |
| Internal note | AD | A, T, AD | A, T, AD | A, T, AD | A, T, AD | A, AD | — | — |
| Read summary (SEC-T-01) | — | A, T, AD | A, T, AD | A, T, AD | A, T, AD | A, T, AD | A, AD | AD |
| Transfer / escalate (same group) | — | — | A, AD | A, AD | — | — | — | — |
| Start work | — | — | A | — | — | — | — | — |
| Ask requester (put on hold) | — | — | — | A | — | — | — | — |
| Resume work | — | A (`waiting_on = TECHNICIAN`) | — | — | — | — | — | — |
| Resolve | — | — | — | A, AD | — (F04) | — | — | — |
| Link related | — | A, T, AD | A, T, AD | A, T, AD | A, AD | A, AD | — | — |
| Confirm & rate | — | — | — | — | — | R, AD* | rate only, R (window) | — |
| Not fixed / reopen | — | — | — | — | — | R, AD* | R, AD* (≤ 7 days) | — |
| Cancel | R, AD | R, A, AD | R, A, AD | A, AD (R: *Ask to cancel*) | — | — | — | — |
| Create follow-up | — | — | — | — | — | — | R (> 7 days) | R |

`*` on behalf of the requester, reason required.

## 5. Wireframes (low fidelity)

**SCR-T-01 New request**

```text
┌───────────────────────────────────────────────────────────────┐
│ New request                                                    │
│ Describe what you need — no category to pick.                  │
│ You can also email it-support@…                                │
├───────────────────────────────────────────────────────────────┤
│ What do you need? *                                            │
│ [ Since this morning I get "Access denied" when opening      ] │
│ [ \\fs01\accounting. I need it for the month-end close.      ] │
│ Subject (optional) [                                    ] 0/200│
│ ☐ This request is for someone else                             │
│ ┌ Evidence ───────────────────────────────────────────────┐   │
│ │  Drag files here, browse, or paste a screenshot (Ctrl+V) │   │
│ │  🖼 error.png   180 KB  ✔ Clean                      ✕   │   │
│ └──────────────────────────────────────────────────────────┘   │
│ Might help: KB-112 "Request access to a shared folder"  (F04)  │
│                                          [Cancel]  [ Submit ]  │
└───────────────────────────────────────────────────────────────┘
```

**SCR-T-04 Work queues (technician)**

```text
┌──────────────────┬──────────────────────────────────────────────────────────────┐
│ QUEUES           │ [🔍 code or keyword ] [Status▾][Priority▾][Source▾][SLA▾] [Save view]│
│ Assigned to me 12│ Code            Title             Src Prio Status      SLA        │
│ Sub-group queue 5│ TK-2026-004890  PC won't boot      ✉  P2  ASSIGNED    ⚠ 35m      │
│ On hold         3│ TK-2026-004871  VPN drops hourly   🌐 P3  IN_PROGRESS ✔ 5h       │
│ Due soon        2│ TK-2026-004702  New mouse          ✉  P3  NEEDS_INFO  ⏸ paused   │
│ Breached        1│ TK-2026-004655  Printer 3F jam     ✉  P2  IN_PROGRESS ⛔ -20m     │
│ SAVED VIEWS      │                                         ‹ 1 2 3 ›  25 / page      │
│ My P1–P2         │                                                                │
└──────────────────┴──────────────────────────────────────────────────────────────┘
```

**SCR-T-02 Ticket detail (staff)**

```text
┌──────────────────────────────────────────────────────────────────────────────────┐
│ TK-2026-004871  VPN drops every hour  IN_PROGRESS  P3  SLA ✔ 5h  ✉ EMAIL  L1       │
│ IT Helpdesk › Network · Owner: Minh T.                                             │
│ [Ask requester] [Resolve] [Transfer] [Escalate] [Return for re-routing] [⋯]        │
├────────────────────────────────────────────────────────┬─────────────────────────┤
│ ▾ Summary v2 · updated after a new message  [Regenerate]│ Requester  Lan N. (Sales)│
│   Problem: VPN client disconnects about every 60 min¹   │ Category   Network/VPN 🤖│
│   Impact: one user, cannot work remotely²               │ Priority   P3 🤖         │
│   Already tried: AI guided reinstall — not fixed³       │ Support    L1 🤖         │
│   Why it came to you: agent could not resolve           │ Created    09:10         │
│   Open points: client version unknown         👍 👎     │ 1st resp.  09:45         │
│ [Conversation] Attachments Email History AI             │ Related    —             │
│ ✉ Lan N. · 09:12   VPN keeps dropping…  📎 log.txt      │ 🤖 = Filled by AI        │
│ 🔒 Minh T. · 09:40  Checked gateway logs…               │                         │
│ ┌──────────────────────────────────────────────────┐    │                         │
│ │ (•) Public reply  ( ) Internal note  📎           │    │                         │
│ │ [                                         ] [Send]│    │                         │
│ └──────────────────────────────────────────────────┘    │                         │
└────────────────────────────────────────────────────────┴─────────────────────────┘
```

**SCR-T-02 Ticket detail (requester, on hold)**

```text
┌───────────────────────────────────────────────────────────┐
│ TK-2026-004871  VPN drops every hour   Waiting for your reply│
├───────────────────────────────────────────────────────────┤
│ ⏸ Your technician needs more information:                  │
│ "Which VPN client version do you have? (Help → About)"     │
│ [ It's 4.8                                     ] 📎 [Send] │
│ You can also just reply to our email.                       │
├───────────────────────────────────────────────────────────┤
│ Conversation · Attachments · History                        │
│ ✉ You · 09:12      VPN keeps dropping…  📎 log.txt          │
│ Technician · 09:45 Which VPN client version do you have?   │
└───────────────────────────────────────────────────────────┘
```
