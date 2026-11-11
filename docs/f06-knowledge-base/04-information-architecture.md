# 04. Information Architecture (IA) & Data Schema — F06 Knowledge Base

> Design status: TARGET V1
>
> System name: **AI-Powered IT Ticket Routing and Automated Resolution System**
>
> Scope: **F06 Knowledge Base — KB**
>
> Sub-specifications: [IA Overview](information-architecture/README.md) · [Screens & Navigation](information-architecture/01-screens-and-navigation.md) · [User Flows](information-architecture/02-user-flows.md) · [UI States & Accessibility](information-architecture/03-ui-states-and-accessibility.md) · [ERD & Data Dictionary](information-architecture/04-data-dictionary-and-erd.md)

---

## 1. Information Architecture & Navigation Map

```text
KNOWLEDGE BASE PORTAL & MANAGEMENT CONSOLE
│
├── 1.0 PUBLIC HELP CENTER (REQUESTER & TECHNICIAN)
│   ├── 1.1 Natural Language Search Bar (AI Semantic + Auto-suggest)
│   ├── 1.2 Category Taxonomy Hierarchy
│   │   ├── Hardware & Peripherals
│   │   ├── Office Applications & Productivity Software
│   │   ├── Corporate Network, Wi-Fi & VPN
│   │   └── Identity, Accounts & Access Permissions
│   ├── 1.3 Featured & Pinned Resolutions
│   └── 1.4 Ticket Deflection Widget ("Did this resolve your issue?")
│
├── 2.0 ARTICLE DETAIL VIEW
│   ├── 2.1 Breadcrumb Hierarchy
│   ├── 2.2 Metadata Header (Code, Version, Updated Date, Author, Scope Badge)
│   ├── 2.3 Structured Content Blocks:
│   │   ├── 2.3.1 Problem Description & Symptoms
│   │   ├── 2.3.2 Root Cause Analysis
│   │   ├── 2.3.3 Step-by-Step Resolution Procedures
│   │   └── 2.3.4 Script & Driver Downloads
│   ├── 2.4 Related Articles & Tag Taxonomy
│   └── 2.5 Reader Feedback Widget (👍 Helpful / 👎 Report Issue Modal)
│
├── 3.0 TECHNICIAN AUTHORING WORKSPACE
│   ├── 3.1 Personal Drafts & Submission Queue
│   ├── 3.2 AI Inbox: Resolved Tickets Ready for KB Conversion
│   └── 3.3 Article Editor & Auto-save Engine
│
└── 4.0 ADMINISTRATIVE GOVERNANCE CONSOLE
    ├── 4.1 Pending Review Queue with Visual Diff Inspection
    ├── 4.2 Flagged Content & Outdated Issue Queue
    ├── 4.3 Category & Tag Taxonomy Management
    └── 4.4 Operational Health Metrics (Deflection Rate, Helpful Ratio)
```

---

## 2. Article State Transition Machine

```mermaid
stateDiagram-v2
    [*] --> Draft : Authoring initiated / AI synthesis
    Draft --> Draft : Auto-save revision
    Draft --> Pending_Review : Submitted for review

    Pending_Review --> Published : Approved by Admin
    Pending_Review --> Changes_Requested : Revisions required
    Pending_Review --> Rejected : Rejected by Admin

    Changes_Requested --> Draft : Author amends draft
    Rejected --> Draft : Restructured by author

    Published --> Under_Revision : Forked for active revision
    Under_Revision --> Pending_Review : Resubmitted for review

    Published --> Flagged : Helpful rate < 60% OR >= 3 reports
    Flagged --> Under_Revision : Assigned for maintenance
    Flagged --> Archived : Obsolete procedure deprecated

    Published --> Archived : Deprecated by lifecycle policy
    Archived --> [*]

    note right of Archived
        Archived articles remain read-only
        and keep audit history for traceability
    end note
```

---

## 3. Entity Relationship Diagram (ERD)

