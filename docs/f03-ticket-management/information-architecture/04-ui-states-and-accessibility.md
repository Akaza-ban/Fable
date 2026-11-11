# UI States and Accessibility — F03 Ticket Management

> Design status: TARGET V1
>
> Related: [IA overview](README.md) · [Screens and navigation](01-screens-and-navigation.md) · [Screen content](02-screen-content.md) · [User flows](03-user-flows.md)

Copy in quotes comes from the use cases; copy marked *(proposed)* may change during UI/UX design.

## 1. UI state contract

| State | When | What the user sees | Screens |
|---|---|---|---|
| Loading | List or ticket loading | Page shell and sidebar stay; skeleton rows / panels | All |
| Empty | Nothing exists yet | *"You have no requests yet"* + *Create ticket* (SCR-T-03); *"Nothing here — nice work"* (SCR-T-04 view) | SCR-T-03, SCR-T-04 |
| No results | Filters match nothing | Active filters and *Clear filters* *(proposed)* | SCR-T-03, SCR-T-04 |
| Validation error | `400` | Keep entered values, error next to the field, focus the first error; uploaded attachments stay on the form | SCR-T-01, dialogs |
| File blocked | Malware scan result `INFECTED` | File row *⛔ Blocked*, *"This file was blocked for security reasons"*; ticket can be submitted without it | SCR-T-01, composer |
| Uploading / scanning | Attachment pending | Progress, *Scanning…*; *Submit* / *Send* disabled | SCR-T-01, composer |
| Rate limited | `429` (BR-TK-19) | Banner *"You've created many tickets recently, please wait a moment"* | SCR-T-01 |
| Draft restored | Unsent form found in the browser (UC-TK-01 A3) | Banner with *Discard draft* | SCR-T-01 |
| Saving | Mutation in flight | Submit button disabled with progress (no double submission) | All dialogs |
| Conflict | `409 STALE_VERSION` (BR-TK-21) | Nothing saved; keep input; *"This ticket was just changed by someone else. Reload to see the change."* *(proposed)* + *Reload* | SCR-T-02, dialogs, bulk actions |
| Invalid transition | `409 INVALID_TRANSITION` | Reload the ticket; show its current status and allowed actions | SCR-T-02 |
| Automation running | Status `AUTOMATING` | Banner *"Automation running — actions locked"*; only reading and messages allowed (BR-TK-08) | SCR-T-02 |
| Closed / cancelled | Final status (BR-TK-17) | Composer replaced by *"This ticket is closed. Reopen it to continue."* (requester, within window) or nothing (`CANCELLED`) | SCR-T-02 |
| Search unavailable | Search backend down (UC-TK-02 E4) | Banner; keyword search disabled; plain filters still work | SCR-T-04, SCR-T-03 |
| Forbidden | Out-of-scope ticket or page | `/403`, never reveals existence | All |

## 2. Status labels

Requesters see plain-language labels; staff see internal codes (BR-TK-18, D-TK-IA-08). Chips always use text and an icon, never color alone.

| Internal status | Staff chip | Requester chip | Requester call to action |
|---|---|---|---|
| `NEW` | NEW | Submitted | — |
| `NEEDS_INFO` | NEEDS_INFO (+ waiting on AI / technician) | **Waiting for your reply** | Reply box focused, counts in *Needs my action* |
| `TRIAGE` | TRIAGE | Waiting for a technician | — |
| `ASSIGNED` | ASSIGNED | Waiting for a technician | — |
| `IN_PROGRESS` | IN_PROGRESS | In progress | — |
| `AUTOMATING` | AUTOMATING | Being processed automatically | — |
| `RESOLVED` | RESOLVED | **Resolved — please confirm** | *Yes, it's solved* / *No, still a problem* |
| `CLOSED` | CLOSED (+ close reason) | Closed | *Reopen* (≤ 7 days) or *Create follow-up ticket* |
| `CANCELLED` | CANCELLED (+ cancel reason) | Cancelled | — |

## 3. SLA badges (staff only)

| SLA state (F05) | Badge | Example |
|---|---|---|
| `ON_TRACK` | ✔ + time left | ✔ 5h |
| `AT_RISK` | ⚠ + time left | ⚠ 35m |
| `BREACHED` | ⛔ + time overdue | ⛔ −20m |
| `PAUSED` | ⏸ Paused | ⏸ paused (in `NEEDS_INFO`) |

Durations use `2 h 15 min` and dates `dd/MM/yyyy HH:mm` in Vietnam time (F05 DF-IA-03).

## 4. Message markers

| Marker | Meaning | Who sees it |
|---|---|---|
| 🔒 **Internal note** (yellow background + text label) | `visibility = INTERNAL` | Staff only (BR-TK-10) |
| **AI Assistant** | Message written by the AI Service (F04) | Everyone |
| **System** | Status event in the conversation (assigned, resolved, …) | Everyone (requester: allowlisted events) |
| *[Removed by administrator: reason]* | Redacted message (BR-TK-13) | Everyone |
| **On behalf of {requester}** | Admin confirmed or reopened for the requester | Staff |

## 5. Responsive layout

| Width | SCR-T-02 | Lists |
|---|---|---|
| ≥ 1200 px | Conversation in the centre, details panel on the right, action bar under the header | SCR-T-04 full table with left view nav |
| 768–1199 px | Details panel collapses below the header as a summary; tabs scroll horizontally | View nav becomes a dropdown; fewer columns (column picker) |
| < 768 px | One column: header, banner with call to action, conversation, sticky composer; secondary actions in a `⋯` menu | SCR-T-03 cards, sticky *+ New ticket*; SCR-T-04 cards with code, title, status, SLA |

Create ticket and ticket detail must work at **360 px** (NFR-TK-UX-001): requesters often report from phones, including pasting or photographing an error.

## 6. Accessibility (WCAG 2.1 AA)

- Every form field has a visible label; character counters and errors are linked with `aria-describedby`; errors are announced.
- Drop zone is also a keyboard-reachable *Browse* button; pasting works from the description field; upload progress and scan results are `aria-live="polite"`.
- New messages and status changes on SCR-T-02 are announced in a polite live region; the SLA countdown is **not** announced every second.
- Dialogs trap focus, close with Esc and return focus to the button that opened them.
- Status chips, SLA badges and the internal-note marker use text and icons, not color alone; contrast ≥ 4.5:1.
- The work-queue table uses real table markup with sortable column headers (`aria-sort`); bulk-select checkboxes have labels with the ticket code.
- Vietnamese and English content carries the right `lang` attribute.
