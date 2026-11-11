# Screen Inventory

> Design status: V1 — draft, awaiting user review
>
> Implementation status: NOT STARTED

The inventory holds **18 active screens: 11 pages (including the system page SCR-90) and 7 modals**. Of the 11 pages, 6 are shared or belong to other features (SCR-01, SCR-90, SCR-91, SCR-T-01..03) and 5 are workflow-specific. SCR-WF-10 was removed with the Team Lead (D-IA-07) and SCR-WF-04 with the assignment rules (D-IA-13); their rows stay and their numbers are not reused. Screen IDs are numbered by area (DF-IA-04). "Opened from": `Navigation` means the sidebar, the top bar or the post-login redirect; `SCR-xx` is the parent page with the button or link. A modal is always opened from a page, never from another modal. Screens marked *(F03)* or *(F01)* belong to other features; only what the workflow needs is listed here.

## Inventory

| ID | Screen | Presentation | Roles | Opened from | Main content | Related stories |
|---|---|---|---|---|---|---|
| `SCR-01` | Login *(F01)* | Page | All roles (public) | Navigation | Email, password, Log in. After login, goes to the role home page. | — |
| `SCR-90` | Access denied | System page | All roles | Navigation | The message "You do not have permission to access this page" and a link to the home page. Never reveals whether the object exists. | — |
| `SCR-91` | Notifications | Page | All roles | Navigation (top bar) | Newest first, "Unread only" filter. Each item: time, content, link to its object. | US-WF-09, 10, 12, 13, 14, 18 |
| `SCR-T-01` | Create ticket *(F03)* | Page | All roles | Navigation (+ Create ticket), SCR-T-03 | Free-text description; quick templates (Access request, New account, Account check, Broken device) pre-fill hints to reduce AI clarification rounds; attachments. | US-WF-08 |
| `SCR-T-02` | Ticket detail *(F03)* | Page | Per ticket scope | SCR-T-03, SCR-WF-31, SCR-91 | **AI conversation panel** (questions, answers); **progress bar**: Received → AI analyzing → Approved / More info needed / Handed to technician → Executing → Awaiting confirmation → Closed; decision **reason** (summary for the requester); **AI summary** for technicians; **SLA tab**: deadline, remaining time, pauses; **Automation tab** (Technician, Admin): API calls, error codes, *Retry* / *Resolved manually* buttons; *Confirm resolved* / *Not fixed* buttons (Requester); assignee with the workload score at assignment time (Technician, Admin); *Reassign* button (Admin only). | US-WF-04, 08, 09, 10, 11, 15, 17, 19, 20 |
| `SCR-T-03` | Ticket list *(F03)* | Page | All roles, scope by role | Navigation | Table: ID, title, status, flow (AI self-service / Clarification / Human), assignee, SLA deadline. Filters by status, flow, "waiting for my answer", "SLA at risk". | US-WF-08, 12 |
| `SCR-WF-01` | Workload settings | Page | Admin | Navigation | Points per priority (P1–P3), complexity of each request type (Simple / Medium / Complex) and its factor, maximum load per technician, default support team; a live view of each technician's current status (`ON` / `OFF`) and workload score; *Assignment preview* button. No assignment rules (D-IA-13). | US-WF-21 |
| `SCR-WF-02` | SLA & business calendar | Page (2 tabs) | Admin | Navigation | *SLA policies* tab: P1–P3 table (High, Medium, Low) edited inline (response, resolution, warning threshold), raise-priority-on-breach option, version number. *Business calendar* tab: working hours per weekday, list of holidays. | US-WF-03, 04 |
| `SCR-WF-03` | AI scope | Page (3 tabs) | Admin | Navigation | *AI actions* tab: switches for `AUTO_CLASSIFY`, `ASK_FOR_INFO`, `AUTO_APPROVE`, `SUGGEST_KB`, `SUGGEST_PRIORITY`; confidence threshold; maximum clarification rounds. *Automation actions* tab: whitelist (operation, connector, parameters, maximum duration, fallback team on failure). *Entitlement catalog* tab: choose a department → table of entitlements / account templates the AI may grant. | US-WF-05, 06, 07 |
| `SCR-WF-04` | ~~Assignment rule form~~ | — | — | — | **Removed (D-IA-13).** No assignment rules: the team comes from the AI category. Number not reused. | — |
| `SCR-WF-05` | Assignment preview | Modal | Admin | SCR-WF-01 | Choose a sample ticket or enter a description → shows the AI category, the responsible team (or the default support team), each member's status and workload score, who is excluded and why, and the expected technician. States clearly *"A preview does not assign any real ticket"*. | US-WF-21 |
| `SCR-WF-06` | Automation action form | Modal | Admin | SCR-WF-03 (Automation actions tab) | Request type, operation (`CHECK_ACCOUNT`, `CREATE_ACCOUNT`, `GRANT_ACCESS`), connector (chosen from the declared list, no secrets entered), required parameters, maximum duration, fallback team on failure. | US-WF-06 |
| `SCR-WF-07` | Department entitlement form | Modal | Admin | SCR-WF-03 (Entitlement catalog tab) | Department, system, entitlement or account template, maximum duration. | US-WF-07 |
| `SCR-WF-10` | ~~Triage queue~~ | — | — | — | **Removed (D-IA-07).** No Team Lead and no manual triage: tickets whose category has no responsible team go to the default support team. Number not reused. | — |
| `SCR-WF-11` | Reassign | Modal | Admin only | SCR-T-02 | Last-resort manual reassignment: choose a team and a technician, showing each person's availability status and workload score; **reason required**. | US-WF-11, 13 |
| `SCR-WF-20` | Retry execution | Modal | Assigned technician, Admin | SCR-T-02 (Automation tab) | Summary of the previous error; a sentence stating the effect: *"The system will re-check the current policy and call the API again. No duplicate grant."*; *Retry* button. | US-WF-20 |
| `SCR-WF-21` | Mark resolved manually | Modal | Assigned technician, Admin | SCR-T-02 (Automation tab) | **Required** note (what was done on the external system); the ticket becomes *Awaiting confirmation*. | US-WF-20 |
| `SCR-WF-22` | Report not fixed | Modal | Requester (ticket owner) | SCR-T-02 | **Required** reason (e.g. *"I still cannot open the folder"*); the ticket is handed to a technician of the owning team. | US-WF-19 |
| `SCR-WF-30` | AI decisions | Page | Admin | Navigation, SCR-91 | Daily metric cards: **AI self-resolution rate**, clarification rate, human-routing rate, AI unavailability count. Decision table: time, ticket, request type, flow, confidence, reason; filters by flow, type, date, department. | US-WF-16 |
| `SCR-WF-31` | AI decision detail | Page | Admin | SCR-WF-30, SCR-T-02 | Original ticket content; **AI-extracted information** (beneficiary, system, entitlement, duration) and confidence; **each policy condition** with ✓ / ✗ and the compared values; configuration version used; clarification rounds; API execution history. Read-only. | US-WF-15, 16 |

