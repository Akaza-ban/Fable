# Screens, Routes, and Navigation — F06 Knowledge Base

> Design status: TARGET V1
>
> Related: [IA overview](README.md) · [User flows](02-user-flows.md) · [UI states](03-ui-states-and-accessibility.md)

---

## 1. Dedicated F06 Screens and Modals

| ID | Screen / Modal Name | Type | Route / Trigger | Target Actor | Core Content & Capabilities | Mapped Stories |
|---|---|---|---|---|---|---|
| **SCR-KB-01** | Solution Search Portal | Page | `/kb` | All Roles | Natural language semantic search bar, category taxonomy pills, popular solutions, recent searches. Public view filters out internal-only articles. | US-KB-01 |
| **SCR-KB-02** | Article Detail View | Page | `/kb/articles/:id` | All Roles | Breadcrumbs, metadata header, problem description, root cause, ordered step-by-step fix, syntax-highlighted code, image attachments, helpful voting widget. | US-KB-01, US-KB-06 |
| **SCR-KB-03** | Article Authoring Studio | Page | `/kb/editor` or `/kb/editor/:id` | Technician, Admin | Rich text & markdown editor, metadata sidebar (Category, Tags, Audience scope), attachment dropzone, auto-save status indicator, changelog summary input. | US-KB-02, US-KB-03 |
| **SCR-KB-04** | AI Ticket-to-KB Synthesis | Modal | Triggered on resolved ticket | Technician | AI-extracted fields (Title, Problem, Cause, Solution steps), PII-redacted badges, confidence score, button to transfer into `SCR-KB-03`. | US-KB-04 |
| **SCR-KB-05** | Approval Queue & Diff Review | Page | `/kb/approvals` | Admin | Pending review data table, side-by-side Visual Diff View modal (additions in green, deletions in red), Approve & Publish button, Request Changes form. | US-KB-05 |
| **SCR-KB-06** | Flagged Content Maintenance | Page | `/kb/flagged` | Admin, Technician | Data table of articles receiving negative feedback (`Helpful < 60%`), user report comments, assign revision task, deprecate/archive action. | US-KB-07 |
| **SCR-KB-07** | Report Inaccurate Article | Modal | Triggered from `SCR-KB-02` | Requester, Technician | Feedback categorization (Outdated, Incorrect command, Missing steps, Broken link), detail explanation textarea, submit button. | US-KB-06 |

---

## 2. Host Pages and Embedded F06 Sections

F06 owns KB article search, visibility filtering, and article content. When these capabilities are embedded in another module, the host module continues to own its page and workflow: F03 owns ticket creation and ticket state changes; F04 owns AI-generated answers and AI resolution decisions.

F06 embeds specialized intelligence and self-service components into host pages owned by **F03 Ticket Management** and **F04 AI Ticket Processing**:

| Host Page | Owner | Route | F06 Embedded Section | Purpose & Content | Stories |
|---|---|---|---|---|---|
| `SCR-T-01` Create Ticket | F03 | `/tickets/new` | **SEC-KB-01** (KB suggestions) | F03 sends the in-progress title and description; F06 returns top matching articles filtered by audience. F04 may use these articles for a cited answer or self-service decision. F03 owns the form and ticket submission/cancellation. | US-KB-08 |
| `SCR-T-02` Ticket Detail | F03 | `/tickets/:id` | **SEC-KB-02** (KB references) | F06 provides authorized published article references. F04 owns any AI-generated answer; F03 owns inserting a selected reference into a reply draft and sending the reply. | US-KB-01 |
| Resolve Dialog | F03 | modal on `/tickets/:id` | **SEC-KB-03** (KB draft trigger) | F03 supplies the resolved ticket context and may offer a trigger for `SCR-KB-04`. F06 creates an article draft; F03 retains ownership of ticket status and resolution. | US-KB-04 |

---

## 3. Routes and Permissions Matrix

Access enforcement is strictly applied at the API gateway layer (`403 Forbidden`) and matched in UI rendering:

