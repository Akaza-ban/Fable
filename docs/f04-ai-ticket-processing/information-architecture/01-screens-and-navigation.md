# Screens, Routes, and Navigation — F04 AI Ticket Processing

> Design status: TARGET V1
>
> Related: [IA overview](README.md) · [User flows](02-user-flows.md) · [UI states](03-ui-states-and-accessibility.md)

## 1. Host pages and their F04 sections

Host pages belong to other branches. F04 only adds the sections listed here. Routes of host pages are **proposed** until their owners publish them.

| Host page | Owner | Proposed route | F04 sections |
|---|---|---|---|
| SCR-T-01 Create ticket | F03 | `/tickets/new` | SEC-AI-01, SEC-AI-02. The form asks only for free text (subject, description, attachments); no category picker (AC-AI-20.7) |
| Ticket list (my tickets / assigned tickets / all tickets) | F03 | `/tickets` | SEC-AI-14 |
| SCR-T-02 Ticket detail | F03 | `/tickets/:id` | SEC-AI-03 … SEC-AI-12 |
| Resolve dialog | F03 | modal on `/tickets/:id` | SEC-AI-13 |
| SCR-WF-31 AI decision detail | F05 | defined by F05 | SEC-AI-15 |

### Sections

| ID | Section | Host | Content | Stories |
|---|---|---|---|---|
| SEC-AI-01 | KB suggestions while writing | SCR-T-01 | Up to 3 articles (title, one-line reason, link) after 20 characters; *This solved my problem* (no ticket is created); "No matching article" / "Suggestions unavailable" | US-AI-05 |
| SEC-AI-02 | Duplicate warning | SCR-T-01 | "You already have an open ticket that looks the same: #T-…" with *Open #T-…* and *Submit anyway*; only the requester's own open tickets | US-AI-12 |
| SEC-AI-03 | AI conversation | SCR-T-02 timeline | AI questions (with *Yes / No* for a confirmation question) and a reply box; agent KB replies with *Confirm* / *Not solved*; result replies after automation. Every AI message has the `AI` author badge | US-AI-03, US-AI-04, US-AI-16, US-AI-17 |
| SEC-AI-04 | KB suggestions (requester) | SCR-T-02 | Up to 3 articles for all users; *This solved my problem* → SCR-AI-05; hidden in `AUTOMATING`; helpful / not helpful | US-AI-05, US-AI-14 |
| SEC-AI-05 | Escalation banner | SCR-T-02 top | Escalation class, reason, who set it (F04 agent or F05), what the agent already did (questions, KB reply, proposed action) | US-AI-20, US-AI-21 |
| SEC-AI-06 | AI summary | SCR-T-02 | Seven sections, at most 150 words; each fact links to its message; "Updated after a new message"; earlier versions; *Regenerate*; helpful / not helpful | US-AI-07, US-AI-08, US-AI-14 |
| SEC-AI-07 | AI-filled fields and domain form | SCR-T-02 side panel | Request type, `domain/subcategory`, support level, requester (business unit, department, site), requested for, extracted fields, `formFields` of the domain agent; "Filled by AI" marker; source quote on hover / focus; *Correct* → SCR-AI-02 | US-AI-01, US-AI-02, US-AI-13, US-AI-15, US-AI-20 |
| SEC-AI-08 | Priority suggestion | SCR-T-02 side panel | Suggested P1–P4, reason, source (`AI` or person); *Accept*; *Change* → SCR-AI-03 | US-AI-09, US-AI-10 |
| SEC-AI-09 | Solution steps | SCR-T-02 | Up to 6 steps, each with a citation `[KB-087 §3]`; *Insert into reply* (editable draft); *Find solutions again*; helpful / not helpful | US-AI-06, US-AI-14 |
| SEC-AI-10 | Similar tickets | SCR-T-02 | Up to 5 tickets (number, title, status, similarity %, resolution); *Link as duplicate* (F03); *Not related*; for a suspected bug, the linked burst | US-AI-11, US-AI-21 |
| SEC-AI-11 | Requirement brief | SCR-T-02 tab | The brief in eight sections, its state (draft / confirmed / not confirmed); requester: *Review and confirm* → SCR-AI-04; technician: edit | US-AI-18 |
| SEC-AI-12 | Original email / cleaned text | SCR-T-02 | Toggle between the original email and `cleanBody`; sender, created by, requested for | US-AI-15 |
| SEC-AI-13 | Resolution and reply draft | Resolve dialog | Drafted Resolution (at most 80 words) and reply, editable; "Please describe" where nothing is known; marker "Drafted by AI" | US-AI-16 |
| SEC-AI-14 | AI columns and filters | Ticket list | Columns: AI category, escalation class, priority source. Filters: escalation class, domain, suspected-bug cluster. Requested change to F03 | US-AI-20, US-AI-21 |
| SEC-AI-15 | Analysis versions | SCR-WF-31 tab | Every analysis version: trigger, contract fields, sources, `examplesUsed`, model and prompt version, status (`COMPLETED` / `INVALID` / `FAILED`) with the failed check | US-AI-02 |

