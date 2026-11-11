# Information Architecture — F06 Knowledge Base (KB)

> Design status: TARGET V1 (aligned with F04 & F05 specifications)
>
> Implementation status: NOT STARTED

This folder specifies the screen hierarchy, navigation routes, user flows, and interaction states for **F06 Knowledge Base**. It maps all user stories (`US-KB-01` … `US-KB-08`) and use cases (`UC-KB-01` … `UC-KB-05`) into concrete user-facing interfaces, embedded host sections, and administrative governance consoles.

## Architecture & Integration Scope

F06 delivers both dedicated self-service/governance pages and embedded widgets hosted within F03 (Ticket Management) and F04 (AI Processing):

| Kind | ID Prefix | Meaning |
|---|---|---|
| Dedicated Page / Modal | `SCR-KB-xx` | Standalone page or modal owned and operated by F06 |
| Host Page Section | `SEC-KB-xx` | An F06 component embedded inside pages owned by F03 or F04 |
| User Flow | `FLOW-KB-xx` | End-to-end task flows across screens |

### Related Documents
* [Screens, Routes, and Navigation](01-screens-and-navigation.md)
* [User Flows](02-user-flows.md)
* [UI States and Accessibility](03-ui-states-and-accessibility.md)
* [F06 User Stories](../02-user-stories.md) · [F06 Use Cases](../03-use-cases.md)

---

## IA Architectural Decisions

| ID | Decision | Rationale |
|---|---|---|
| **D-KB-IA-01** | **Embedded Self-Service Deflection:** F06 embeds a real-time solution card section (`SEC-KB-01`) directly inside F03's ticket creation view (`/tickets/new`). | Deflects up to 35% of repetitive tickets before requesters hit submit. |
| **D-KB-IA-02** | **Isolated Revision Branching:** Editing an active published article forks an isolated draft revision (`v1.1 (Draft)`) rather than modifying live content in-place. | Ensures public readers continue accessing validated instructions (`v1.0`) while updates await approval. |
| **D-KB-IA-03** | **Role-Scoped Article Visibility:** Public Help Center queries automatically filter out all records tagged `INTERNAL_ONLY`. | Protects sensitive infrastructure guides, passwords, and internal IT procedures from non-IT staff. |
| **D-KB-IA-04** | **1-Click AI Ticket-to-KB Synthesis:** Embedded modal on resolved tickets triggers automated PII scrubbing and pre-fills structured fields. | Captures operational troubleshooting know-how with near-zero transcription overhead for technicians. |
| **D-KB-IA-05** | **Visual Diff Review Inspection:** Admin approval queue provides side-by-side color-coded diffing (Green = added, Red = removed). | Eliminates manual text auditing errors and accelerates lead review turnaround times. |
| **D-KB-IA-06** | **Automated Quality Feedback Loop:** Articles receiving 3+ negative reports within 30 days automatically enter the `FLAGGED` maintenance queue. | Guarantees obsolete or inaccurate technical instructions are promptly flagged and corrected. |

---

## Information Architecture Diagram

```mermaid
flowchart TB
    req["<b>Requester</b>"]
    tech["<b>Technician</b>"]
    admin["<b>Admin</b>"]

    subgraph public_portal["Public Help Center & Self-Service"]
        direction TB
        scr_kb_01["<b>SCR-KB-01</b> Search Portal<br/><code>/kb</code>"]
        scr_kb_02["<b>SCR-KB-02</b> Article Detail View<br/><code>/kb/articles/:id</code>"]
        scr_kb_07["<b>SCR-KB-07</b> Report Issue Modal"]
    end

    subgraph tech_workspace["Support & Authoring Workspace"]
        direction TB
        scr_kb_03["<b>SCR-KB-03</b> Article Authoring Studio<br/><code>/kb/editor</code>"]
        scr_kb_04["<b>SCR-KB-04</b> AI Synthesis Modal<br/>(on resolved ticket)"]
    end

    subgraph admin_gov["Governance & Quality Management"]
        direction TB
        scr_kb_05["<b>SCR-KB-05</b> Visual Diff Review Queue<br/><code>/kb/approvals</code>"]
        scr_kb_06["<b>SCR-KB-06</b> Flagged Article Maintenance<br/><code>/kb/flagged</code>"]
    end

    subgraph host_embed["Embedded in F03 / F04 Host Pages"]
        direction TB
        sec_kb_01["<b>SEC-KB-01</b> Real-time Deflection<br/>(on <code>/tickets/new</code>)"]
        sec_kb_02["<b>SEC-KB-02</b> Technician Solution Steps<br/>(on <code>/tickets/:id</code>)"]
    end

    req -.-> scr_kb_01 & scr_kb_02 & scr_kb_07 & sec_kb_01
    tech -.-> scr_kb_01 & scr_kb_02 & scr_kb_03 & scr_kb_04 & sec_kb_02
    admin -.-> scr_kb_01 & scr_kb_02 & scr_kb_03 & scr_kb_05 & scr_kb_06

    scr_kb_01 --> scr_kb_02
    scr_kb_02 --> scr_kb_07
    scr_kb_04 --> scr_kb_03
    scr_kb_03 --> scr_kb_05
    scr_kb_07 --> scr_kb_06

    classDef actor fill:#1e293b,stroke:#475569,stroke-width:1.5px,color:#f8fafc;
    classDef page fill:#1e40af,stroke:#60a5fa,stroke-width:1.5px,color:#ffffff;
    classDef embed fill:#334155,stroke:#94a3b8,stroke-width:1.5px,color:#ffffff;

    class req,tech,admin actor;
    class scr_kb_01,scr_kb_02,scr_kb_03,scr_kb_04,scr_kb_05,scr_kb_06,scr_kb_07 page;
    class sec_kb_01,sec_kb_02 embed;
```
