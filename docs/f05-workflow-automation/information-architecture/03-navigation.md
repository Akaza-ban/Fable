# Navigation

> Design status: V1 — draft, awaiting user review
>
> Implementation status: NOT STARTED

## Global navigation (D-IA-02)

Every role uses the same layout: a **left sidebar** with the pages of the role, and a **top bar**.

**Top bar, for every role:**

| Element | Content |
|---|---|
| Quick create | **+ Create ticket**, opens SCR-T-01. Shown for every role. |
| Notifications | The word "Notifications" followed by the unread count, e.g. "Notifications 3"; always text with a number, never only a coloured dot. Opens SCR-91 (D-IA-09). |
| User | Name and role, e.g. "Lan — Requester", "Hung — Technician (Hardware)". |
| Availability (Technician only) | Switch *Available / Busy / Off* next to the name, always as text with the status word (D-IA-11). Changing it takes effect on the next assignment; switching to *Off* asks for confirmation: *"Your not-started tickets will be moved to available colleagues."* *(proposed)* |
| Log out | Ends the session and goes to SCR-01. |

**Sidebar by role.** Only permitted items are rendered. The first item is the home page (DF-IA-02).

| Role | Sidebar items (in order) | Badges |
|---|---|---|
| **Requester** | **My tickets** (SCR-T-03, home) · Notifications (SCR-91) | My tickets: tickets where the AI is waiting for my answer + tickets waiting for my confirmation |
| **Technician** | **Assigned to me** (SCR-T-03, home) · Notifications (SCR-91) | Assigned to me: open tickets; tickets near or past their SLA shown separately |
| **Admin** | **AI decisions** (SCR-WF-30, home) · All tickets (SCR-T-03) · **Workflow configuration** group: Assignment rules (SCR-WF-01), SLA (SCR-WF-02), AI scope (SCR-WF-03) · Notifications (SCR-91) | All tickets: tickets waiting in a team queue (no eligible technician) + SLA breaches |

Notes:

- A badge is a number computed by the server, always shown as digits.
- A role never sees a disabled item for a page it cannot open; the item is simply not rendered.
- When the Dashboard (F02) is specified, the Admin home page will move to the Dashboard and "AI decisions" becomes the second item.

## Contextual navigation

- **List → detail:** opening a ticket from SCR-T-03 → SCR-T-02. Filters, sort order and scroll position are kept in the URL; "Back to list" restores them.
- **Breadcrumbs on detail pages:**
  - `My tickets / #T-102938` when opened from SCR-T-03.
  - `AI decisions / D-55120 / #T-102938` when opened from SCR-WF-31.
  - `Notifications / #T-102938` when opened from SCR-91.
  - SCR-WF-31 shows `AI decisions / D-55120`.
- **After creating a ticket (SCR-T-01):** go straight to SCR-T-02 of the new ticket, showing *"The AI is analyzing your request…"* (DF-IA-07).
- **After answering an AI question:** stay on SCR-T-02; the conversation shows the answer and *"The AI is re-analyzing…"*.
- **After confirming resolved:** stay on SCR-T-02; the ticket becomes *Closed*.
- **After reporting not fixed (SCR-WF-22):** stay on SCR-T-02; the ticket shows *"Handed over to a technician"*.
- **After reassigning (SCR-WF-11, Admin):** stay on SCR-T-02 with the new assignee shown (DF-IA-05).
- **After retrying (SCR-WF-20) or resolving manually (SCR-WF-21):** stay on SCR-T-02, Automation tab, with the new status.
- **After saving configuration** (SCR-WF-04, 06, 07, or saving the SLA table): stay on the configuration page; the new/edited row is highlighted.
- **After login:** go to the role home page. **After log out / session end:** go to SCR-01.
- **Conflict** (data changed by someone else meanwhile): stay on the page, keep the entered data, and show a **Reload** button.

## Notifications

The notification centre SCR-91 has an unread count in the top bar (D-IA-09). Notifications are in-app only in V1. Each item has a recipient, a link to its object and a read mark.

| Recipient | Event | Opens |
|---|---|---|
| Requester | AI clarification: more information needed (UC-WF-04) | SCR-T-02 |
| Requester | Reminder to answer the AI question, day 1 and day N−1 (UC-WF-06) | SCR-T-02 |
| Requester | Ticket auto-closed for no response, with the 7-day reopen deadline | SCR-T-02 |
| Requester | Request approved by the AI and being processed (UC-WF-07) | SCR-T-02 |
| Requester | Done, please confirm; or the account check result (UC-WF-08, 09) | SCR-T-02 |
| Requester | Request handed over to a technician, with a summary reason (DF-IA-08) | SCR-T-02 |
| Requester | Time-bound access will be revoked in 1 day (US-WF-18) | SCR-T-02 |
| Technician | New ticket assigned, with the AI summary and routing reason (UC-WF-05) | SCR-T-02 |
| Technician | Ticket reached its SLA warning threshold (UC-WF-06) | SCR-T-02 |
| Technician | Automation failed, ticket assigned to you (UC-WF-09) | SCR-T-02 Automation tab |
| Technician | Requester reported not fixed | SCR-T-02 |
| Technician | One of your not-started tickets was moved to a colleague (you went *Off*, or SLA breach) | SCR-T-02 |
| Admin | Ticket breached its SLA, with the automatic reassignment if any (UC-WF-06) | SCR-T-02 |
| Admin | Ticket waiting in a team queue for more than 30 minutes because no technician is eligible *(proposed)* | SCR-T-02 |
| Admin | AI Service unavailable, new tickets are being routed to humans (DF-IA-06) | SCR-WF-30 |

Other rules:

- Opening a notification opens its object and marks it read; the unread count drops accordingly.
- A notification that cannot be saved **never** blocks the action that caused it; the failure is logged.
- At ~8,000 tickets/day, Admin notifications only cover system events (AI failure), never individual tickets.

**Related:** [Information architecture index](README.md) · [Routes and permissions](02-routes-and-permissions.md) · [User flows](05-user-flows.md)
