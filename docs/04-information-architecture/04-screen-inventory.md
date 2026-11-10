# Screen Inventory

> Design status: V1 — draft, awaiting user review
>
> Implementation status: NOT STARTED

The inventory holds **20 screens: 12 pages (including the system page SCR-90) and 8 modals**. Of the 12 pages, 6 are shared or belong to other features (SCR-01, SCR-90, SCR-91, SCR-T-01..03) and 6 are workflow-specific. Screen IDs are numbered by area (DF-IA-04). "Opened from": `Navigation` means the sidebar, the top bar or the post-login redirect; `SCR-xx` is the parent page with the button or link. A modal is always opened from a page, never from another modal. Screens marked *(F03)* or *(F01)* belong to other features; only what the workflow needs is listed here.

## Inventory

| ID | Screen | Presentation | Roles | Opened from | Main content | Related stories |
|---|---|---|---|---|---|---|
| `SCR-01` | Login *(F01)* | Page | All roles (public) | Navigation | Email, password, Log in. After login, goes to the role home page. | — |
| `SCR-90` | Access denied | System page | All roles | Navigation | The message "You do not have permission to access this page" and a link to the home page. Never reveals whether the object exists. | — |
| `SCR-91` | Notifications | Page | All roles | Navigation (top bar) | Newest first, "Unread only" filter. Each item: time, content, link to its object. | US-WF-09, 10, 12, 13, 14, 18 |
| `SCR-T-01` | Create ticket *(F03)* | Page | All roles | Navigation (+ Create ticket), SCR-T-03 | Free-text description; quick templates (Access request, New account, Account check, Broken device) pre-fill hints to reduce AI clarification rounds; attachments. | US-WF-08 |
| `SCR-T-02` | Ticket detail *(F03)* | Page | Per ticket scope | SCR-T-03, SCR-WF-10, SCR-WF-31, SCR-91 | **AI conversation panel** (questions, answers); **progress bar**: Received → AI analyzing → Approved / More info needed / Handed to technician → Executing → Awaiting confirmation → Closed; decision **reason** (summary for the requester); **AI summary** for technicians; **SLA tab**: deadline, remaining time, pauses; **Automation tab** (Technician, Admin): API calls, error codes, *Retry* / *Resolved manually* buttons; *Confirm resolved* / *Not fixed* buttons (Requester); *Reassign* button (Team Lead, Admin). | US-WF-04, 08, 09, 10, 15, 17, 19, 20 |
| `SCR-T-03` | Ticket list *(F03)* | Page | All roles, scope by role | Navigation | Table: ID, title, status, flow (AI self-service / Clarification / Human), assignee, SLA deadline. Filters by status, flow, "waiting for my answer", "SLA at risk". | US-WF-08, 12 |
| `SCR-WF-01` | Assignment rules | Page | Admin | Navigation | Ordered rule list (drag and drop); each row: name, conditions, target team, distribution strategy, Enable/Disable switch; *Add*, *Dry run*, *Archive* buttons. | US-WF-01, 02 |
| `SCR-WF-02` | SLA & business calendar | Page (2 tabs) | Admin | Navigation | *SLA policies* tab: P1–P4 table edited inline (response, resolution, warning threshold), raise-priority-on-breach option, version number. *Business calendar* tab: working hours per weekday, list of holidays. | US-WF-03, 04 |
| `SCR-WF-03` | AI scope | Page (3 tabs) | Admin | Navigation | *AI actions* tab: switches for `AUTO_CLASSIFY`, `ASK_FOR_INFO`, `AUTO_APPROVE`, `SUGGEST_KB`, `SUGGEST_PRIORITY`; confidence threshold; maximum clarification rounds. *Automation actions* tab: whitelist (operation, connector, parameters, maximum duration, fallback team on failure). *Entitlement catalog* tab: choose a department → table of entitlements / account templates the AI may grant. | US-WF-05, 06, 07 |
| `SCR-WF-04` | Assignment rule form | Modal | Admin | SCR-WF-01 | Name; conditions (request type, category, priority, requester department — combined with AND); target team; strategy (`Round robin` / `Least loaded`). A new rule is saved as Disabled. | US-WF-01 |
| `SCR-WF-05` | Rule dry run | Modal | Admin | SCR-WF-01 | Choose a sample ticket or enter attributes → shows the matching rule, team and expected technician. States clearly *"A dry run does not assign any real ticket"*. | US-WF-02 |
| `SCR-WF-06` | Automation action form | Modal | Admin | SCR-WF-03 (Automation actions tab) | Request type, operation (`CHECK_ACCOUNT`, `CREATE_ACCOUNT`, `GRANT_ACCESS`), connector (chosen from the declared list, no secrets entered), required parameters, maximum duration, fallback team on failure. | US-WF-06 |
| `SCR-WF-07` | Department entitlement form | Modal | Admin | SCR-WF-03 (Entitlement catalog tab) | Department, system, entitlement or account template, maximum duration. | US-WF-07 |
| `SCR-WF-10` | Triage queue | Page | Admin, Team Lead | Navigation, SCR-91 | Tickets in `TRIAGE`: ID, AI summary, why no rule matched, remaining SLA; *Assign* button. Sorted by remaining SLA ascending by default. | US-WF-11 |
| `SCR-WF-11` | Reassign | Modal | Admin; Team Lead (within own team) | SCR-WF-10, SCR-T-02 | Choose a team (Admin) or a member (Team Lead), showing each person's open-ticket count; **reason required**. | US-WF-11 |
| `SCR-WF-20` | Retry execution | Modal | Assigned technician, Admin | SCR-T-02 (Automation tab) | Summary of the previous error; a sentence stating the effect: *"The system will re-check the current policy and call the API again. No duplicate grant."*; *Retry* button. | US-WF-20 |
| `SCR-WF-21` | Mark resolved manually | Modal | Assigned technician, Admin | SCR-T-02 (Automation tab) | **Required** note (what was done on the external system); the ticket becomes *Awaiting confirmation*. | US-WF-20 |
| `SCR-WF-22` | Report not fixed | Modal | Requester (ticket owner) | SCR-T-02 | **Required** reason (e.g. *"I still cannot open the folder"*); the ticket is handed to a technician of the owning team. | US-WF-19 |
| `SCR-WF-30` | AI decisions | Page | Admin | Navigation, SCR-91 | Daily metric cards: **AI self-resolution rate**, clarification rate, human-routing rate, AI unavailability count. Decision table: time, ticket, request type, flow, confidence, reason; filters by flow, type, date, department. | US-WF-16 |
| `SCR-WF-31` | AI decision detail | Page | Admin | SCR-WF-30, SCR-T-02 | Original ticket content; **AI-extracted information** (beneficiary, system, entitlement, duration) and confidence; **each policy condition** with ✓ / ✗ and the compared values; configuration version used; clarification rounds; API execution history. Read-only. | US-WF-15, 16 |