| Item / Route | Requester | Technician | Admin | Unauthorized Result |
|---|:---:|:---:|:---:|---|
| **SCR-KB-01** (`/kb`) | ✓ (Public articles) | ✓ (Public + Internal) | ✓ (All) | Redirect to `/login` |
| **SCR-KB-02** (`/kb/articles/:id`) | ✓ (Public only) | ✓ | ✓ | `404 Not Found` if private or unlisted |
| **SCR-KB-03** (`/kb/editor`) | ✗ | ✓ | ✓ | `403 Forbidden` page |
| **SCR-KB-04** (AI Synthesis modal) | ✗ | ✓ (Assigned tickets) | ✓ | `403 Forbidden` |
| **SCR-KB-05** (`/kb/approvals`) | ✗ | ✗ (Read-only own drafts) | ✓ | `403 Forbidden` page |
| **SCR-KB-06** (`/kb/flagged`) | ✗ | ✓ (Assigned maintenance) | ✓ | `403 Forbidden` page |
| **SCR-KB-07** (Report Modal) | ✓ | ✓ | ✓ | Prompt login |
| **SEC-KB-01** (Deflection on create) | ✓ | ✓ | ✓ | Hidden |
| **SEC-KB-02** (Sidebar steps) | ✗ | ✓ | ✓ | Not rendered |
| **SEC-KB-03** (Synthesis prompt) | ✗ | ✓ | ✓ | Not rendered |

---

## 4. Navigation by Role

```text
GLOBAL SIDEBAR NAVIGATION MATRIX
├── [REQUESTER]
│   └── Knowledge Base                  -> /kb (SCR-KB-01)
│
├── [TECHNICIAN]
│   ├── Knowledge Base                  -> /kb (SCR-KB-01, with Internal filter)
│   ├── Author New Guide                -> /kb/editor (SCR-KB-03)
│   ├── My Drafts & Revisions           -> /kb/editor?tab=drafts
│   └── Flagged Maintenance Queue       -> /kb/flagged (SCR-KB-06)
│
└── [ADMINISTRATOR]
    ├── Knowledge Base                  -> /kb (SCR-KB-01)
    ├── Author New Guide                -> /kb/editor (SCR-KB-03)
    ├── Approval Queue (Diff Review)    -> /kb/approvals (SCR-KB-05)
    ├── Flagged Content Queue           -> /kb/flagged (SCR-KB-06)
    └── Taxonomy & Category Config      -> /admin/kb/categories
```

---

## 5. User Story to Screen Mapping

| Story ID | User Story Title | Primary Screen | Secondary Screens / Sections |
|---|---|---|---|
| **US-KB-01** | Intelligent Semantic Solution Search | `SCR-KB-01` (Portal) | `SCR-KB-02` (Detail), `SEC-KB-02` |
| **US-KB-02** | Rich Text & Markdown Article Authoring | `SCR-KB-03` (Editor) | - |
| **US-KB-03** | Article Revisions & Version Branching | `SCR-KB-03` (Editor) | `SCR-KB-05` (Diff Review) |
| **US-KB-04** | AI Article Synthesis from Resolved Tickets | `SCR-KB-04` (Synthesis Modal) | `SEC-KB-03`, `SCR-KB-03` |
| **US-KB-05** | Approval Workflow & Visual Diff Review | `SCR-KB-05` (Approvals) | `SCR-KB-02` |
| **US-KB-06** | Article Rating & Outdated Issue Reporting | `SCR-KB-02` (Detail) | `SCR-KB-07` (Report Modal) |
| **US-KB-07** | Flagged Article Review & Deprecation Queue | `SCR-KB-06` (Flagged Queue) | `SCR-KB-03` |
| **US-KB-08** | Real-Time Self-Service Ticket Deflection | `SEC-KB-01` (Host on F03) | `SCR-KB-02` |

---

## 6. Cross-Module Touchpoints & Requests

| Branch | Touchpoint / Dependency | Contract Specification |
|---|---|---|
| **F03 Ticket Management** | Host `SEC-KB-01` on `/tickets/new` | Receives live draft title and description; returns deflected article click callback to discard draft. |
| **F03 Ticket Management** | Host `SEC-KB-02` on `/tickets/:id` | Renders verified KB resolution steps; provides button to copy Markdown into ticket response. |
| **F03 Ticket Management** | Host `SEC-KB-03` on Resolve dialog | Triggers `SCR-KB-04` with `ticket_id`, conversation messages, and resolution notes. |
| **F04 AI Processing** | KB retrieval for AI answers | F04 consumes authorized, published article results from F06 and owns answer generation, citations, and any AI self-resolution decision. F04 does not create or publish KB articles. |
| **F05 Workflow & Automation** | Article lifecycle events | F06 emits article submission/flagging events when integration is required; F05 owns workflow rules, routing, assignment, and SLA behavior. |
| **F07 Reports & Analytics** | KB usage and quality events | F06 provides article views, feedback, and deflection-related events; F07 owns aggregation and report presentation. |
| **F08 Activity Log** | KB audit events | F06 emits article lifecycle and moderation events; F08 owns audit history and browsing. |
