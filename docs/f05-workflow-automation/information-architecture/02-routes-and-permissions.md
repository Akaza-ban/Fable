# Routes and Page Permissions

> Design status: V1 — draft, awaiting user review
>
> Implementation status: NOT STARTED

## Routes

Routes are logical paths (DF-IA-01). Modals have no route of their own: they open on top of their parent page and close back to it (D-IA-03).

| Route | Screen | Notes |
|---|---|---|
| `/login` | SCR-01 Login | Public. Every protected route redirects here when there is no session. |
| `/tickets` | SCR-T-03 Ticket list | Filters live in the query string (`?view=mine`, `?view=assigned`, `?flow=C`, `?status=NEEDS_INFO`) so Back restores them. |
| `/tickets/new` | SCR-T-01 Create ticket | `?template=access` / `account` / `hardware` pre-selects a quick template. |
| `/tickets/:id` | SCR-T-02 Ticket detail | `?tab=automation` opens the Automation tab. Opened from SCR-T-03, SCR-WF-31 or SCR-91. |
| `/notifications` | SCR-91 Notifications | The user's own notifications. |
| `/workflow/assignment-rules` | SCR-WF-01 Assignment rules | `?tab=rules` / `workload`. |
| `/workflow/sla` | SCR-WF-02 SLA & business calendar | `?tab=policies` / `calendar`. |
| `/workflow/ai-scope` | SCR-WF-03 AI scope | `?tab=ai-actions` / `automation-actions` / `entitlements`. |
| `/workflow/ai-decisions` | SCR-WF-30 AI decisions | Filters in the query string (`?outcome=ROUTE_TO_HUMAN&date=...`). |
| `/workflow/ai-decisions/:id` | SCR-WF-31 AI decision detail | |
| `/403` | SCR-90 Access denied | Also shown in place of any page the role may not open. |

- Several roles share a route when they share a screen (`/tickets`, `/tickets/:id`, `/notifications`). The server returns only the records and actions the role may use.
- **Data scope is checked by the server on every request**, not by the route. Changing `:id` in the URL to another person's ticket → SCR-90, without revealing whether the ticket exists.
- A signed-in user who opens `/login` goes to the role home page.

## Page permissions

Rows are pages, columns are roles. A dash `—` means the role cannot open the page: the sidebar does not show it, and a direct URL gives SCR-90.

| Screen | Requester | Technician | Admin |
|---|---|---|---|
| SCR-01 Login | Public | Public | Public |
| SCR-T-01 Create ticket | ✓ | ✓ | ✓ |
| SCR-T-02 Ticket detail | Own tickets | Assigned tickets | All |
| SCR-T-03 Ticket list | Own tickets | Assigned tickets | All |
| SCR-91 Notifications | Own | Own | Own |
| SCR-WF-01 Assignment rules (Rules, Workload) | — | — | ✓ |
| SCR-WF-02 SLA & business calendar | — | — | ✓ |
| SCR-WF-03 AI scope | — | — | ✓ |
| SCR-WF-30 AI decisions | — | — | ✓ |
| SCR-WF-31 AI decision detail | — | — | ✓ |
| SCR-90 Access denied | ✓ | ✓ | ✓ |

Notes:

- **There is no Team Lead** (D-IA-07). No technician sees another technician's tickets or reassigns them; distribution is automatic (UC-WF-05).
- **Availability switch** (D-IA-11): every Technician sees *Available / Busy / Off* in the top bar; Requesters and Admins do not.
- **SCR-T-02 shows role-dependent panels** (D-IA-04):
  - Requester: AI conversation, progress bar, *Confirm resolved* / *Not fixed* buttons, summary reason (DF-IA-08). No Automation tab.
  - Technician: AI summary, routing reason, SLA tab, Automation tab with *Retry* / *Resolved manually* when the ticket is assigned to them.
  - Admin: everything, plus the *Reassign* button (SCR-WF-11) and a *View AI decision* link to SCR-WF-31.
- **Modals** follow the permission of the page that opens them, plus the specific conditions listed in the [screen inventory](04-screen-inventory.md).

## Behaviour when access is missing

| Situation | Result |
|---|---|
| Not signed in / session expired | Go to SCR-01 with a short message; after signing in, return to the page that was open. |
| Opening the URL of a forbidden page | SCR-90 with a link to the role home page. |
| Opening a ticket outside the user's scope | SCR-90; never says whether the ticket exists. |
| Clicking an action the server refuses (`403`) | Stay on the page and show *"You do not have permission to perform this action"*. |
| Account deactivated while in use | The next request ends the session and goes to SCR-01. |

**Related:** [Information architecture index](README.md) · [Sitemap](01-sitemap.md) · [Navigation](03-navigation.md)