```mermaid
erDiagram
    KB_CATEGORIES ||--o{ KB_ARTICLES : contains
    KB_ARTICLES ||--|{ KB_ARTICLE_VERSIONS : tracks
    KB_ARTICLES ||--o{ KB_FEEDBACKS : receives
    KB_ARTICLES }|--|{ KB_TAGS : labeled_with
    TICKETS ||--o| KB_AI_EXTRACTIONS : sources
    KB_AI_EXTRACTIONS ||--o| KB_ARTICLES : seeds

    KB_CATEGORIES {
        uuid category_id PK
        string name "Category name"
        string slug "URL slug"
        uuid parent_id FK "Parent category"
        int display_order
        boolean is_active
    }

    KB_ARTICLES {
        uuid article_id PK
        string kb_code "Display code (e.g. KB-1042)"
        string title "Article Title"
        string slug "Unique URL slug"
        uuid category_id FK
        string target_audience "PUBLIC or INTERNAL_ONLY"
        string status "DRAFT, PENDING_REVIEW, CHANGES_REQUESTED, PUBLISHED, FLAGGED, UNDER_REVISION, ARCHIVED, REJECTED"
        string current_version "Active version (e.g. 1.2)"
        uuid author_id FK
        uuid approver_id FK
        int view_count
        int helpful_up_count
        int helpful_down_count
        uuid source_ticket_id FK "Originating ticket"
        timestamp published_at
        timestamp created_at
        timestamp updated_at
    }

    KB_ARTICLE_VERSIONS {
        uuid version_id PK
        uuid article_id FK
        string version_number "e.g. 1.0, 1.1"
        string title
        text problem_description
        text root_cause
        jsonb solution_steps
        text full_content_markdown
        text change_summary
        uuid created_by FK
        timestamp created_at
    }

    KB_TAGS {
        uuid tag_id PK
        string tag_name
        string tag_type "KEYWORD or ERROR_CODE"
    }

    KB_FEEDBACKS {
        uuid feedback_id PK
        uuid article_id FK
        uuid user_id FK
        string feedback_type "HELPFUL, UNHELPFUL, REPORT_ERROR"
        string report_reason "OUTDATED, INCORRECT, MISSING_INFO, BROKEN_LINK"
        text comment
        string status "PENDING, RESOLVED, DISMISSED"
        timestamp created_at
    }

    KB_AI_EXTRACTIONS {
        uuid extraction_id PK
        uuid ticket_id FK
        text raw_ticket_summary
        text extracted_problem
        text extracted_cause
        jsonb extracted_steps
        string suggested_title
        uuid suggested_category_id FK
        boolean pii_cleaned
        timestamp generated_at
    }
```

---

## 4. Data Dictionary

### Table: `kb_articles`
| Column | Type | Constraints | Description |
| :--- | :--- | :--- | :--- |
| `article_id` | UUID | PK, NOT NULL | Unique internal identifier |
| `kb_code` | VARCHAR(20) | UNIQUE, NOT NULL | User-facing code (e.g. KB-1042) |
| `title` | VARCHAR(255) | NOT NULL | Standard article title |
| `category_id` | UUID | FK, NOT NULL | Reference to `kb_categories` |
| `target_audience` | ENUM | NOT NULL, DEFAULT 'PUBLIC' | `PUBLIC` or `INTERNAL_ONLY` |
| `status` | ENUM | NOT NULL, DEFAULT 'DRAFT' | `DRAFT`, `PENDING_REVIEW`, `CHANGES_REQUESTED`, `PUBLISHED`, `FLAGGED`, `UNDER_REVISION`, `REJECTED`, `ARCHIVED` |
| `current_version`| VARCHAR(10) | NOT NULL, DEFAULT '1.0' | Currently published version string |
| `author_id` | UUID | FK, NOT NULL | ID of technician author |
| `approver_id` | UUID | FK, NULLABLE | ID of approving administrator |
| `source_ticket_id`| UUID | FK, NULLABLE | Linked originating ticket if synthesized by AI |
| `view_count` | INT | DEFAULT 0 | Cumulative reader views |
| `helpful_up_count`| INT | DEFAULT 0 | Total positive helpful votes |
| `helpful_down_count`| INT | DEFAULT 0 | Total negative / problem reports |
| `published_at` | TIMESTAMP | NULLABLE | Timestamp of publication |

---

### Table: `kb_feedbacks`
| Column | Type | Constraints | Description |
| :--- | :--- | :--- | :--- |
| `feedback_id` | UUID | PK, NOT NULL | Unique feedback entry ID |
| `article_id` | UUID | FK, NOT NULL | Foreign key to `kb_articles` |
| `user_id` | UUID | FK, NULLABLE | User who submitted feedback |
| `feedback_type` | ENUM | NOT NULL | `HELPFUL`, `UNHELPFUL`, `REPORT_ERROR` |
| `report_reason` | ENUM | NULLABLE | `OUTDATED`, `INCORRECT_STEPS`, `MISSING_CONTEXT`, `BROKEN_LINK` |
| `comment` | TEXT | NULLABLE | Detailed reader report |
| `status` | ENUM | DEFAULT 'PENDING' | `PENDING`, `RESOLVED`, `DISMISSED` |
| `created_at` | TIMESTAMP | DEFAULT NOW() | Timestamp of submission |

---

## 5. Data Flow — Resolved Ticket to Published Article

How a resolved ticket becomes a published article that requesters find on their own. Ported from the earlier Vietnamese draft on `feature/kho-kien-thuc-kb`.

```mermaid
flowchart TB
    resolved["Ticket RESOLVED"] -->|"Technician clicks Generate KB Draft"| extract["AI Service<br/>extracts content, scrubs PII"]
    extract -->|"Structured draft"| draft["Draft in the KB editor"]
    draft -->|"Technician reviews and submits"| review["Pending review"]
    review -->|"Admin checks the diff view and approves"| published["Published article"]
    published -->|"Embeddings"| vector["Vector index<br/>(semantic search)"]
    published -->|"Users read and search"| selfservice["Requester resolves the issue alone"]
    vector -->|"AI suggests articles while a ticket is created"| deflection["Ticket deflection"]
    deflection --> selfservice

    classDef ai fill:#7b3fbf,color:#fff,stroke:#5a2d8c
    class extract,vector ai
```
