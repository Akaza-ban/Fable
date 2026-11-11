# Screens, Routes, and Navigation — F04 AI Ticket Processing

> Design status: TARGET V1
>
> Related: [IA overview](README.md) · [User flows](02-user-flows.md) · [UI states](03-ui-states-and-accessibility.md)

## 1. Host pages and their F04 sections

Host pages belong to other branches. F04 only adds the sections listed here. Routes of host pages are **proposed** until their owners publish them.

| Host page | Owner | Proposed route | F04 sections |
|---|---|---|---|
| SCR-T-01 Create ticket | F03 | `/tickets/new` | SEC-AI-01 |
| SCR-T-04 Work queues | F03 | `/tickets` | SEC-AI-14 |
| SCR-T-02 Ticket detail | F03 | `/tickets/:id` | SEC-AI-03, 04, 05, 07, 09, 11, 16, 17, 18, 19 |
| Resolve dialog (DLG-T-05) | F03 | modal on `/tickets/:id` | SEC-AI-13 |

### Sections

| ID | Section | Host | Content | Stories |
|---|---|---|---|---|
| SEC-AI-01 | KB suggestions while writing | SCR-T-01 | Up to 3 articles (title, one-line reason, link) after 20 characters; *This solved my problem* (no ticket is created); "No matching article" / "Suggestions unavailable" | US-AI-05 |
| SEC-AI-03 | AI messages | SCR-T-02 timeline | The agent's KB replies with *Confirm* / *Not solved*; result replies after execution; the last line of a guidance session. Every AI message has the `AI` author badge | US-AI-16, US-AI-17 |
| SEC-AI-04 | KB suggestions (requester) | SCR-T-02 | Up to 3 articles for all users; *This solved my problem* → SCR-AI-05; hidden in `AUTOMATING`; helpful / not helpful | US-AI-05, US-AI-14 |
| SEC-AI-05 | Escalation banner | SCR-T-02 top | Escalation class, reason, who set it (F04 agent or F05), what the agent already did (KB reply, guidance transcript, proposed action and its run); for a suspected bug, the linked tickets of the burst | US-AI-20, US-AI-21, US-AI-27 |
| SEC-AI-07 | Domain form | SCR-T-02 details panel | The agent and its `formFields` (for example company code, transaction, error for SAP); "Filled by AI" marker; source quote on hover / focus; *Correct* → SCR-AI-07 | US-AI-20, US-AI-14 |
| SEC-AI-09 | Solution steps | SCR-T-02 | Up to 6 steps, each with a citation `[KB-087 §3]`; *Insert into reply* (editable draft); *Find solutions again*; helpful / not helpful | US-AI-06, US-AI-14 |
| SEC-AI-11 | Requirement brief | SCR-T-02 tab | The brief in eight sections, its state (draft / confirmed / not confirmed); requester: *Review and confirm* → SCR-AI-04; technician: edit | US-AI-18 |
| SEC-AI-13 | Resolution and reply draft | Resolve dialog | Drafted Resolution (at most 80 words) and reply, editable; "Please describe" where nothing is known; marker "Drafted by AI" | US-AI-16 |
| SEC-AI-14 | AI columns and filters | SCR-T-04 | Columns: escalation class, agent outcome, automation status, review state. Filters: escalation class, domain, suspected-bug burst, failed automation, resolved by AI (with *Not reviewed*) | US-AI-20, US-AI-21, US-AI-25, US-AI-28 |
| SEC-AI-16 | Automation run | SCR-T-02 tab *Automation* | Operation and parameters (from the approved decision), approval source (F05 policy or an Administrator in F03), run status, attempts (time, result code, duration), verification result, error code and failed step; *Retry*; *Resolved manually* → SCR-AI-06. Requester: one status line in plain words ("Being processed", "Done", "A technician is taking over") | US-AI-22, US-AI-23, US-AI-24, US-AI-25 |
| SEC-AI-17 | Guided troubleshooting | SCR-T-02 | Requester: the current step card (step *n*, what to do, citation, *It works now* · *Still the same* · *I see something else*, free text, *I need a person*) and earlier steps. Technician and Admin: the full transcript with findings | US-AI-26, US-AI-27 |
| SEC-AI-18 | Agent results | SCR-T-02 tab *AI* (F03 / F05) | Every agent result version: outcome, form, reason, `examplesUsed`, model and prompt version, status (`COMPLETED` / `INVALID` / `FAILED`) with the failed check | US-AI-20, US-AI-19 |
| SEC-AI-19 | AI resolution review | SCR-T-02 top, on tickets the agent resolved alone | How the agent resolved it (outcome, article or guidance transcript, executed action with approval and verification, Resolution); review state and reviewer; *Looks right*; *Resolved incorrectly* with a required reason. Read-only ticket otherwise | US-AI-28 |

## 2. F04 pages and modals