## 2. F04 pages and modals

| ID | Name | Type | Opened from | Route | Content | Stories |
|---|---|---|---|---|---|---|
| SCR-AI-01 | Learning examples | Page | Admin sidebar, *AI* group | `/admin/ai/learning-examples` | Table (domain, request type, category, source ticket, date, origin: corrected / confirmed, used count); filter by domain and origin; *Exclude* with reason; excluded examples shown greyed | US-AI-19 |
| SCR-AI-02 | Correct AI fields | Modal | SEC-AI-07 *Correct* | on `/tickets/:id` | Request type, `domain/subcategory`, extracted fields; only values from the seeded lists; note "This does not re-route the ticket" | US-AI-13 |
| SCR-AI-03 | Change priority | Modal | SEC-AI-08 *Change* | on `/tickets/:id` | New priority, reason (required); note that the SLA changes (F05) | US-AI-10 |
| SCR-AI-04 | Review requirement brief | Modal | SEC-AI-11, EML-AI-05 | `/tickets/:id?action=review-brief` | Editable brief; *Confirm brief*; *Not now* | US-AI-18 |
| SCR-AI-05 | Confirm resolution | Modal | SEC-AI-03, SEC-AI-04, EML-AI-01, EML-AI-03, EML-AI-04, EML-AI-06 | `/tickets/:id?action=solved&ref=:messageId` or `?action=not-solved&ref=:messageId` | The article, AI reply, result reply or technician reply being confirmed; *Yes, this solved it* / *No, I still need help*; one POST on click (D-AI-IA-02). For KB answers it calls the F04/F03 close; for automation and technician replies it calls F05 `confirmTicketResolution` / `reopenTicket` (UC-WF-09) | US-AI-05, US-AI-16, US-AI-17 |

`ref` is the id of the message or KB suggestion being confirmed. The modal opens pre-selected but never acts until the user clicks.

## 3. Emails

The AI writes the text; F03 sends acknowledgements and technician replies, F05 sends workflow messages. Every link follows D-AI-IA-02.

