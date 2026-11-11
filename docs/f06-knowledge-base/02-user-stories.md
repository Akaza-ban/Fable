# 02. User Stories & Acceptance Criteria — F06 Knowledge Base

> Design status: TARGET V1
>
> Methodology: Agile / Scrum & INVEST Assessment
>
> Format: `As a [Role], I want to [Action], So that [Benefit]`

---

## 1. Personas & Actors

| Persona | Role Code | Responsibilities & Core Goals |
| :--- | :--- | :--- |
| **Requester** | `REQUESTER` | Business employee seeking immediate self-help solutions for common IT issues (VPN, printers, credentials) without opening a ticket. |
| **Technician** | `TECHNICIAN` | Support engineer standardizing troubleshooting steps, contributing new articles, and converting resolved tickets into KB drafts with AI. |
| **Administrator** | `ADMIN` | IT Lead governing content quality, validating technical accuracy, reviewing visual diffs, and auditing flagged articles. |
| **AI Knowledge Engine** | `AI` | Background intelligence service reading closed tickets, masking sensitive PII, and drafting structured knowledge drafts. |

---

## 2. Product Backlog & Story Estimates

| Story ID | Title | Role | Priority | Points |
| :--- | :--- | :--- | :---: | :---: |
| **US-KB-01** | Intelligent Semantic Solution Search | Requester, Technician | **Must** | 5 |
| **US-KB-02** | Rich Text & Markdown Article Authoring | Technician | **Must** | 5 |
| **US-KB-03** | Article Revisions & Version Branching | Technician, Admin | **Must** | 5 |
| **US-KB-04** | AI Article Synthesis from Resolved Tickets | Technician, AI | **Must** | 8 |
| **US-KB-05** | Approval Workflow & Visual Diff Review | Admin | **Must** | 5 |
| **US-KB-06** | Article Rating & Outdated Issue Reporting | Requester, Technician | **Should** | 3 |
| **US-KB-07** | Flagged Article Review & Deprecation Queue | Admin, Technician | **Should** | 5 |
| **US-KB-08** | Real-Time Self-Service Ticket Deflection | Requester | **Could** | 5 |

---

## 3. User Story Specifications & Acceptance Criteria (Gherkin BDD)

### US-KB-01: Intelligent Semantic Solution Search
> *As a* **Requester or Technician**,  
> *I want to* query the knowledge base using natural language inquiries or system error codes,  
> *So that* I can find validated troubleshooting steps immediately without waiting for support.

```gherkin
Feature: Knowledge Base Semantic Search

  Scenario: Natural language search with answer extraction
    Given The user is on the Knowledge Base portal search bar
    When The user types "cannot print documents over office wifi"
    And Clicks "Search"
    Then The system returns articles ranked by semantic similarity score
    And Article "KB-1015: Office Wi-Fi Network Printer Troubleshooting" is displayed at rank 1
    And An extracted highlight snippet displays the core fix steps directly

  Scenario: Audience scope filtering
    Given The logged-in user has the "REQUESTER" role
    When The user searches for "Cisco Core Router Configuration"
    Then The system excludes all articles flagged as "INTERNAL_ONLY"
    And Only returns public articles matching the query
```

---

### US-KB-02: Rich Text & Markdown Article Authoring
> *As a* **Technician**,  
> *I want to* author troubleshooting articles with rich formatting, code blocks, screenshots, and metadata,  
> *So that* complex technical resolutions are standardized and easily reproducible.

```gherkin
Feature: Article Authoring & Validation

  Scenario: Mandatory field enforcement on submission
    Given The technician is in the Article Editor workspace
    When The technician leaves "Title" or "Category" empty
    And Clicks "Submit for Review"
    Then The action is blocked with HTTP 400 validation error
    And The missing fields are highlighted in red with descriptive tooltips

  Scenario: Auto-save draft mechanism
    Given The technician is editing an article draft
    When No keystroke occurs for 20 continuous seconds
    Then The system saves the current draft state to backend storage
    And Displays a subtle toast: "Draft auto-saved at HH:mm:ss"
```

---

### US-KB-03: Article Revisions & Version Branching
> *As a* **Technician or Admin**,  
> *I want to* create an unpublishable draft revision for an active published article,  
> *So that* active readers are uninterrupted while updates are being reviewed.