**Note on SLA:** SCR-WF-02 edits the table inline on the page, without a modal, because it has only 3 fixed rows (P1–P3).

## Requirement-to-screen mapping

| Requirement | Primary screen / action |
|---|---|
| US-WF-01 | Removed (no assignment rules) |
| US-WF-02 | Removed with US-WF-01 |
| US-WF-03 | SCR-WF-02 edit the SLA table, Business calendar tab |
| US-WF-04 | Result visible on SCR-T-02 SLA tab (pauses) |
| US-WF-05 | SCR-WF-03 AI actions tab |
| US-WF-06 | SCR-WF-03 Automation actions tab → SCR-WF-06 |
| US-WF-07 | SCR-WF-03 Entitlement catalog tab → SCR-WF-07 |
| US-WF-08 | SCR-T-01 create a ticket → SCR-T-02 follow progress |
| US-WF-09 | SCR-T-02 conversation panel: answer the AI question |
| US-WF-10 | Notification SCR-91 → SCR-T-02 (AI summary and routing reason) |
| US-WF-11 | Top bar availability switch (D-IA-11); result visible on SCR-T-02 (assignee) and in notification SCR-91 when a ticket is moved; Admin fallback SCR-WF-11 |
| US-WF-12 | Notification SCR-91; SCR-T-03 "SLA at risk" filter |
| US-WF-13 | Notification SCR-91 for the Admin; breach flag on SCR-T-02, SCR-T-03; automatic reassignment visible on SCR-T-02 |
| US-WF-14 | Notification SCR-91; SCR-T-02 shows the auto-close deadline and the reopen button |
| US-WF-15 | Result visible on SCR-T-02 (progress bar *Approved*) and SCR-WF-31 |
| US-WF-16 | SCR-WF-30 → SCR-WF-31 |
| US-WF-17 | Result visible on SCR-T-02 (progress bar *Executing*, Automation tab) |
| US-WF-18 | Notification SCR-91 before revocation; the revocation appears in the Automation tab |
| US-WF-19 | SCR-T-02 *Confirm resolved* button; SCR-WF-22 *Not fixed* |
| US-WF-20 | SCR-T-02 Automation tab → SCR-WF-20 *Retry* or SCR-WF-21 *Resolved manually* |
| US-WF-21 | SCR-WF-01 Workload settings; SCR-WF-05 Assignment preview shows the team and the scores |

