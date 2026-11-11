# User Flows — F06 Knowledge Base

> Design status: TARGET V1
>
> Related: [IA overview](README.md) · [Screens and navigation](01-screens-and-navigation.md) · [UI states](03-ui-states-and-accessibility.md)

---

## FLOW-KB-01: Requester Self-Service Search & Real-Time Deflection

This flow illustrates how requesters resolve technical issues independently, deflecting tickets before submission.

```mermaid
sequenceDiagram
    autonumber
    actor User as Requester
    participant Form as SCR-T-01 (Create Ticket)
    participant SEC as SEC-KB-01 (Deflection)
    participant KB as SCR-KB-02 (Article Detail)
    participant AI as AI Semantic Vector Service

    User->>Form: Enters issue description (> 20 chars)
    Form->>AI: Query embeddings (debounce 300ms)
    AI-->>SEC: Return top 3 matching articles (score >= 0.70)
    SEC->>User: Render instant solution suggestion cards
    alt Solution works (Deflection Success)
        User->>SEC: Clicks suggested article link
        SEC->>KB: Opens article in drawer / new tab
        User->>KB: Executes step-by-step fix
        User->>KB: Clicks "This solved my issue"
        KB-->>Form: Notify F03 of successful self-service outcome
        Form->>Form: F03 discards the unsubmitted draft; no ticket is created
        Form-->>Form: Record deflection outcome for F07 analytics
    else Solution does not apply
        User->>Form: Continues typing & clicks "Submit Ticket anyway"
        Form-->>Form: Create ticket in F03
    end
```

---

## FLOW-KB-02: Technician Manual Article Authoring & Review Submission

```mermaid
sequenceDiagram
    autonumber
    actor Tech as Technician
    participant Editor as SCR-KB-03 (Authoring Studio)
    participant Storage as KB Persistence
    actor Admin as Admin
    participant Queue as SCR-KB-05 (Approval Queue)

    Tech->>Editor: Opens /kb/editor
    Tech->>Editor: Enters Title, Category, Audience (Public / Internal)
    Tech->>Editor: Writes steps in Markdown & attaches screenshots
    loop Auto-Save (Every 20s)
        Editor->>Storage: Persist draft payload (state: DRAFT)
        Storage-->>Editor: Toast: "Draft auto-saved at HH:mm:ss"
    end
    Tech->>Editor: Enters changelog note & clicks "Submit for Review"
    Editor->>Storage: Transition state -> PENDING_REVIEW
    Storage->>Queue: Push article revision into Review Queue
    Queue->>Admin: Send in-app notification to reviewers
```

---

## FLOW-KB-03: AI 1-Click Synthesis from Resolved Ticket to KB Draft

```mermaid
sequenceDiagram
    autonumber
    actor Tech as Technician
    participant Ticket as SCR-T-02 (Ticket Detail)
    participant Modal as SCR-KB-04 (AI Synthesis Modal)
    participant AI as AI Knowledge Extractor
    participant Editor as SCR-KB-03 (Authoring Studio)

    Tech->>Ticket: Marks ticket state as RESOLVED
    Ticket->>Modal: Prompts "Generate KB Draft with AI"
    Tech->>Modal: Clicks "Generate Draft"
    Modal->>AI: Send conversation history & resolution notes
    AI->>AI: Scrub PII (mask emails, IPs, credentials)
    AI->>AI: Structure into: Problem, Cause, Steps
    AI-->>Modal: Return synthesized draft with confidence score (e.g. 94%)
    Modal->>Tech: Render preview with PII-redacted badges
    Tech->>Modal: Clicks "Transfer to KB Editor"
    Modal->>Editor: Pre-fills authoring form with AI content
    Tech->>Editor: Refines tone & clicks "Submit for Review"
```

---

## FLOW-KB-04: Admin Visual Diff Inspection & Publication Workflow

```mermaid
sequenceDiagram
    autonumber
    actor Admin as Admin
    participant Queue as SCR-KB-05 (Approval Queue)
    participant Diff as Visual Diff Modal
    participant Vector as Vector Search Index
    actor Author as Technician (Author)

    Admin->>Queue: Opens /kb/approvals
    Admin->>Queue: Clicks "Inspect Diff" on KB-1042 (v1.2)
    Queue->>Diff: Render side-by-side comparison (v1.1 vs v1.2)
    Diff->>Admin: Highlight green additions & red strikethrough deletions
    alt Approved
        Admin->>Diff: Clicks "Approve & Publish"
        Diff->>Vector: Update search embeddings index
        Diff-->>Author: Notify: "Your article KB-1042 v1.2 has been published"
    else Revisions Required
        Admin->>Diff: Enters feedback note & clicks "Request Changes"
        Diff-->>Author: Return article to author (state: CHANGES_REQUESTED)
    end
```

---

## FLOW-KB-05: Reader Helpfulness Rating & Outdated Issue Escalation

```mermaid
sequenceDiagram
    autonumber
    actor User as Reader (User / Tech)
    participant Detail as SCR-KB-02 (Article Detail)
    participant Modal as SCR-KB-07 (Report Issue Modal)
    participant Queue as SCR-KB-06 (Flagged Maintenance)
    actor Admin as Administrator

    User->>Detail: Reads article instructions
    alt Found Helpful
        User->>Detail: Clicks "Helpful"
        Detail-->>Detail: Increment helpful_up_count & show thank-you toast
    else Broken or Outdated
        User->>Detail: Clicks "Not Helpful / Report Issue"
        Detail->>Modal: Open feedback modal
        User->>Modal: Selects reason ("Outdated steps") & types description
        User->>Modal: Clicks "Submit Report"
        Modal-->>Detail: Increment helpful_down_count
        alt Negative threshold reached (>= 3 reports)
            Detail->>Queue: Transition state -> FLAGGED
            Queue->>Admin: Alert: "KB-1042 flagged for maintenance"
            Admin->>Queue: Assign revision task to original author
        end
    end
```