**Note on SLA:** SCR-WF-02 edits the table inline on the page, without a modal, because it has only 4 fixed rows.

## Requirement-to-screen mapping

| Requirement | Primary screen / action |
|---|---|
| US-WF-01 | SCR-WF-01 → SCR-WF-04 create / edit a rule |
| US-WF-02 | SCR-WF-01 drag to reorder; SCR-WF-05 dry run |
| US-WF-03 | SCR-WF-02 edit the SLA table, Business calendar tab |
| US-WF-04 | Result visible on SCR-T-02 SLA tab (pauses) |
| US-WF-05 | SCR-WF-03 AI actions tab |
| US-WF-06 | SCR-WF-03 Automation actions tab → SCR-WF-06 |
| US-WF-07 | SCR-WF-03 Entitlement catalog tab → SCR-WF-07 |
| US-WF-08 | SCR-T-01 create a ticket → SCR-T-02 follow progress |
| US-WF-09 | SCR-T-02 conversation panel: answer the AI question |
| US-WF-10 | Notification SCR-91 → SCR-T-02 (AI summary and routing reason) |
| US-WF-11 | SCR-WF-10 triage queue → SCR-WF-11 reassign |
| US-WF-12 | Notification SCR-91; SCR-T-03 "SLA at risk" filter |
| US-WF-13 | Notification SCR-91 for the Team Lead; breach flag on SCR-T-02, SCR-T-03 |
| US-WF-14 | Notification SCR-91; SCR-T-02 shows the auto-close deadline and the reopen button |
| US-WF-15 | Result visible on SCR-T-02 (progress bar *Approved*) and SCR-WF-31 |
| US-WF-16 | SCR-WF-30 → SCR-WF-31 |
| US-WF-17 | Result visible on SCR-T-02 (progress bar *Executing*, Automation tab) |
| US-WF-18 | Notification SCR-91 before revocation; the revocation appears in the Automation tab |
| US-WF-19 | SCR-T-02 *Confirm resolved* button; SCR-WF-22 *Not fixed* |
| US-WF-20 | SCR-T-02 Automation tab → SCR-WF-20 *Retry* or SCR-WF-21 *Resolved manually* |

Stories that describe system behaviour (US-WF-04, 15, 17, 18) have no screen of their own; the table names where the user **sees the result**.

**Reachability of Must stories:** every Must story is reachable from the home page of its role in at most 2 steps. If Should/Could stories are cut, only these are lost: the SCR-WF-05 dry run (US-WF-02), SLA pausing (US-WF-04; the Business calendar tab stays), reminders and auto-close (US-WF-14), and the revocation notice (US-WF-18). No home page depends on a Should story.

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
| SCR-WF-01 Assignment rules | `listAssignmentRules`, `updateAssignmentRule`, `reorderAssignmentRules`, `archiveAssignmentRule` |
| SCR-WF-02 SLA & business calendar | `listSlaPolicies`, `updateSlaPolicy`, `getBusinessCalendar`, `updateBusinessCalendar` |
| SCR-WF-03 AI scope | `listAiActionSettings`, `updateAiActionSetting`, `listAutomationActions`, `updateAutomationAction`, `listDepartmentEntitlements`, `listConnectors`, `listDepartments` |
| SCR-WF-04 Assignment rule form | `createAssignmentRule`, `updateAssignmentRule`, `listTeams` |
| SCR-WF-05 Rule dry run | `dryRunAssignmentRules` |
| SCR-WF-06 Automation action form | `createAutomationAction`, `updateAutomationAction`, `listConnectors`, `listTeams` |
| SCR-WF-07 Department entitlement form | `updateDepartmentEntitlements`, `listConnectors` |
| SCR-WF-10 Triage queue | `listTriageQueue` |
| SCR-WF-11 Reassign | `reassignTicket`, `listTeams`, `listTeamMembersWithLoad` |
| SCR-WF-20 Retry execution | `retryWorkflowRun` |
| SCR-WF-21 Mark resolved manually | `resolveWorkflowRunManually` |
| SCR-WF-22 Report not fixed | `reopenTicket` |
| SCR-WF-30 AI decisions | `listAiDecisions`, `getAiDecisionMetrics` |
| SCR-WF-31 AI decision detail | `getAiDecision`, `listWorkflowRuns` |

**Related:** [Information architecture index](README.md) · [Sitemap](01-sitemap.md) · [Routes and permissions](02-routes-and-permissions.md) · [Traceability](../traceability/master-traceability.md)
