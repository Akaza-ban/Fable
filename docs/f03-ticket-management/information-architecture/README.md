# Information Architecture — F03 Ticket Management

> Design status: TARGET V1 — screen, route, table and operationId names are **proposed** and will be finalized with the OpenAPI contract and ERD.
>
> Implementation status: NOT IMPLEMENTED

This folder places the 22 F03 stories on screens. Unlike F04, **F03 owns the ticket pages** that every role works in: the create form, the requester's list, the staff work queues and the ticket detail. Other branches add their parts to these pages (F04 AI sections `SEC-AI-xx`, the F05 *Automation* and *SLA* tabs, the F08 *History* tab). The inventory has two kinds of entries:

| Kind | ID | Meaning |
|---|---|---|
| Page | `SCR-T-xx` | A page with its own route, owned by F03 |
| Dialog | `DLG-T-xx` | A short form opened on a page, with no route of its own (D-TK-IA-03) |

Inputs: [Use cases](../02-use-cases.md), [User stories and acceptance criteria](../01-user-stories.md). Shared screens (`SCR-01` Login, `SCR-90` Access denied, `SCR-91` Notifications) and the triage queue `SCR-WF-10` come from the [F05 information architecture](../../f05-workflow-automation/information-architecture/README.md).

Four roles only (D-TK-IA-07): **Requester** (`REQUESTER`), **Technician** (`TECHNICIAN`), **Admin** (`ADMIN`) and the **AI Service** (`AI`). The AI Service has no screen; its messages appear in the conversation signed *"AI Assistant"* (BR-TK-18).

| File | Content |
|---|---|
| [01-screens-and-navigation.md](01-screens-and-navigation.md) | Sitemap and routes, screen and dialog inventory, page permissions, navigation per role, notifications, story-to-screen mapping, requests to other branches |
| [02-screen-content.md](02-screen-content.md) | Zones of each page, dialog fields, action availability by status and role, low-fidelity wireframes |
| [03-user-flows.md](03-user-flows.md) | Six flows: create with evidence, technician handles and resolves, ask the requester, transfer and escalate, not fixed and reopen, cancel |
| [04-ui-states-and-accessibility.md](04-ui-states-and-accessibility.md) | UI state contract, plain vs internal status labels, SLA badges, responsive layout, accessibility |
| [05-data-and-api.md](05-data-and-api.md) | Main tables, ERD, indexes, API endpoints, published events |

## IA decisions

