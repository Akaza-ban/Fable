# Screen Content — F03 Ticket Management

> Design status: TARGET V1
>
> Related: [IA overview](README.md) · [Screens and navigation](01-screens-and-navigation.md) · [User flows](03-user-flows.md) · [UI states](04-ui-states-and-accessibility.md)

Zones of each page, dialog fields, which actions each role sees in each status, and low-fidelity wireframes. Sections of other branches are named where they sit (`SEC-AI-xx` from [F04](../../f04-ai-ticket-processing/information-architecture/01-screens-and-navigation.md)).

## 1. Pages

### 1.1 SCR-T-01 Create ticket

| Zone | Content |
|---|---|
| Header | "New request"; short hint *"Describe what you need — our assistant will handle simple requests automatically."* |
| Form | Title* (counter 0/200); Description* (multi-line, counter 0/5,000, placeholder with examples: *what happened, since when, error message*); Category hint (dropdown: Account & access, Hardware, Software, Network, Email, Other, **I'm not sure** = default); Urgency (Low / Normal / High); Affected device or system (text); ☐ *This request is for someone else* → user picker (name / e-mail, active users) |
| Evidence | Drop zone *"Drag files here, browse, or paste a screenshot (Ctrl+V)"*; allowed types and limits text; per-file row: thumbnail / icon, name, size, progress, scan status (Scanning… / ✔ Clean / ⛔ Blocked), remove button |
| Suggestions panel (Could) | *"Is it one of these?"* up to 5 similar **own** open tickets (code, title, status, *Add my info here*) and 3 KB articles (title, *Open*) — F04 SEC-AI-01, SEC-AI-02 |
| Footer | *Submit* (disabled while uploads/scans are pending or required fields are invalid); *Cancel* (confirm discard if dirty) |
| States | Field errors inline; `429` banner *"You've created many tickets recently, please wait a moment"*; draft restored banner |

### 1.2 SCR-T-02 Ticket detail

| Zone | Requester / beneficiary | Staff (Technician, Admin) |
|---|---|---|
| Header | Code, title, **plain-language status** (BR-TK-18), created date, *"Requested by …"* when beneficiary | Code, title, **internal status**, priority badge, SLA badge + countdown (or *Paused*), flow A/B/C chip, escalation level, reopen count |
| Action bar | Status-dependent (see §3): *Confirm / Not fixed*, *Reopen*, *Cancel request*, *Ask to cancel*, *Create follow-up ticket* | Status-dependent: *Claim*, *Start work*, *Ask requester*, *Resume work*, *Resolve*, *Assign*, *Transfer*, *Escalate*, *Change category / priority*, *Link*, *Cancel* |
| Banner | Contextual call to action: *"Your technician needs more information"*, *"Resolved — is it working now?"* with buttons | AI summary card (collapsible, SEC-AI-06) + routing reason (*"Hardware request"*, *"Entitlement X not in catalog"*), escalation banner (SEC-AI-05); *"Waiting for requester since …"*; *"Automation running — actions locked"* when `AUTOMATING` |
| Tab *Conversation* | Public messages and AI Assistant messages in time order (SEC-AI-03); composer (public only) + attachments | All messages; internal notes highlighted (yellow, 🔒); composer with toggle **Public reply / Internal note**, `@mention`, attachments; *Redact* (Admin) |
| Details panel (right) | Category (display), submitted on, assigned team name (no individual names unless they replied), resolution note when resolved | Requester (name, department, contact), beneficiary, category (+ source AI / human), request type, priority (+ source), team, assignee, support tier, urgency, affected system, created / first response / started / resolved / closed timestamps, SLA policy version, links (duplicate of, related, follow-up of); F04 SEC-AI-07, SEC-AI-08 |
| Tab *Attachments* | All attachments on public messages; preview images; download | + attachments on internal notes; blocked files listed (name only, *Blocked*) |
| Tab *History* (F08) | Allowlisted events only (BR-NK-04) | Full timeline with actor, source (`USER` / `AI` / `AUTOMATION`), before/after, reason |
| Tab *SLA* (F05) | — | Response / resolution deadlines, pauses (from–to), warnings, breach flag |
| Tab *Automation* (F05) | — (requester sees outcome summary in banner) | Workflow runs, attempts, error codes; *Retry* / *Resolved manually* (F05 UC-WF-09) |
| Tab *AI* (F04 / F05) | — | AI decision: category, confidence, extracted fields, policy-check results, clarification rounds |

### 1.3 SCR-T-03 My tickets

| Zone | Content |
|---|---|
| Tabs | *Open* · *Needs my action* (badge) · *Resolved & closed* · *Cancelled* |
| Toolbar | Search (code / keyword), date filter, *+ New ticket* |
| Card / row | Code, title, plain status chip, *Action needed* badge, last activity (*"Technician replied 2 h ago"*), *Requested by …* for beneficiary tickets |
| Empty state | Illustration + *"You have no requests yet"* + *Create ticket* |
| Mobile | Cards stacked, one column, sticky *+ New ticket* button |

### 1.4 SCR-T-04 Ticket list / work queues

| Zone | Content |
|---|---|
| Left nav | Views listed in [navigation](01-screens-and-navigation.md#4-navigation) with live counts; *Saved views* (rename / delete) |
| Toolbar | Search box (code or keyword); filter chips: status, priority, category / request type, team, assignee (incl. *Unassigned*), requester, department, created / updated range, SLA state, flow A/B/C, waiting on, escalated, reopened, has attachments; sort (SLA due ↑ default, priority, created, updated); *Save view*; column picker |
| Table | ☐ select, code, title (+ attachment icon), requester, priority, status, team, assignee, SLA badge + time left, updated; F04 AI columns (SEC-AI-14); row click → SCR-T-02 |
| Bulk actions | Claim, Assign (Admin) — max 50; result summary per ticket |
| States | Loading skeleton; empty view (*"Nothing here — nice work"*); search-down banner; `400` filter error inline |
| Paging | 25 / 50 / 100 per page; total count (approximate above 10,000) |

## 2. Dialog fields

| Dialog | Fields | Validation |
|---|---|---|
| DLG-T-01 Assign / reassign | Team (any), technician list with open-ticket count and absence (unavailable disabled), reason (required for reassign) | BR-TK-15, BR-TK-05 |
| DLG-T-02 Transfer | Target team (with categories and tier), reason code (`WRONG_TEAM` / `NEEDS_OTHER_SKILL` / `OTHER`), note ≥ 10 chars | ≠ current team |
| DLG-T-03 Escalate | Type: *Higher support tier* (eligible teams only) or *Admin*; reason; work done so far ≥ 20 chars | BR-TK-16 |
| DLG-T-04 Ask requester | Question ≥ 10 chars (sent publicly); info: *"SLA will pause until the requester replies"* | — |
| DLG-T-05 Resolve | Resolution code; requester-visible note ≥ 20 chars (F04 SEC-AI-13 draft); internal root cause (optional); original ticket code (when `DUPLICATE`); ☐ *Suggest a KB article* | BR-TK-07 |
| DLG-T-06 Cancel | Requester: optional note. Staff: reason code (`DUPLICATE` / `SPAM_OR_TEST` / `REQUESTER_WITHDREW` / `OUT_OF_SCOPE`) + note ≥ 10 chars (+ original code if `DUPLICATE`) | BR-TK-08 |
| DLG-T-07 Not fixed / reopen | Reason ≥ 10 chars, attachments; info about remaining reopen window | BR-TK-11 |
| DLG-T-08 Confirm & rate | 1–5 stars (optional), comment ≤ 1,000 chars | — |
| DLG-T-09 Category / priority | New category / request type, new priority (technician: *propose*), reason | BR-TK-05, BR-WF-08 |

## 3. Action availability (status × role)

`R` = requester (owner), `A` = individual assignee, `T` = team member (not assignee), `AD` = Admin. Anything not listed → hidden in UI and rejected by API (D-TK-IA-04, BR-TK-04).

| Action | `NEW` | `NEEDS_INFO` | `TRIAGE` | `ASSIGNED` | `IN_PROGRESS` | `AUTOMATING` | `RESOLVED` | `CLOSED` | `CANCELLED` |
|---|---|---|---|---|---|---|---|---|---|
| Public reply | R, AD | R, A, AD | R, AD | R, A, T, AD | R, A, T, AD | R, A, AD | R, A, AD | — | — |
| Internal note | AD | A, T, AD | AD | A, T, AD | A, T, AD | A, T, AD | A, AD | — | — |
| Claim | — | — | — | T (no assignee) | — | — | — | — | — |
| Assign / reassign | — | — | AD | AD | AD | — | — | — | — |
| Transfer / escalate | — | — | — | A, AD | A, AD | — | — | — | — |
| Start work | — | — | — | A | — | — | — | — | — |
| Ask requester | — | — | — | — | A | — | — | — | — |
| Resume work | — | A (`waiting_on = TECHNICIAN`) | — | — | — | — | — | — | — |
| Change category / priority | — | — | AD | A, AD | A, AD | — | — | — | — |
| Resolve | — | — | — | — | A, AD | — (F05) | — | — | — |
| Confirm & rate | — | — | — | — | — | — | R, AD* | rate only, R (window) | — |
| Not fixed / reopen | — | — | — | — | — | — | R, AD* | R, AD* (≤ 7 days) | — |
| Cancel | R, AD | R, A, AD | R, AD | R, A, AD | A, AD (R: *Ask to cancel*) | — | — | — | — |
| Create follow-up | — | — | — | — | — | — | — | R (> 7 days) | — |

`*` on behalf of the requester, reason required.

## 4. Wireframes (low fidelity)

**SCR-T-01 Create ticket**

```text
┌───────────────────────────────────────────────────────────────┐
│ New request                                                    │
│ Describe what you need — simple requests are handled instantly │
├───────────────────────────────────────────────────────────────┤
│ Title *  [ Cannot access shared Accounting folder        ] 42/200│
│ Description *                                                  │
│ [ Since this morning I get "Access denied" when opening      ] │
│ [ \\fs01\accounting ...                                       ] │
│ Category [ I'm not sure ▾ ]   Urgency ( ) Low (•) Normal ( ) High│
│ Affected system [ File server ]                                │
│ ☐ This request is for someone else                             │
│ ┌ Evidence ───────────────────────────────────────────────┐   │
│ │  Drag files here, browse, or paste a screenshot (Ctrl+V) │   │
│ │  🖼 error.png   180 KB  ✔ Clean                      ✕   │   │
│ └──────────────────────────────────────────────────────────┘   │
│ Is it one of these?  TK-2026-004211 Folder access (Open) [Add] │
│                                          [Cancel]  [ Submit ]  │
└───────────────────────────────────────────────────────────────┘
```

**SCR-T-04 Work queues (technician)**

```text
┌──────────────────┬────────────────────────────────────────────────────────────┐
│ QUEUES           │ [🔍 code or keyword ] [Status▾][Priority▾][Team▾][SLA▾] [Save view]│
│ Assigned to me 12│ ☐ Code            Title               Prio Status      SLA       │
│ Team unassigned 5│ ☐ TK-2026-004890  PC won't boot        P2  ASSIGNED    ⚠ 35m     │
│ Waiting reqster 3│ ☐ TK-2026-004871  VPN drops hourly     P3  IN_PROGRESS ✔ 5h      │
│ Due soon        2│ ☐ TK-2026-004702  New mouse            P4  NEEDS_INFO  ⏸ paused  │
│ Breached        1│ ☐ TK-2026-004655  Printer 3F jam       P3  IN_PROGRESS ⛔ -20m    │
│ SAVED VIEWS      │                                         ‹ 1 2 3 ›  25 / page    │
│ My P1–P2         │ [Claim] [Assign]  (bulk)                                       │
└──────────────────┴────────────────────────────────────────────────────────────┘
```

**SCR-T-02 Ticket detail (staff)**

```text
┌─────────────────────────────────────────────────────────────────────────────┐
│ TK-2026-004871  VPN drops every hour   IN_PROGRESS  P3  SLA ✔ 5h left  Flow C │
│ [Ask requester] [Resolve] [Transfer] [Escalate] [Category/Priority] [⋯]      │
├───────────────────────────────────────────────────┬─────────────────────────┤
│ ▸ AI summary: VPN client disconnects ~60 min;      │ Requester  Lan N. (Sales)│
│   routed to human: "Network incident"              │ Team       Network L1    │
│ [Conversation] Attachments History SLA AI          │ Assignee   Minh T.       │
│ ─────────────────────────────────────────────────  │ Category   Network (AI)  │
│ Lan N. · 09:12   VPN keeps dropping…  📎 log.txt   │ Priority   P3 (AI)       │
│ 🔒 Minh T. · 09:40  Checked gateway logs…          │ Created    09:10         │
│ Minh T. · 09:45  Could you send your client ver.? │ 1st resp.  09:45         │
│ ┌───────────────────────────────────────────────┐  │ Links      —             │
│ │ (•) Public reply  ( ) Internal note  📎        │  │                         │
│ │ [                                         ]    │  │                         │
│ │                                     [Send]     │  │                         │
│ └───────────────────────────────────────────────┘  │                         │
└───────────────────────────────────────────────────┴─────────────────────────┘
```

**SCR-T-02 Ticket detail (requester, `RESOLVED`)**

```text
┌───────────────────────────────────────────────────────────┐
│ TK-2026-004871  VPN drops every hour   Resolved — please confirm │
├───────────────────────────────────────────────────────────┤
│ ✅ Your technician resolved this request:                   │
│ "Updated your VPN client to 5.2; the timeout is fixed."    │
│ Is it working now?  [ Yes, it's solved ] [ No, still a problem ]│
├───────────────────────────────────────────────────────────┤
│ Conversation · Attachments · History                        │
│ You · 09:12        VPN keeps dropping…  📎 log.txt          │
│ Technician · 09:45 Could you send your client version?     │
│ You · 10:02        It's 4.8                                 │
│ Technician · 11:30 Updated your VPN client to 5.2 …        │
└───────────────────────────────────────────────────────────┘
```