Stories that describe system behaviour (US-WF-04, 15, 17, 18) have no screen of their own; the table names where the user **sees the result**.

**Reachability of Must stories:** every Must story is reachable from the home page of its role in at most 2 steps. If Should/Could stories are cut, only these are lost: SLA pausing (US-WF-04; the Business calendar tab stays), reminders and auto-close (US-WF-14), and the revocation notice (US-WF-18). No home page depends on a Should story.

## Screen-to-API mapping

operationIds are **proposed** and will be finalized with the OpenAPI contract.

| Screen | API operations |
|---|---|
| SCR-01 Login | `login`, `logout`, `getSession` |
| SCR-90 Access denied | `getSession` |
| SCR-91 Notifications | `listNotifications`, `getUnreadNotificationCount`, `markNotificationRead` |
| SCR-T-01 Create ticket | `createTicket`, `listTicketTemplates` |
| SCR-T-02 Ticket detail | `getTicket`, `listTicketMessages`, `answerClarification`, `getTicketSla`, `getCurrentAiDecisionSummary`, `listWorkflowRuns`, `getWorkflowRun`, `confirmTicketResolution`, `reopenTicket` |
| SCR-T-03 Ticket list | `listTickets`, `getTicketCounts` |
| SCR-WF-01 Workload settings | `getWorkloadSettings`, `updateWorkloadSettings`, `listTechnicianWorkload`, `listTeams` |
| SCR-WF-02 SLA & business calendar | `listSlaPolicies`, `updateSlaPolicy`, `getBusinessCalendar`, `updateBusinessCalendar` |
| SCR-WF-03 AI scope | `listAiActionSettings`, `updateAiActionSetting`, `listAutomationActions`, `updateAutomationAction`, `listDepartmentEntitlements`, `listConnectors`, `listDepartments` |
| SCR-WF-05 Assignment preview | `previewAssignment` |
| SCR-WF-06 Automation action form | `createAutomationAction`, `updateAutomationAction`, `listConnectors`, `listTeams` |
| SCR-WF-07 Department entitlement form | `updateDepartmentEntitlements`, `listConnectors` |
| SCR-WF-11 Reassign | `reassignTicket`, `listTeams`, `listTechnicianWorkload` |
| Top bar availability switch (F09) | `getMyAvailability`, `updateMyAvailability` |
| SCR-WF-20 Retry execution | `retryWorkflowRun` |
| SCR-WF-21 Mark resolved manually | `resolveWorkflowRunManually` |
| SCR-WF-22 Report not fixed | `reopenTicket` |
| SCR-WF-30 AI decisions | `listAiDecisions`, `getAiDecisionMetrics` |
| SCR-WF-31 AI decision detail | `getAiDecision`, `listWorkflowRuns` |

**Related:** [Information architecture index](README.md) · [Sitemap](01-sitemap.md) · [Routes and permissions](02-routes-and-permissions.md) · [Traceability](../../traceability/master-traceability.md)