| ID | Decision | Why |
|---|---|---|
| D-TK-IA-01 | **One ticket detail page (SCR-T-02) for every role**, with role-dependent zones and tabs. The server returns only what the role may see (BR-TK-10). | Same as F05 D-IA-04; avoids near-identical pages and keeps internal data out of requester responses. |
| D-TK-IA-02 | **Two list pages:** SCR-T-03 *My tickets* (cards, plain-language statuses, "needs my action") for everyone's own requests, and SCR-T-04 *Work queues* (table, filters, bulk actions) for staff. | A requester and a technician ask different questions of a list: "where is my request?" vs "what do I work on next?". |
| D-TK-IA-03 | **Every ticket action with input is a dialog** (DLG-T-01 … 09) opened on SCR-T-02 or the work queue, with no route. | Keeps the conversation and details visible while acting (same as F05 D-IA-03). |
| D-TK-IA-04 | **The action bar shows only actions allowed for the current status and role** ([action matrix](02-screen-content.md#3-action-availability-status--role)); forbidden actions are hidden, not disabled. The API still rejects them (BR-TK-04). | Fewer wrong clicks at 8,000 tickets/day; one source of truth in the transition table. |
| D-TK-IA-05 | **Views, filters, sort and page live in the URL** (`?view=`, `?status=`, …), so Back restores them and a view can be shared. | Staff move between list and detail all day. |
| D-TK-IA-06 | **F03 hosts tabs owned by other branches:** *History* (F08), *SLA* and *Automation* (F05), *AI* and the AI sections (F04). F03 defines where they sit; their owners define the content. | One ticket page; each branch changes only its part. |
| D-TK-IA-07 | **No Team Lead / Operations Lead role.** The system has four roles: Requester, Technician, Admin, AI Service. Triage, assign / reassign, escalation target and priority approval belong to the **Admin**; technicians claim, transfer and escalate. | Team decision: the system has only four roles. |
| D-TK-IA-08 | **Requesters see plain-language statuses and an outcome summary**, never internal codes, priority sources or individual technician names unless the technician replied (BR-TK-18). | Requesters need to know what is expected from them, not the internal workflow. |

## Information architecture

Solid lines: a page and the pages or dialogs opened from it. Dashed arrows: which areas each actor uses. Blue: F03-owned pages and dialogs. White: pages and tabs owned by other branches.

```mermaid
flowchart TB
    req["<b>Requester</b>"]
    tech["<b>Technician</b>"]
    admin["<b>Admin</b>"]

    subgraph mine["My requests — all roles"]
        direction TB
        t03["<b>SCR-T-03 My tickets</b><br/>• Tabs: Open, Needs my action,<br/>Resolved and closed, Cancelled<br/>• Plain-language status, last activity"]
        t01["<b>SCR-T-01 Create ticket</b><br/>• Title, description, urgency<br/>• Evidence: drag, browse, paste<br/>• On behalf of a colleague<br/>• Similar tickets and KB (Could)"]
    end

    subgraph work["Work queues — Technician, Admin"]
        direction TB
        t04["<b>SCR-T-04 Work queues</b><br/>• Assigned to me, Team unassigned,<br/>Waiting, Due soon, Breached<br/>• Escalated to me, Triage (Admin)<br/>• Search, filters, saved views<br/>• Bulk claim / assign"]
    end

    subgraph detail["Ticket detail — every role, scope by role"]
        direction TB
        t02["<b>SCR-T-02 Ticket detail</b><br/>• Header: code, status, SLA badge<br/>• Action bar by status and role<br/>• Conversation: public reply / internal note<br/>• Details panel, Attachments tab"]
        dlgStaff["<b>Staff dialogs</b><br/>DLG-T-01 Assign · 02 Transfer<br/>03 Escalate · 04 Ask requester<br/>05 Resolve · 09 Category / priority"]
        dlgReq["<b>Requester dialogs</b><br/>DLG-T-06 Cancel · 07 Not fixed / reopen<br/>08 Confirm and rate"]
    end

    subgraph others["Owned by other branches"]
        direction TB
        hist["<b>History tab</b> (F08)"]
        wf["<b>SLA, Automation tabs</b> (F05)<br/><b>AI tab, SEC-AI sections</b> (F04)"]
        triage["<b>SCR-WF-10 Triage queue</b> (F05)"]
        notif["<b>SCR-91 Notifications</b> (F05)<br/>+ e-mail"]
    end

    t03 --- t01
    t03 --- t02
    t01 --- t02
    t04 --- t02
    t02 --- dlgStaff & dlgReq
    t02 --- hist & wf
    triage --- t02
    notif --- t02

    req -.-> mine
    req -.-> detail
    tech -.-> mine
    tech -.-> work
    tech -.-> detail
    admin -.-> mine
    admin -.-> work
    admin -.-> detail
    admin -.-> triage

    classDef actor fill:none,stroke:none,color:#111
    classDef f03 fill:#e8f1fc,color:#111,stroke:#1168bd,text-align:left
    classDef ext fill:#fff,color:#111,stroke:#999,stroke-dasharray:4 3,text-align:left
    class req,tech,admin actor
    class t01,t02,t03,t04,dlgStaff,dlgReq f03
    class hist,wf,triage,notif ext
```

## Screen hierarchy per role

| Role | Home page | Pages | Dialogs |
|---|---|---|---|
| Requester | SCR-T-03 *My tickets* | SCR-T-01, SCR-T-02 (own and beneficiary tickets), SCR-T-03 | DLG-T-06, 07, 08 (own tickets; beneficiary can only reply, BR-TK-22) |
| Technician | SCR-T-04 *Assigned to me* | SCR-T-01, SCR-T-02 (assigned and team tickets), SCR-T-03, SCR-T-04 | DLG-T-02, 03, 04, 05, 06, 09 (as assignee) |
| Admin | Dashboard (F02) | Every page, every ticket, including views *Triage* (= SCR-WF-10), *Escalated to me*, *All open tickets* | Every dialog, including DLG-T-01 assign / reassign; DLG-T-07, 08 on behalf of the requester (reason required) |
| AI Service | — | No UI; writes messages through the API (F04 / F05) | — |

**Related:** [F03 README](../README.md) · [Screens and navigation](01-screens-and-navigation.md) · [Screen content](02-screen-content.md) · [User flows](03-user-flows.md) · [UI states](04-ui-states-and-accessibility.md) · [Data and API](05-data-and-api.md)
