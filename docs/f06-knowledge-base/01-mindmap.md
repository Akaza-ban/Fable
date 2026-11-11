# 01. Mindmap — F06 Knowledge Base (KB)

> Design status: TARGET V1
>
> System name: **AI-Powered IT Ticket Routing and Automated Resolution System**
>
> Scope: **F06 Knowledge Base — KB**

---

## 1. System Mindmap (Mermaid Diagram)

```mermaid
mindmap
  root((F06: KNOWLEDGE BASE))
    (1) Authoring & Editing
      Rich Text & Markdown Editor
      Syntax-highlighted Code Blocks & Logs
      Screenshots & Media Attachments
      Metadata: Categories, Tags, Error Codes
      Access Scope: Public vs Internal Only
      Auto-save Draft & Revision Branching
    (2) Review & Publishing Workflow
      Review Queue (Draft -> Pending Review)
      Visual Diff View (Additions in Green, Deletions in Red)
      Reviewer Actions: Approve, Request Changes, Reject
      Version Control (v1.0, v1.1, v2.0...) & Changelogs
      Deprecation & Archival
    (3) Smart Search & Retrieval
      AI Semantic / RAG Search
      Instant Extracted Answer Snippets
      Multi-facet Filters: Category, OS, Popularity
      Self-service Deflection at Ticket Creation
      Technician Console Quick-insert Widget
    (4) AI Article Generation from Resolved Tickets
      Trigger on Ticket Resolved / Closed
      PII Sanitization & Redaction (Names, IPs, Tokens)
      Structured Extraction: Problem, Root Cause, Solution Steps
      1-Click Transfer to Authoring Editor
    (5) Feedback & Quality Loop
      Helpfulness Voting: Thumbs Up / Thumbs Down
      Detailed Reporting: Outdated, Broken Link, Incorrect Steps
      Automated Flagging (Flagged Queue)
      Author Notifications & Review Reminders
```

---

## 2. Functional Breakdown Catalog

| Ref | Capability | Description | Target User |
| :--- | :--- | :--- | :--- |
| **F06.1** | **Authoring & Editing** | Rich text and markdown authoring tool with heading formatting, code blocks, screenshots, category classification, error codes, and auto-saving every 20 seconds. | Technician, Admin |
| **F06.2** | **Review & Publishing** | Approval workflow with visual Diff View showing highlighted additions and deletions. Supports version increments (v1.0 -> v1.1) and rollback audit trails. | Admin |
| **F06.3** | **Smart Search & Retrieval** | Natural language semantic search (RAG) with intent detection, faceted filtering, and ticket deflection suggestions during requester ticket drafting. | Requester, Technician |
| **F06.4** | **AI Article Generation** | Automatic synthesis of closed tickets into structured KB articles with strict PII scrubbing (masking emails, credentials, private IPs). | AI Service → Technician |
| **F06.5** | **Feedback & Quality Loop** | Thumbs up/down rating widget, issue reporting modal, automated threshold flagging (`Helpful < 60%`), and revision assignment queues. | Requester, Technician, Admin |

---

## 3. Integration Touchpoints with Other Modules

*   **F03 Ticket Management:** Supplies resolved tickets as data sources for KB extraction; embeds KB search into the technician ticket workspace.
*   **F04 AI Ticket Processing:** Leverages verified KB articles as semantic grounding data to answer how-to inquiries and generate automated deflection responses.
*   **F05 Workflow & Automation:** Provides automated routing and notification hooks when articles are submitted for review or flagged by users.