```gherkin
Feature: Version Branching & Changelog

  Scenario: Create isolated draft from published article
    Given Article "KB-1042" is currently in "PUBLISHED" state at version v1.0
    When The technician clicks "Edit Article"
    Then The system forks a draft revision tagged "v1.1 (Draft)"
    And The public continues to view and search v1.0 without interruption
```

---

### US-KB-04: AI Article Synthesis from Resolved Tickets
> *As a* **Technician**,  
> *I want* the AI Service to analyze closed tickets and automatically draft a structured KB article,  
> *So that* resolution know-how is captured with zero manual transcription overhead.

```gherkin
Feature: AI Synthesis & PII Sanitization

  Scenario: Synthesizing a resolved ticket into a structured article
    Given Ticket "#TK-4921" has reached state "RESOLVED"
    When The technician clicks "Generate KB Draft with AI"
    Then The system returns a structured modal within 5.0 seconds containing:
      | Field | Expected Content |
      | Title | Standardized issue title |
      | Problem Description | User-reported symptoms |
      | Root Cause | Underlying technical failure |
      | Solution Steps | Ordered step-by-step resolution |

  Scenario: Automated PII redaction
    Given The source ticket contains customer email "nam.hoang@company.com" and IP "192.168.1.50"
    When The AI extracts the article content
    Then The email is redacted to "h***@company.com"
    And The private IP address is replaced with placeholder "<Client_IP>"
```

---

### US-KB-05: Approval Workflow & Visual Diff Review
> *As an* **Administrator**,  
> *I want to* inspect proposed revisions using side-by-side visual diffing,  
> *So that* only accurate, secure, and policy-compliant instructions reach production.

```gherkin
Feature: Visual Diff & Review Actions

  Scenario: Side-by-side Diff View inspection
    Given An article revision for "KB-1042" is in "PENDING_REVIEW"
    When The admin opens Diff View mode
    Then Added words are highlighted in green
    And Removed legacy text is struck through in red

  Scenario: Approval publishing
    Given The admin verifies the technical steps are sound
    When The admin clicks "Approve & Publish"
    Then The article state transitions to "PUBLISHED"
    And Search vectors are regenerated for real-time indexing
    And An in-app notification is dispatched to the author
```

---

### US-KB-06: Article Rating & Outdated Issue Reporting
> *As a* **Reader (Requester or Technician)**,  
> *I want to* vote on article usefulness and flag broken or outdated instructions,  
> *So that* the support team is alerted to obsolete procedures.

```gherkin
Feature: Article Feedback & Quality Assurance

  Scenario: Submitting an issue report
    Given A user encounters a broken link or obsolete command in "KB-1042"
    When The user clicks "Not Helpful / Report Issue"
    And Selects reason "Outdated instructions" with description text
    And Submits the modal
    Then A feedback record is created with status "PENDING"
    And If negative reports reach 3 within 30 days, article status becomes "FLAGGED"
```

---

### US-KB-07: Flagged Article Review & Deprecation Queue
> *As an* **Administrator or Technician**,  
> *I want to* review articles flagged as outdated, inaccurate, or low quality,  
> *So that* obsolete instructions are corrected or archived before they mislead users.

```gherkin
Feature: Flagged Article Maintenance

  Scenario: Flagged article enters maintenance queue
    Given Article "KB-1042" has received 3 negative reports within 30 days
    When The system evaluates article quality scores
    Then The article state transitions to "FLAGGED"
    And The article appears in the maintenance queue for assigned technicians and administrators

  Scenario: Deprecating an obsolete article
    Given An article is no longer valid because the platform or process has changed
    When The admin clicks "Archive Article"
    Then The article becomes read-only and visible only to authorized internal roles
    And A replacement or redirect notice is displayed to readers when relevant
```

---

### US-KB-08: Real-Time Self-Service Ticket Deflection
> *As a* **Requester**,  
> *I want to* be shown relevant KB results while creating a support ticket,  
> *So that* I can resolve common issues without waiting for a human response.

```gherkin
Feature: Real-Time KB Deflection During Ticket Creation

  Scenario: Relevant solution is suggested while typing a new ticket
    Given The user is creating a ticket with title "Cannot connect to VPN after password reset"
    When The user types the description and the system runs semantic matching
    Then The top 3 relevant KB articles are displayed above the submit button
    And The user can click a result to view the resolution without leaving the form

  Scenario: Ticket creation is avoided when a known fix is found
    Given A high-confidence KB match exists for the user's issue
    When The user clicks "This solved my issue"
    Then The ticket draft is closed or marked as resolved by self-service guidance
    And The deflection event is logged for analytics
```