| ID | Name | Type | Opened from | Route | Content | Stories |
|---|---|---|---|---|---|---|
| SCR-AI-01 | Learning examples | Page | Admin sidebar, *AI* group | `/admin/ai/learning-examples` | Table (domain, how it was resolved, source ticket, date, origin: technician / confirmed agent / requester confirmed, used count); filter by domain and origin; *Exclude* with reason; excluded examples shown greyed | US-AI-19 |
| SCR-AI-04 | Review requirement brief | Modal | SEC-AI-11, EML-AI-05 | `/tickets/:id?action=review-brief` | Editable brief; *Confirm brief*; *Not now* | US-AI-18 |
| SCR-AI-05 | Confirm resolution | Modal | SEC-AI-03, SEC-AI-04, SEC-AI-17, EML-AI-01, 03, 04, 06, 07 | `/tickets/:id?action=solved&ref=:messageId` or `?action=not-solved&ref=:messageId` | The article, AI reply, guidance result, result reply or technician reply being confirmed; *Yes, this solved it* / *No, I still need help*; one POST on click (D-AI-IA-02). KB answers call the F03 close; other replies call F05 `confirmTicketResolution` / `reopenTicket` | US-AI-05, US-AI-16, US-AI-17, US-AI-24, US-AI-26 |
| SCR-AI-06 | Resolve run manually | Modal | SEC-AI-16 *Resolved manually* | on `/tickets/:id` | What was done (required note); then opens the Resolve dialog with the AI draft | US-AI-25 |
| SCR-AI-07 | Correct domain form | Modal | SEC-AI-07 *Correct* | on `/tickets/:id` | The domain form fields; note "This does not start any action or re-route the ticket" | US-AI-14 |

`ref` is the id of the message, step or KB suggestion being confirmed. The modal opens pre-selected but never acts until the user clicks. A guidance answer from an email link (EML-AI-07) opens SEC-AI-17 with the answer pre-selected; it is saved only on click.

## 3. Emails

The AI writes the text; F03 sends the emails. Every link follows D-AI-IA-02.