| ID | Email | Sent when | Links | Stories |
|---|---|---|---|---|
| EML-AI-01 | Acknowledgement with KB suggestions | An email ticket is analyzed and at least one article is above the threshold | Each article (F06 page); *This solved my problem* → SCR-AI-05; *I still need help* → SCR-T-02 | US-AI-05 |
| EML-AI-02 | AI question | F05 sets `NEEDS_INFO` | *Reply to this email* (F03 threads it); *Open ticket* → SCR-T-02 | US-AI-03, US-AI-04 |
| EML-AI-03 | Agent KB reply | F05 accepts `KB_ANSWERABLE` | Cited steps with article links; *Confirm* / *Not solved* → SCR-AI-05 | US-AI-17 |
| EML-AI-04 | Result reply after automation | F05 verified a workflow run | *Confirm* / *Not solved* → SCR-AI-05; *Open ticket* → SCR-T-02 | US-AI-16 |
| EML-AI-05 | Requirement brief to confirm | A brief is drafted for a `BUSINESS_REQUIREMENT` | *Review and confirm* → SCR-AI-04 | US-AI-18 |
| EML-AI-06 | Technician reply | A technician resolves the ticket with the AI draft | *Confirm* / *Not solved* → SCR-AI-05; *Open ticket* → SCR-T-02 (sent in the technician's name) | US-AI-16 |

## 4. Routes and permissions

The API enforces every row; hiding an element is only for usability. "Scope" means tickets assigned to the technician or one of their groups (F01 US-03).

| Item | Requester | Technician | Admin | Unauthorized result |
|---|---|---|---|---|
| SEC-AI-01, SEC-AI-02 | ✓ (own tickets only for SEC-AI-02) | ✓ | ✓ | — |
| SEC-AI-03 | Own tickets | Scope | ✓ | `403` |
| SEC-AI-04 | Own tickets | Scope | ✓ | `403` |
| SEC-AI-05, SEC-AI-06, SEC-AI-08, SEC-AI-09, SEC-AI-10, SEC-AI-12 | ✗ not rendered, not in the API response | Scope | ✓ | Field absent from the response |
| SEC-AI-07 | Category only | Scope | ✓ | Fields absent from the response |
| SEC-AI-11 | Own tickets (review, confirm) | Scope (read, edit) | ✓ | `403` |
| SEC-AI-13 | ✗ | Scope | ✓ | `403` |
| SEC-AI-14 | ✗ (AI columns not shown in *My tickets*) | ✓ | ✓ | — |
| SEC-AI-15 | ✗ | ✗ | ✓ | `403` |
| SCR-AI-01 `/admin/ai/learning-examples` | ✗ | ✗ | ✓ | `#/403` page |
| SCR-AI-02, SCR-AI-03 | ✗ | Scope | ✓ | `403` |
| SCR-AI-04, SCR-AI-05 | Own tickets | Read only | Read only | `403` |

A requester who opens an email link to someone else's ticket gets `#/403`, and nothing changes.

## 5. Navigation

| Role | Sidebar | What changes because of F04 |
|---|---|---|
| Requester | My tickets, New ticket, Profile (F01, F03) | Nothing new in the sidebar. *New ticket* is a free-text form |
| Technician | Assigned tickets, Profile | Ticket list gets AI columns and filters (SEC-AI-14). Saved filter *Suspected bugs* |
| Admin | Dashboard, Tickets, Users, Groups, …, **AI** | One **AI** group: AI scope (SCR-WF-03, F05) · AI decisions (SCR-WF-30, F05) · Learning examples (SCR-AI-01, F04) · AI suggestions (F01) · Service accounts (F01) |

Contextual navigation inside SCR-T-02: the escalation banner links to the AI conversation and the summary; a citation opens the KB article (F06) in a new tab; a similar ticket opens its own SCR-T-02 (if in scope); the Admin sees *View analysis* → SCR-WF-31 tab SEC-AI-15.

## 6. Story-to-screen mapping

Every Must story is reachable by its role.

| Story | MoSCoW | Role | Screens / sections / emails |
|---|---|---|---|
| US-AI-01 | Must | Requester | SCR-T-01 (free text), SEC-AI-07 (category on SCR-T-02) |
| US-AI-02 | Must | Admin | SEC-AI-15, SEC-AI-07 |
| US-AI-03 | Must | Requester | SEC-AI-03, EML-AI-02 |
| US-AI-04 | Must | Requester | SEC-AI-03 reply box, EML-AI-02 reply |
| US-AI-05 | Must | Requester | SEC-AI-01, SEC-AI-04, EML-AI-01, SCR-AI-05 |
| US-AI-06 | Must | Technician | SEC-AI-09 |
| US-AI-07 | Must | Technician | SEC-AI-06, F05 assignment notification |
| US-AI-08 | Should | Technician | SEC-AI-06 |
| US-AI-09 | Should | Technician | SEC-AI-08 |
| US-AI-10 | Should | Technician | SEC-AI-08, SCR-AI-03 |
| US-AI-11 | Should | Technician | SEC-AI-10 |
| US-AI-12 | Could | Requester | SEC-AI-02 |
| US-AI-13 | Should | Technician | SEC-AI-07, SCR-AI-02 |
| US-AI-14 | Should | Technician, Requester | SEC-AI-04, SEC-AI-06, SEC-AI-09 |
| US-AI-15 | Must | Requester (benefit), Technician (sees) | SEC-AI-12, SEC-AI-07 |
| US-AI-16 | Must | Technician, Requester | SEC-AI-13, EML-AI-04, EML-AI-06, SCR-AI-05 |
| US-AI-17 | Must | Requester | SEC-AI-03, EML-AI-03, SCR-AI-05 |
| US-AI-18 | Should | Requester, Technician | SEC-AI-11, SCR-AI-04, EML-AI-05 |
| US-AI-19 | Should | Admin | SCR-AI-01 |
| US-AI-20 | Must | Requester, Technician | SCR-T-01, SEC-AI-05, SEC-AI-07, SEC-AI-14 |
| US-AI-21 | Should | Technician | SEC-AI-05, SEC-AI-10, SEC-AI-14 |

## 7. Requests to other branches

| Owner | Request |
|---|---|
| F03 | Free-text create form without a category picker; host the sections above on SCR-T-01, SCR-T-02, the ticket list and the Resolve dialog; `?action=` handling on `/tickets/:id` (D-AI-IA-02); decide what happens to tickets linked as duplicates when the main ticket is resolved (flow 4) |
| F05 | Tab SEC-AI-15 on SCR-WF-31; the `AUTO_RESOLVE_KB` setting on SCR-WF-03 (Q-AI-10) |
| F01 | The **AI** sidebar group for Admin; SSO return to the original link after login |
| F06 | Article pages that open at a section (`#§3`) for citations; the self-service flag |