| ID | Email | Sent when | Links | Stories |
|---|---|---|---|---|
| EML-AI-01 | Acknowledgement with KB suggestions | An email ticket is routed and at least one article is above the threshold | Each article (F06 page); *This solved my problem* → SCR-AI-05; *I still need help* → SCR-T-02 | US-AI-05 |
| EML-AI-03 | Agent KB reply | The agent's outcome is `KB_ANSWER` | Cited steps with article links; *Confirm* / *Not solved* → SCR-AI-05 | US-AI-17 |
| EML-AI-07 | Guidance step | The agent sends a guidance step to an email requester | The step with its article link; *It works now* / *Still the same* / *I see something else* → SEC-AI-17; or reply to the email (F03 threads it) | US-AI-26 |
| EML-AI-04 | Result reply after execution | UC-AI-13 verified a run, or a `CHECK_ACCOUNT` returned | *Confirm* / *Not solved* → SCR-AI-05; *Open ticket* → SCR-T-02 | US-AI-16, US-AI-24 |
| EML-AI-08 | Access ends tomorrow | One day before a time-bound grant is revoked | *Open ticket* → SCR-T-02; how to ask for an extension (new ticket) | US-AI-23 |
| EML-AI-05 | Requirement brief to confirm | A brief is drafted for a `BUSINESS_REQUIREMENT` | *Review and confirm* → SCR-AI-04 | US-AI-18 |
| EML-AI-06 | Technician reply | A technician resolves the ticket with the AI draft | *Confirm* / *Not solved* → SCR-AI-05; *Open ticket* → SCR-T-02 (sent in the technician's name) | US-AI-16 |

## 4. Routes and permissions

The API enforces every row; hiding an element is only for usability. "Scope" means tickets assigned to the technician or one of their groups (F01 US-03).

| Item | Requester | Technician | Admin | Unauthorized result |
|---|---|---|---|---|
| SEC-AI-01 | ✓ | ✓ | ✓ | — |
| SEC-AI-03, SEC-AI-04 | Own tickets | Scope | ✓ | `403` |
| SEC-AI-17 | Own tickets (answer) | Scope (read) | ✓ (read) | `403` |
| SEC-AI-16 | Own tickets (status line only) | Scope; *Retry* / *Resolved manually* for the assignee | ✓ | Fields absent from the response; actions `403` |
| SEC-AI-05, SEC-AI-07, SEC-AI-09 | ✗ not rendered, not in the API response | Scope | ✓ | Field absent from the response |
| SEC-AI-11 | Own tickets (review, confirm) | Scope (read, edit) | ✓ | `403` |
| SEC-AI-13 | ✗ | Scope | ✓ | `403` |
| SEC-AI-14 | ✗ (AI columns not shown in *My tickets*) | ✓ | ✓ | — |
| SEC-AI-18 | ✗ | ✗ | ✓ | `403` |
| SEC-AI-19 | ✗ not rendered | Review scope (BR-AI-16) | ✓ | `403` |
| SCR-AI-01 `/admin/ai/learning-examples` | ✗ | ✗ | ✓ | `#/403` page |
| SCR-AI-04, SCR-AI-05 | Own tickets | Read only | Read only | `403` |
| SCR-AI-06 | ✗ | Assignee | ✓ | `403` |
| SCR-AI-07 | ✗ | Scope | ✓ | `403` |

A requester who opens an email link to someone else's ticket gets `#/403`, and nothing changes. A technician who opens a ticket through the review scope sees the ticket read-only with SEC-AI-19; every action that changes the ticket is hidden and returns `403`.

## 5. Navigation

| Role | Sidebar | What changes because of F04 |
|---|---|---|
| Requester | My tickets, New ticket, Profile (F01, F03) | Nothing new in the sidebar |
| Technician | Work queues, Profile | Work queues get AI columns and filters (SEC-AI-14). Saved filters *Suspected bugs*, *Failed automation* and *Resolved by AI* |
| Admin | Dashboard, Tickets, Users, Groups, …, **AI** | One **AI** group: AI scope (SCR-WF-03, F05) · AI decisions (SCR-WF-30, F05) · Learning examples (SCR-AI-01, F04) · AI suggestions (F01) · Service accounts (F01) |

Contextual navigation inside SCR-T-02: the escalation banner links to the guidance transcript, the automation tab and the AI messages; a citation opens the KB article (F06) in a new tab; a linked burst ticket opens its own SCR-T-02 (if in scope); the Admin sees *View agent results* → SEC-AI-18.

## 6. Story-to-screen mapping

Every Must story is reachable by its role.

| Story | MoSCoW | Role | Screens / sections / emails |
|---|---|---|---|
| US-AI-05 | Must | Requester | SEC-AI-01, SEC-AI-04, EML-AI-01, SCR-AI-05 |
| US-AI-06 | Must | Technician | SEC-AI-09 |
| US-AI-17 | Must | Requester | SEC-AI-03, EML-AI-03, SCR-AI-05 |
| US-AI-20 | Must | Requester, Technician | SEC-AI-05, SEC-AI-07, SEC-AI-14, SEC-AI-18 |
| US-AI-21 | Should | Technician | SEC-AI-05, SEC-AI-14 |
| US-AI-26 | Should | Requester | SEC-AI-17, EML-AI-07, SCR-AI-05 |
| US-AI-27 | Should | Technician | SEC-AI-17 transcript, SEC-AI-05 |
| US-AI-22 | Must | Requester | SEC-AI-16 status line |
| US-AI-23 | Could | Admin, Requester | SEC-AI-16, EML-AI-08 |
| US-AI-24 | Must | Requester | SEC-AI-16, EML-AI-04, SCR-AI-05 |
| US-AI-25 | Must | Technician | SEC-AI-14 filter, SEC-AI-05, SEC-AI-16, SCR-AI-06 |
| US-AI-16 | Must | Technician, Requester | SEC-AI-13, SEC-AI-03, EML-AI-04, EML-AI-06, SCR-AI-05 |
| US-AI-18 | Should | Requester, Technician | SEC-AI-11, SCR-AI-04, EML-AI-05 |
| US-AI-14 | Should | Technician, Requester | SEC-AI-04, SEC-AI-07, SEC-AI-09, SEC-AI-17, SCR-AI-07 |
| US-AI-28 | Should | Technician | SEC-AI-14 filter *Resolved by AI*, SEC-AI-19 |
| US-AI-19 | Should | Admin | SCR-AI-01, SEC-AI-18 |

## 7. Requests to other branches

| Owner | Request |
|---|---|
| F03 | Host the sections above on SCR-T-01, SCR-T-02, SCR-T-04 and the Resolve dialog, including a read-only SCR-T-02 for the review scope (SEC-AI-19), including the new *Automation* tab (SEC-AI-16) and the guidance card (SEC-AI-17); `?action=` handling on `/tickets/:id` (D-AI-IA-02); send EML-AI-07 and EML-AI-08; the resolution code `GUIDED_BY_AI` (Q-AI-18) |
| F05 | Remove the *Automation* tab from its own design of SCR-T-02 (now SEC-AI-16); the settings `AUTO_RESOLVE_KB` and `GUIDE_L2` on SCR-WF-03 (Q-AI-10, Q-AI-15); take over SEC-AI-15 and the AI-filled ticket fields |
| F01 | The **AI** sidebar group for Admin; SSO return to the original link after login; the read-only **review scope** in the ticket scope check (BR-AI-16) |
| F06 | Article pages that open at a section (`#§3`) for citations; the self-service flag; troubleshooting procedures with a check and a next step for each step, and a flag on steps that need admin rights |
