# AI-Powered IT Ticket Routing and Automated Resolution System

The system helps an enterprise receive, classify, and dispatch internal IT support requests — processing approximately **8,000 tickets per day** across corporate departments. The AI Service evaluates incoming requests: basic requests within scope are **approved by the AI and executed automatically through backend APIs**; requests requiring hands-on human intervention are routed directly to technicians with an AI summary; when context is missing, it **asks the requester for clarification**.

## 1. Requirements — User Stories and Use Cases

Requirements define what the system must do, who uses it, and the value delivered. The detailed specification is split into feature branches F01–F09; F02–F08 each have a folder under `docs/`, while F01 Authentication & Authorization and F09 User Management are not specified yet. This section gives the overview across branches.

### 1.1 Product mind map

![System Mind Map](docs/mindmap.png)

The outline for XMind import is in [the mind map outline](docs/mindmap-xmind.md).

The mind map groups features by area (8 groups); the specification folders are organized by feature branch. The groups map to the branches as follows:

| Mind map group | Specified in | Branch design status |
| :--- | :--- | :--- |
| 1. Requester Self-Service | [F03 Ticket Management](docs/f03-ticket-management/README.md); KB deflection in [F06](docs/f06-knowledge-base/README.md) | TARGET V1 |
| 2. Technician Workspace | [F03 Ticket Management](docs/f03-ticket-management/README.md); ticket summary on handoff in F03, AI resolution in [F04](docs/f04-ai-ticket-processing/README.md) | TARGET V1 |
| 3. Administration & Coordination | [F05 Workflow & Automation](docs/f05-workflow-automation/README.md) (rules, SLA, AI scope); RBAC and user directory in F09 (not specified) | TARGET V1 |
| 4. Knowledge Base | [F06 Knowledge Base](docs/f06-knowledge-base/README.md) | TARGET V1 |
| 5. AI Intelligent Processing | [F04 AI Ticket Processing](docs/f04-ai-ticket-processing/README.md) | TARGET V1 (draft for team review) |
| 6. Automation Workflow Engine | [F05 Workflow & Automation](docs/f05-workflow-automation/README.md) | TARGET V1 |
| 7. Approval & Automated Execution | [F05 Workflow & Automation](docs/f05-workflow-automation/README.md) | TARGET V1 |
| 8. Dashboard & Governance Telemetry | [F02 Dashboard](docs/f02-dashboard/README.md), [F07 Reports & Analytics](docs/f07-reports-analytics/README.md), [F08 Activity Log](docs/f08-activity-log/README.md) | F02 draft; F07, F08 TARGET V1 (draft for team review) |
| Login (no group) | F01 Authentication & Authorization (not specified) | — |

The mind map is an early outline and is out of date in a few places. The system has four actors (see [1.2](#12-user-stories)), so the Team Lead role in the mind map is dropped; some branch docs still name Team Lead and are being aligned. The technician summary (at most 150 words rather than 3 lines) and the reopen window (7 days rather than 48 hours) follow the branch docs. The "Manager" who approves elevated permissions in groups 7.2 and 7.3 is the Administrator. Whether some sensitive entitlements still need that approval is open (US-WF-15 in F05); the agreed direction is to let the AI route and approve as many requests as possible, so that the Administrator's manual routing and approval work is kept to a minimum.

### 1.2 User stories

**Actors**

The system has four actors: three human roles (Requester, Technician, Administrator) and the AI Service.

*   **Requester (Người gửi ticket):** Business employee who submits tickets, tracks status, answers AI clarifying questions, verifies KB solutions, and confirms resolution.
*   **Technician (Kỹ thuật viên):** Support engineer who handles what the AI cannot resolve: troubleshooting, fixing, updating states and solutions, escalating to a higher tier, and authoring KB articles.
*   **Administrator (Admin):** System owner who configures routing rules, SLA policies, and AI auto-approval whitelists, reviews KB revisions via visual diff, and monitors operational dashboards and audit logs. The Administrator is also the approver for elevated permissions (the "Manager" in the mind map).
*   **AI Service (Hệ thống AI):** Intelligent background service that classifies and routes tickets, auto-resolves whitelisted requests, prompts for missing context, summarizes conversations, links duplicate cases, and synthesizes KB drafts from resolved tickets.

In the use case diagrams, **User** is the common parent of the three human roles (login, dashboard, KB search).

**Core backlog**

This is the system-level backlog: 22 stories across the branches. The last column lists the detailed stories in the branch folders, which hold the acceptance criteria.

| Story ID | Role | User Story (`As a... I want to... So that...`) | Priority | Branch stories |
| :--- | :--- | :--- | :---: | :--- |
| **US01** | `REQUESTER` | **As a** Requester, **I want to** submit tickets with screenshots and error logs, **So that** the AI and technicians understand my issue without asking again. | **Must** | US-TK-01, US-TK-02 |
| **US02** | `REQUESTER` | **As a** Requester, **I want** real-time KB solution suggestions while typing, **So that** I can resolve issues without submitting a ticket. | **Must** | US-AI-05, US-TK-04, US-KB-08 |
| **US03** | `REQUESTER` | **As a** Requester, **I want to** answer AI clarifying questions inside my ticket, **So that** my incomplete request has sufficient context for routing. | **Must** | US-AI-03, US-AI-04, US-WF-09, US-TK-16 |
| **US04** | `REQUESTER` | **As a** Requester, **I want to** track ticket status and confirm resolution, **So that** I know when my issue is fixed. | **Must** | US-TK-05, US-TK-20, US-WF-19, US-NK-02 |
| **US05** | `REQUESTER` | **As a** Requester, **I want to** report "not fixed" or reopen a closed ticket within 7 days, **So that** I don't have to start over when the problem comes back. | **Must** | US-TK-21 |
| **US06** | `AI` | **As the** AI Service, **I want to** classify tickets and extract structured fields, **So that** requests are routed to the right flow and team. | **Must** | US-AI-01, US-AI-02 |
| **US07** | `AI` | **As the** AI Service, **I want to** auto-approve in-scope basic requests after the policy check and call execution APIs, **So that** repetitive tickets are resolved instantly. | **Must** | US-WF-08, US-WF-15, US-WF-17, US-AI-17, US-AI-20 |
| **US08** | `AI` | **As the** AI Service, **I want to** route tickets that need a human to the right technician team, and send tickets that match no rule or that I fail to process to a shared triage queue, **So that** no ticket is left without an owner. | **Must** | US-WF-10, US-WF-11, US-NK-07 |
| **US09** | `TECHNICIAN` | **As a** Technician, **I want to** see an AI summary of at most 150 words (problem, impact, key details, what was tried, why it came to a human) on every ticket routed to me, **So that** I grasp the problem without reading long threads. | **Must** | US-AI-07, US-AI-08 |
| **US10** | `TECHNICIAN` | **As a** Technician, **I want to** claim tickets from my team queue or the triage queue, update states, and record the resolution and root cause, **So that** the ticket lifecycle is cleanly tracked. | **Must** | US-TK-11, US-TK-17, US-TK-18 |
| **US11** | `TECHNICIAN` | **As a** Technician, **I want to** transfer a misrouted ticket to another team or escalate it to a higher tier with the work done so far, **So that** hard or misrouted tickets reach the right expertise in time. | **Must** | US-TK-12, US-TK-13 |
| **US12** | `TECHNICIAN` | **As a** Technician, **I want** tickets whose automated execution or verification failed to be handed to me with the error log, **So that** I can retry or resolve them manually and no automation failure is forgotten. | **Must** | US-WF-20 |
| **US13** | `TECHNICIAN` | **As a** Technician, **I want to** be reminded before a ticket breaches its SLA, with the SLA paused while the requester owes an answer, **So that** I can act in time and am not penalized for delays outside my control. | **Must** | US-WF-12, US-WF-04 |
| **US14** | `TECHNICIAN` | **As a** Technician, **I want** AI to synthesize a KB article draft from a resolved ticket with 1-click, **So that** know-how is captured with zero effort. | **Must** | US-KB-04 |
| **US15** | `TECHNICIAN` | **As a** Technician, **I want to** author and revise troubleshooting guides in rich text/markdown, **So that** procedures are standardized. | **Must** | US-KB-02, US-KB-03 |
| **US16** | `TECHNICIAN` | **As a** Technician, **I want** a personal workbench with my tickets sorted by priority and SLA and figures for my team, **So that** I know what to work on next. | **Must** | US-DB-08, US-BC-02 |
| **US17** | `ADMINISTRATOR` | **As an** Administrator, **I want to** configure rule-based routing and SLA timers (P1–P4), **So that** ticket allocation matches team capacity. | **Must** | US-WF-01, US-WF-02, US-WF-03 |
| **US18** | `ADMINISTRATOR` | **As an** Administrator, **I want to** define the AI auto-execution whitelist, entitlement matrix and confidence thresholds, **So that** automated actions are strictly governed. | **Must** | US-WF-05, US-WF-06, US-WF-07 |
| **US19** | `ADMINISTRATOR` | **As an** Administrator, **I want to** review KB articles using a side-by-side Visual Diff, **So that** only validated steps are published. | **Must** | US-KB-05 |
| **US20** | `ADMINISTRATOR` | **As an** Administrator, **I want** an operations dashboard with ticket volume, SLA risk and breached tickets, and to be notified when a ticket breaches its SLA, **So that** I can step in before incidents pile up. | **Must** | US-DB-01, US-DB-03, US-WF-13 |
| **US21** | `ADMINISTRATOR` | **As an** Administrator, **I want** reports on the AI self-resolution rate, handling time, SLA breaches and satisfaction, with CSV/XLSX export, **So that** I can decide whether to widen or narrow what the AI may do. | **Must** | US-BC-03, US-BC-04, US-BC-05, US-BC-06 |
| **US22** | `ADMINISTRATOR` | **As an** Administrator, **I want** an immutable activity log of every user, AI and system action, including why the AI approved, asked back or routed each request, **So that** AI decisions and incidents can be reviewed and trusted. | **Must** | US-NK-03, US-NK-04, US-NK-05, US-WF-16 |

Each branch keeps its detailed backlog with acceptance criteria in its own folder; see [Documentation](#documentation).

### 1.3 Use cases

#### Role-based use case allocation

The system organizes capabilities across discrete use cases mapped to the 4 operational roles (Admin, Requester, Technician, AI Service):

```mermaid
flowchart TB
    subgraph actors["System Actors & Roles"]
        direction LR
        admin(["1. Admin"]):::actorPill
        requester(["2. Requester"]):::actorPill
        tech(["3. Technician"]):::actorPill
        ai(["4. AI"]):::actorPill
    end

    subgraph systemBoundary["AI-Powered IT Service Desk — System Boundary (Role-Based Use Case Allocation)"]
        direction LR

        cardAdmin["<b>1. Admin</b><br/><br/>• <b>UC01</b> Login & SSO Auth<br/>• <b>UC02</b> Logout<br/>• <b>UC03</b> View Operational Dashboard<br/>• <b>UC11</b> Author & Edit KB Articles<br/>• <b>UC13</b> Review & Publish KB (Diff View)<br/>• <b>UC14</b> Configure Rules, SLA & Scope<br/>• <b>UC21</b> Operational Reports & Analytics<br/>• <b>UC22</b> System Audit Trail & Logs"]:::colCard

        cardReq["<b>2. Requester</b><br/><br/>• <b>UC01</b> Login & SSO Auth<br/>• <b>UC02</b> Logout<br/>• <b>UC03</b> View Personal Dashboard<br/>• <b>UC04</b> Submit Ticket & Evidence<br/>• <b>UC05</b> Track, Confirm & Reopen Ticket<br/>• <b>UC06</b> Search KB Solutions (Self-Service)<br/>• <b>UC07</b> Rate & Report KB Article<br/>• <b>UC20</b> Communicate on Ticket"]:::colCard

        cardTech["<b>3. Technician</b><br/><br/>• <b>UC01</b> Login & SSO Auth<br/>• <b>UC02</b> Logout<br/>• <b>UC03</b> View Team Dashboard & Workload<br/>• <b>UC08</b> Claim & Triage Tickets<br/>• <b>UC09</b> Update Fix & Root Cause<br/>• <b>UC10</b> AI Synthesize KB from Ticket<br/>• <b>UC11</b> Author & Edit KB Articles<br/>• <b>UC12</b> Escalate Ticket to Tier 2/3<br/>• <b>UC19</b> SLA Reminders & Breach Alerts<br/>• <b>UC20</b> Communicate on Ticket"]:::colCard

        cardAI["<b>4. AI</b><br/><br/>• <b>UC10</b> AI Synthesize KB from Ticket<br/>• <b>UC15</b> Classify & Route Ticket<br/>• <b>UC16</b> Auto-Resolve & Execute Request<br/>• <b>UC17</b> Verify Result & Hand Off on Failure<br/>• <b>UC18</b> Summarize Ticket & Draft Reply<br/>• <b>UC06</b> Semantic KB Solution Matching"]:::colCard
    end

    admin --> cardAdmin
    requester --> cardReq
    tech --> cardTech
    ai --> cardAI

    classDef actorPill fill:#334155,color:#ffffff,stroke:#64748b,stroke-width:2px;
    classDef colCard fill:#1d4ed8,color:#ffffff,stroke:#60a5fa,stroke-width:2px;
```

#### Subsystem boundary and functional use case diagrams

The system has 22 use cases (`UC01` to `UC22`) shared by the three human roles (Requester, Technician, Administrator) and the AI Service. The first diagram shows how the human roles relate; the other four show the use cases grouped by functional domain. Purple nodes call the AI Service.

```mermaid
flowchart LR
    req["<b>Requester</b>"]
    tech["<b>Technician</b>"]
    admin["<b>Administrator</b>"]
    user["<b>User</b>"]

    req ==>|"«generalize»"| user
    tech ==>|"«generalize»"| user
    admin ==>|"«generalize»"| user

    classDef actor fill:#e2e8f0,stroke:#64748b,color:#0f172a
    class req,tech,admin,user actor
    linkStyle default stroke:#94a3b8,stroke-width:1.5px
```

```mermaid
flowchart LR
    subgraph left["Actors"]
        direction TB
        user["<b>User</b>"]
        req["<b>Requester</b>"]
    end

    subgraph system["System Boundary"]
        direction TB
        subgraph access["Access and dashboard"]
            direction TB
            uc01["UC01 Login & SSO Authentication"]
            uc02["UC02 Logout & Session Revocation"]
            uc03["UC03 View Dashboard & Telemetry"]
        end
        subgraph selfservice["Self-service"]
            direction TB
            uc04["UC04 Submit Ticket & Evidence"]
            uc05["UC05 Track, Confirm & Reopen Ticket"]
            uc06["UC06 Search KB Solutions"]
            uc07["UC07 Rate & Report KB Article"]
            uc20["UC20 Communicate on Ticket"]
        end
    end

    subgraph right["Systems"]
        direction TB
        sso(["Enterprise SSO"])
        aiSvc(["AI Service"])
    end

    req ==>|"«generalize»"| user

    user --> uc01 & uc02 & uc03 & uc06 & uc07
    req --> uc04 & uc05 & uc20
    uc01 --> sso
    uc04 & uc05 & uc06 --> aiSvc

    classDef actor fill:#e2e8f0,stroke:#64748b,color:#0f172a
    classDef usecase fill:#1168bd,color:#fff,stroke:#0b4f9e
    classDef ai fill:#7b3fbf,color:#fff,stroke:#5a2d8c
    classDef external fill:#666,color:#fff,stroke:#333
    class user,req actor
    class uc01,uc02,uc03,uc07,uc20 usecase
    class uc04,uc05,uc06 ai
    class sso,aiSvc external
    style left fill:transparent,stroke:#94a3b8,stroke-dasharray:5 5
    style right fill:transparent,stroke:#94a3b8,stroke-dasharray:5 5
    style system fill:transparent,stroke:#3b82f6,stroke-dasharray:5 5
    style access fill:transparent,stroke:#94a3b8
    style selfservice fill:transparent,stroke:#94a3b8
    linkStyle default stroke:#94a3b8,stroke-width:1.5px
```

```mermaid
flowchart LR
    subgraph left["Actors"]
        direction TB
        tech["<b>Technician</b>"]
    end

    subgraph system["System Boundary"]
        direction TB
        subgraph handling["Ticket handling"]
            direction TB
            uc08["UC08 Claim & Triage Tickets"]
            uc09["UC09 Update Fix & Root Cause"]
            uc12["UC12 Escalate Ticket to Tier 2/3"]
            uc20["UC20 Communicate on Ticket"]
            uc19["UC19 Receive SLA Reminders & Breach Alerts"]
        end
    end

    tech --> uc08 & uc09 & uc12 & uc20 & uc19

    classDef actor fill:#e2e8f0,stroke:#64748b,color:#0f172a
    classDef usecase fill:#1168bd,color:#fff,stroke:#0b4f9e
    class tech actor
    class uc08,uc09,uc12,uc20,uc19 usecase
    style left fill:transparent,stroke:#94a3b8,stroke-dasharray:5 5
    style system fill:transparent,stroke:#3b82f6,stroke-dasharray:5 5
    style handling fill:transparent,stroke:#94a3b8
    linkStyle default stroke:#94a3b8,stroke-width:1.5px
```

```mermaid
flowchart LR
    subgraph left["Actors"]
        direction TB
        ai["<b>AI Service</b>"]
    end

    subgraph system["System Boundary"]
        direction TB
        subgraph aiwork["AI processing and automation"]
            direction TB
            uc15["UC15 Classify & Route Ticket"]
            uc16["UC16 Auto-resolve & Execute Request"]
            uc17["UC17 Verify Result & Hand Off on Failure"]
            uc18["UC18 Summarize Ticket & Draft Reply"]
        end
    end

    subgraph right["Actors and systems"]
        direction TB
        tech["<b>Technician</b>"]
        apis(["Backend APIs<br/>(identity and access systems)"])
    end

    ai --> uc15 & uc16 & uc17 & uc18
    uc16 & uc17 --> apis
    uc17 & uc18 --> tech

    classDef actor fill:#e2e8f0,stroke:#64748b,color:#0f172a
    classDef ai fill:#7b3fbf,color:#fff,stroke:#5a2d8c
    classDef external fill:#666,color:#fff,stroke:#333
    class ai,tech actor
    class uc15,uc16,uc17,uc18 ai
    class apis external
    style left fill:transparent,stroke:#94a3b8,stroke-dasharray:5 5
    style right fill:transparent,stroke:#94a3b8,stroke-dasharray:5 5
    style system fill:transparent,stroke:#3b82f6,stroke-dasharray:5 5
    style aiwork fill:transparent,stroke:#94a3b8
    linkStyle default stroke:#94a3b8,stroke-width:1.5px
```

```mermaid
flowchart LR
    subgraph left["Actors"]
        direction TB
        tech["<b>Technician</b>"]
        admin["<b>Administrator</b>"]
    end

    subgraph system["System Boundary"]
        direction TB
        subgraph kb["Knowledge base"]
            direction TB
            uc10["UC10 AI Synthesize KB from Ticket"]
            uc11["UC11 Author & Edit KB Articles"]
            uc13["UC13 Review & Publish (Diff View)"]
        end
        subgraph config["Configuration and oversight"]
            direction TB
            uc14["UC14 Configure Rules, SLA & Scope"]
            uc19["UC19 Receive SLA Reminders & Breach Alerts"]
            uc21["UC21 View & Export Reports"]
            uc22["UC22 View Activity Log"]
        end
    end

    subgraph right["Systems"]
        direction TB
        aiSvc(["AI Service"])
    end

    tech --> uc10 & uc11
    admin --> uc11 & uc13 & uc14 & uc19 & uc21 & uc22
    uc10 --> aiSvc

    classDef actor fill:#e2e8f0,stroke:#64748b,color:#0f172a
    classDef usecase fill:#1168bd,color:#fff,stroke:#0b4f9e
    classDef ai fill:#7b3fbf,color:#fff,stroke:#5a2d8c
    classDef external fill:#666,color:#fff,stroke:#333
    class tech,admin actor
    class uc11,uc13,uc14,uc19,uc21,uc22 usecase
    class uc10 ai
    class aiSvc external
    style left fill:transparent,stroke:#94a3b8,stroke-dasharray:5 5
    style right fill:transparent,stroke:#94a3b8,stroke-dasharray:5 5
    style system fill:transparent,stroke:#3b82f6,stroke-dasharray:5 5
    style kb fill:transparent,stroke:#94a3b8
    style config fill:transparent,stroke:#94a3b8
    linkStyle default stroke:#94a3b8,stroke-width:1.5px
```

**Use case catalog**

| ID | Use Case Name | Primary Actor | Scope & Description | Branch use cases |
| :--- | :--- | :--- | :--- | :--- |
| **UC01** | Login & SSO Authentication | All human roles | Authenticate via enterprise SSO and load role-based workspace permissions. | — (F01 not specified) |
| **UC02** | Logout & Session Revocation | All human roles | Terminate session tokens and clear client-side caches. | — (F01 not specified) |
| **UC03** | View Dashboard & Telemetry | All human roles | View role-scoped workspaces (self-service action cards, active request tracking and KB deflection for Requesters; personal workbench and unassigned team pool for Technicians; enterprise live KPIs, SLA radar and incident stream for Administrators). | UC-DB-01 … UC-DB-05 |
| **UC04** | Submit Ticket & Evidence | Requester | Submit a request in free text through the portal or the support mailbox, with error logs and screenshots; matching KB articles are suggested while typing. | UC-TK-01, UC-AI-03 |
| **UC05** | Track, Confirm & Reopen Ticket | Requester | View the status timeline, answer AI clarifying questions, confirm the fix and rate it, reopen within 7 days, or cancel a request no longer needed. | UC-TK-02, UC-TK-06, UC-AI-02 |
| **UC06** | Search KB Solutions | All human roles | Natural language semantic search for verified solutions with instant answer extraction. | UC-KB-01 |
| **UC07** | Rate & Report KB Article | All human roles | Submit helpfulness votes (Helpful / Unhelpful) or report outdated/inaccurate technical steps. | UC-KB-05 |
| **UC08** | Claim & Triage Tickets | Technician | Claim tickets from the team queue or from the shared triage queue (tickets the AI could not route), or transfer a misrouted ticket to another team. | UC-TK-02, UC-TK-03, UC-DB-04 |
| **UC09** | Update Fix & Root Cause | Technician | Transition ticket states (In Progress, Needs Info, Resolved) and record resolution and root cause. | UC-TK-05 |
| **UC10** | AI Synthesize KB from Ticket | Technician | 1-click generation of structured KB draft from closed ticket with PII scrubbing. | UC-KB-04 |
| **UC11** | Author & Edit KB Articles | Technician / Admin | Author technical guides with rich text, markdown, code blocks, and tags. | UC-KB-02 |
| **UC12** | Escalate Ticket to Tier 2/3 | Technician | Transfer complex or blocked tickets to a higher support tier with the work done so far. | UC-TK-03 |
| **UC13** | Review & Publish (Diff View) | Administrator | Inspect proposed article revisions side-by-side (Diff View) and approve for release. | UC-KB-03 |
| **UC14** | Configure Rules, SLA & Scope | Administrator | Define deterministic assignment rules, SLA policies, and AI auto-approval whitelist. | UC-WF-01, UC-WF-02, UC-WF-03 |
| **UC15** | Classify & Route Ticket | AI Service | Read the ticket, classify it, fill the structured fields, suggest a priority and flag duplicates; then auto-resolve it, ask the requester one question, or route it to a technician team (the shared triage queue when no rule matches). | UC-AI-01, UC-AI-05, UC-AI-06, UC-WF-04, UC-WF-05 |
| **UC16** | Auto-resolve & Execute Request | AI Service | Approve in-scope requests after the deterministic policy check and call the backend APIs; domain agents answer known how-to questions with cited KB steps. | UC-WF-07, UC-WF-08, UC-AI-10 |
| **UC17** | Verify Result & Hand Off on Failure | AI Service | Verify the result on the target system and ask the requester to confirm; hand failed executions to a technician with the error log. | UC-WF-09 |
| **UC18** | Summarize Ticket & Draft Reply | AI Service | Give the technician a summary of at most 150 words when a ticket comes to a human, and draft the resolution and the reply when it is resolved. | UC-AI-04, UC-AI-08 |
| **UC19** | Receive SLA Reminders & Breach Alerts | Technician / Administrator | The SLA scheduler reminds the assignee before a deadline and alerts the Administrator when a ticket breaches its SLA. | UC-WF-06, UC-DB-02 |
| **UC20** | Communicate on Ticket | Requester / Technician | Exchange public replies with attachments; Technicians add internal notes hidden from the requester. | UC-TK-04 |
| **UC21** | View & Export Reports | Administrator | Ticket volume, handling time and SLA breaches, AI self-resolution rate, satisfaction and team workload, with CSV/XLSX export and a weekly email digest; Technicians see figures for their own scope. | UC-BC-01 … UC-BC-08 |
| **UC22** | View Activity Log | Administrator | Search the append-only log of user, AI and system actions, including AI approval decisions and workflow run errors; Technicians and Requesters see the history of their tickets. | UC-NK-01 … UC-NK-05 |

The use cases per branch (UC-DB, UC-TK, UC-AI, UC-WF, UC-KB, UC-BC, UC-NK) are specified in each branch folder. Two branch use cases are left out of this overview on purpose: UC-AI-07 (feedback and learning from verified tickets) and UC-AI-09 (structuring a business requirement) are specified in F04 only.

## 2. Information Architecture, Screen Hierarchy and UI/UX

The information architecture organizes features around each role's goals and follows the latest mind map. Navigation is rendered from the user's role token; every screen must define its access rules and its loading, empty, validation, error, saving and success states.

### 2.1 Information architecture

``` mermaid
flowchart TB
    ROOT["**IT Ticket Routing System**<br/>AI-powered ticket routing and processing"]

    ROOT --> PUBLIC["Public area"]
    ROOT --> SELF["Self-service area<br/>(login required, all roles)"]
    ROOT --> SUPPORT["Support workspace<br/>(Technician / Administrator)"]
    ROOT --> GOV["Governance area<br/>(Administrator)"]

    subgraph PUBLIC_PAGES["PUBLIC SCREENS"]
        direction TB
        LOGIN["**Login** /login<br/>Microsoft SSO<br/>Role-based home after login<br/>Authentication errors"]
    end

    subgraph SELF_PAGES["SELF-SERVICE SCREENS"]
        direction TB
        PORTAL["**Portal** /portal<br/>Action needed: answer the AI, confirm a result<br/>My open requests<br/>Announcements, KB search"]
        NEWREQ["**New request** /tickets/new<br/>Free text, no category form<br/>For someone else, attachments<br/>Real-time KB suggestions"]
        MYTICKETS["**My tickets** /my-tickets<br/>Open, needs my action<br/>Resolved and closed, cancelled"]
        REQDETAIL["**Request detail** /tickets/:code<br/>Conversation with the AI and IT<br/>Guided troubleshooting steps<br/>Confirm and rate / not fixed"]
        KBEXPLORE["**KB search** /kb<br/>Semantic search<br/>Category browsing"]
        KBARTICLE["**KB article** /kb/articles/:id<br/>Step-by-step fix<br/>Helpful vote, report inaccurate"]
        PROFILE["**Profile** /profile<br/>Account information<br/>Role and permissions<br/>Change password (Microsoft account)"]
    end

    subgraph SUPPORT_PAGES["SUPPORT SCREENS"]
        direction TB
        WORKBENCH["**Workbench** /technician/dashboard<br/>My queue by SLA<br/>Claim from the sub-group queue<br/>Personal figures"]
        QUEUES["**Work queues** /tickets<br/>Assigned to me, sub-group queue<br/>On hold, due soon, breached<br/>Suspected bugs, failed automation"]
        CONSOLE["**Ticket console** /tickets/:code<br/>AI summary, escalation reason<br/>Fields filled by AI, domain form<br/>Cited KB steps, internal notes<br/>Transfer, escalate, resolve with AI draft<br/>Automation tab: retry, resolve manually"]
        KBEDITOR["**KB editor** /kb/editor<br/>Rich text / Markdown<br/>AI draft from a resolved ticket"]
        KBFLAG["**Flagged articles** /kb/flagged<br/>Low-rated and reported articles"]
        REPORTS["**Reports** /analytics/*<br/>Tickets, SLA, satisfaction<br/>(own scope for technicians)"]
    end

    subgraph GOV_PAGES["GOVERNANCE SCREENS"]
        direction TB
        DASHBOARD["**Dashboard** /admin/dashboard<br/>Volume and AI resolution rate<br/>SLA breach radar, P1 alerts<br/>Team workload"]
        AIWF["**AI and workflow** /workflow/*<br/>AI decisions and their grounds<br/>AI scope, whitelist, entitlements<br/>Workload scoring, SLA and calendar<br/>Learning examples"]
        APPROVALS["**KB approvals** /kb/approvals<br/>Side-by-side visual diff<br/>Approve / request changes"]
        ANALYTICS["**Analytics** /analytics/*<br/>AI self-resolution rate<br/>Report settings, CSV / XLSX export"]
        AUDIT["**Audit** /audit/*<br/>Activity log<br/>Workflow runs"]
        USERS["**Users and teams** /admin/users, /admin/teams<br/>Add, edit, enable / disable users<br/>Roles and access scope<br/>Teams, sub-groups and members"]
    end

    PUBLIC --> LOGIN

    SELF --> PORTAL
    PORTAL --> NEWREQ
    PORTAL --> MYTICKETS
    MYTICKETS --> REQDETAIL
    NEWREQ --> REQDETAIL
    NEWREQ --> KBARTICLE
    SELF --> KBEXPLORE
    KBEXPLORE --> KBARTICLE
    SELF --> PROFILE

    SUPPORT --> WORKBENCH
    WORKBENCH --> CONSOLE
    SUPPORT --> QUEUES
    QUEUES --> CONSOLE
    CONSOLE --> KBEDITOR
    SUPPORT --> KBFLAG
    KBFLAG --> KBEDITOR
    SUPPORT --> REPORTS

    GOV --> DASHBOARD
    GOV --> AIWF
    GOV --> APPROVALS
    GOV --> ANALYTICS
    GOV --> AUDIT
    GOV --> USERS

    classDef root fill:#ffffff,stroke:#222222,stroke-width:2px,color:#111111;
    classDef public fill:#fff0f0,stroke:#f87171,color:#7f1d1d;
    classDef self fill:#ecfdf5,stroke:#22c55e,color:#14532d;
    classDef support fill:#eff6ff,stroke:#60a5fa,color:#1e3a8a;
    classDef gov fill:#f5f3ff,stroke:#a78bfa,color:#4c1d95;
    classDef publicArea fill:#fecaca,stroke:#dc2626,stroke-width:2px,color:#7f1d1d;
    classDef selfArea fill:#bbf7d0,stroke:#16a34a,stroke-width:2px,color:#14532d;
    classDef supportArea fill:#bfdbfe,stroke:#2563eb,stroke-width:2px,color:#1e3a8a;
    classDef govArea fill:#ddd6fe,stroke:#7c3aed,stroke-width:2px,color:#4c1d95;
    class ROOT root;
    class PUBLIC publicArea;
    class SELF selfArea;
    class SUPPORT supportArea;
    class GOV govArea;
    class LOGIN public;
    class PORTAL,NEWREQ,MYTICKETS,REQDETAIL,KBEXPLORE,KBARTICLE,PROFILE self;
    class WORKBENCH,QUEUES,CONSOLE,KBEDITOR,KBFLAG,REPORTS support;
    class DASHBOARD,AIWF,APPROVALS,ANALYTICS,AUDIT,USERS gov;
```

Access per area follows the role matrix: the self-service area is open to every role; the support workspace is for Technicians and Administrators and hidden (403) for Requesters; the governance area is Administrator only. Request detail and ticket console are the same page (`/tickets/:code`): requesters see the conversation, the AI's answers and guidance, and the confirm actions; staff also see the summary, the AI data, internal notes and the automation tab. The top bar of every role has *+ New request*, notifications (`/notifications`), the user menu (Profile, Log out) and, for technicians, the ON / OFF availability switch.

### 2.2 Screen hierarchy

``` mermaid
flowchart TB
    Login["Login (Microsoft SSO)"] --> Shell["Workspace shell<br/>(sidebar + top bar)"]
    Shell -->|"Requester"| Portal["Portal"]
    Shell -->|"Technician"| Workbench["Workbench"]
    Shell -->|"Administrator"| Dashboard["Dashboard"]

    Portal --> NewReq["New request"]
    NewReq -->|"KB suggestion solves it"| Article["KB article"]
    NewReq -->|"Submit"| ReqDetail["Request detail"]
    Portal --> MyTickets["My tickets"]
    MyTickets --> ReqDetail
    ReqDetail -->|"AI asks for missing data"| Answer["Answer in the conversation"]
    Answer --> ReqDetail
    ReqDetail -->|"Resolved"| Confirm["Confirm and rate / Not fixed"]
    Portal --> Explore["KB search"]
    Explore --> Article

    Workbench -->|"Claim"| Console["Ticket console"]
    Workbench --> Queues["Work queues"]
    Queues --> Console
    Console -->|"Transfer / escalate / return"| Queues
    Console -->|"Resolve"| Synth["AI KB draft (1-click)"]
    Synth --> Editor["KB editor"]
    Editor -->|"Submit for review"| Approvals["KB approvals (diff view)"]
    Approvals -->|"Publish"| Article

    Dashboard --> AIWF["AI and workflow"]
    Dashboard --> Analytics["Analytics"]
    Dashboard --> Audit["Audit"]
    Dashboard --> Users["Users and teams"]
    Shell --> Forbidden["403 Forbidden"]
    Shell --> NotFound["404 Not Found"]
```

Each role lands on its own home after login: Requesters on the Portal, Technicians on the workbench and Administrators on the dashboard. Main requester flow: Portal → New request (free text) → KB suggestions → Submit → the AI routes the ticket (F05) → the domain AI agent answers from the KB, guides the requester, or executes an approved request and verifies it (F04); missing data is asked in the conversation; tickets the agent cannot resolve go to a technician → Confirm and rate. Main technician flow: Workbench → Claim → Ticket console with the AI summary and what the agent already tried → Resolve with the AI-drafted Resolution and reply → 1-click AI KB draft → Edit → Administrator review in diff view → Publish.

The ticket console must clearly show the AI summary, the reason the ticket came to a person, the fields filled by the AI (correctable with a reason), the SLA timer and the conversation with internal notes kept separate. Anything the AI did (answer, guidance step, executed action) is visible on the ticket with its result, and failed executions fall back to a technician. Email links never change a ticket with one click; they open the ticket with the action pre-selected.

Each branch details its own screens: [F02 Dashboard](docs/f02-dashboard/information-architecture/README.md), [F03 Ticket Management](docs/f03-ticket-management/information-architecture/README.md), [F04 AI Ticket Processing](docs/f04-ai-ticket-processing/information-architecture/README.md), [F05 Workflow & Automation](docs/f05-workflow-automation/information-architecture/README.md), [F06 Knowledge Base](docs/f06-knowledge-base/information-architecture/README.md), [F07 Reports & Analytics](docs/f07-reports-analytics/information-architecture/README.md), [F08 Activity Log](docs/f08-activity-log/information-architecture/README.md). Where a branch uses another route or home page, the routes and homes above apply. Users and teams (F09) and Profile (F10) are not specified yet; who approves access outside the catalog, and where, is still open.

### 2.3 Current UI design

The screens are high-fidelity designs drawn in Figma on the shared template *Service Desk UI – DS-01 Relay* and exported to PNG. They follow the role areas of [section 2.1](#21-information-architecture): self-service for the Requester, the support workspace for the Technician, and the governance area for the Administrator. Every label is written in plain language; internal codes stay in the branch documents.

Every screen, by role:

| Role | Screens | Folder |
|---|---|---|
| Requester | 8 PNG | [docs/image/requester/](docs/image/requester/) |
| Technician | 8 PNG | [docs/image/technician/](docs/image/technician/) |
| Administrator | 19 PNG | [docs/image/admin/](docs/image/admin/) |

Screen notes and states are in the branch folders, for example [F07 UI design](docs/f07-reports-analytics/ui-design/README.md), [F08 UI design](docs/f08-activity-log/ui-design/README.md) and [F09 UI draft](docs/f09-user-management/ui-design/README.md).

#### Requester — self-service

##### 1. Sign in with the company account

![Login](docs/image/requester/01_Login-SSO.png)

Sign in with the company Microsoft or Google account; each role lands on its own home page (the demo environment lets you pick a role).

##### 2. Portal (SCR-DB-03)

![Portal](docs/image/requester/02_SCR-DB-03_Portal.png)

The requester home: search for help, requests that need an action (answer the AI, confirm a fix), active requests with progress, announcements and quick services.

##### 3. New request (SCR-T-01)

![New request](docs/image/requester/03_SCR-T-01_New-request.png)

Subject, category and a free-text description, on behalf of a colleague if needed, with attachments; matching solutions are suggested on the right while typing.

##### 4. My requests (SCR-T-03)

![My requests](docs/image/requester/04_SCR-T-03_My-requests.png)

Every request with its status, priority and time left, grouped in tabs such as *Needs my action* and *Resolved & closed*.

##### 5. Request detail (SCR-T-02)

![Request detail](docs/image/requester/05_SCR-T-02_Request-detail.png)

The conversation with the AI and the technician, a banner when an answer is needed, the progress steps, the time left and what the AI understood from the request.

##### 6. Knowledge base search (SCR-KB-01)

![KB search](docs/image/requester/06_SCR-KB-01_KB-search.png)

Describe the problem in your own words; a suggested answer comes first, then verified articles.

##### 7. Knowledge base article (SCR-KB-02)

![KB article](docs/image/requester/07_SCR-KB-02_KB-article.png)

Problem, cause and step-by-step fix, with *Was this article helpful?* and related solutions.

##### 8. Profile

![Profile](docs/image/requester/08_Profile.png)

Account details from the company directory, the role and what it allows, and a link to change the password with Microsoft.

#### Technician — support workspace

##### 1. Workbench (SCR-DB-02)

![Technician workbench](docs/image/technician/01_SCR-DB-02_Technician-workbench.png)

The technician home: today's queue with SLA timers, and the sub-group queue to claim tickets from.

##### 2. Work queues (SCR-T-04)

![Work queues](docs/image/technician/02_SCR-T-04_Work-queues.png)

Assigned tickets and team queues with filters, SLA time left and why each ticket came to a person.

##### 3. Ticket detail (SCR-T-02)

![Ticket detail](docs/image/technician/03_SCR-T-02_Ticket-detail.png)

The ticket console: AI summary, why it came to a person, fields filled by the AI, SLA timer, conversation and internal notes.

##### 4. Article authoring (SCR-KB-03)

![Article authoring studio](docs/image/technician/04_SCR-KB-03_Article-authoring-studio.png)

Edit a solution guide (problem, cause, steps) with a publishing checklist, then submit it for review.

##### 5. Flagged content (SCR-KB-06)

![Flagged content maintenance](docs/image/technician/05_SCR-KB-06_Flagged-content-maintenance.png)

Articles flagged as outdated or unhelpful, waiting for an update.

##### 6. Reports: Tickets (SCR-BC-01)

![Reports Tickets](docs/image/technician/06_SCR-BC-01_Reports-Tickets.png)

Ticket volume and backlog for the technician's own teams.

##### 7. Reports: SLA (SCR-BC-02)

![Reports SLA](docs/image/technician/07_SCR-BC-02_Reports-SLA.png)

Response and resolution times, with the tickets that breached or are at risk.

##### 8. Reports: Satisfaction (SCR-BC-04)

![Reports Satisfaction](docs/image/technician/08_SCR-BC-04_Reports-Satisfaction.png)

Requester ratings and response rate for the technician's own teams.

#### Administrator — governance


##### 2. AI decisions (SCR-WF-30)

![AI decisions](docs/image/admin/02_SCR-WF-30_AI-decisions.png)

Every routing decision of the AI with its outcome: done automatically, asked for details, or sent to a person.

##### 3. AI decision detail (SCR-WF-31)

![AI decision detail](docs/image/admin/03_SCR-WF-31_AI-decision-detail.png)

Why the AI decided: each rule it checked and the values it compared.


##### 5. AI actions and thresholds (SCR-WF-03)

![AI actions and thresholds](docs/image/admin/05_SCR-WF-03_AI-actions-thresholds.png)

How sure the AI must be before acting, which AI actions are on, and which domains the AI agent handles first.

##### 6. Approved actions list (SCR-WF-03)

![Whitelist](docs/image/admin/06_SCR-WF-03-06_Whitelist.png)

The requests the AI may approve and execute by itself.

##### 7. Access by department (SCR-WF-03)

![Entitlement matrix](docs/image/admin/07_SCR-WF-03-07_Entitlement-matrix.png)

Which department may get which access without a manual approval.

##### 8. Workload scoring (SCR-WF-01)

![Workload scoring](docs/image/admin/08_SCR-WF-01-05_Workload-scoring.png)

How the system weighs open tickets to pick the least busy technician.

##### 9. Business calendar (SCR-WF-02)

![Business calendar](docs/image/admin/09_SCR-WF-02_Business-calendar.png)

Working hours and holidays; SLA timers pause outside them.

##### 10. Reports: AI (SCR-BC-03)

![Reports AI](docs/image/admin/10_SCR-BC-03_Reports-AI.png)

How many tickets the AI solved alone each week against the 50% target, how it solved them, and why the rest went to a technician.

##### 11. Report settings (SCR-BC-05)

![Report settings](docs/image/admin/11_SCR-BC-05_Report-settings.png)

Targets, limits and the weekly summary e-mail; every change is saved in the activity log.

##### 12. Export (SCR-BC-10)

![Export](docs/image/admin/12_SCR-BC-10_Export.png)

Download the current report as CSV or Excel with the filters on screen.

##### 13. Export states (SCR-BC-10)

![Export states](docs/image/admin/13_SCR-BC-10_Export-states.png)

Ready, too many rows, and export not possible right now.

##### 14. Activity log (SCR-NK-02)

![Activity log](docs/image/admin/14_SCR-NK-02_Activity-log.png)

Everything done by people, the AI and automatic rules, with what changed written as sentences. Entries cannot be changed or deleted.

##### 15. Automatic actions (SCR-NK-04)

![Automatic actions](docs/image/admin/15_SCR-NK-04_Workflow-runs.png)

Changes the system made by itself, with a warning when many fail for the same reason.

##### 16. Automatic action detail (SCR-NK-05)

![Automatic action detail](docs/image/admin/16_SCR-NK-05_Workflow-run-detail.png)

What happened step by step, why it failed, and what to do next.

##### 17. Users (/admin/users)

![Users](docs/image/admin/17_SCR-UM-01_Users.png)

Find users, see their role, teams and availability, and enable or disable them alone or in bulk.

##### 18. Edit user

![Edit user drawer](docs/image/admin/18_SCR-UM-02_Edit-user-drawer.png)

Change the role, the teams a technician works for, and whether they receive new tickets.

##### 19. Teams and sub-groups (/admin/teams)

![Teams and sub-groups](docs/image/admin/19_SCR-UM-03_Teams.png)

The team structure used to route tickets, with the members, their availability and how busy they are.

## Documentation

- System overview: [Mind map outline](docs/mindmap-xmind.md), [Information architecture (section 2)](#2-information-architecture-screen-hierarchy-and-uiux); the older draft [docs/information-architecture.md](docs/information-architecture.md) is superseded.
- F02 Dashboard: [Index](docs/f02-dashboard/README.md), [Features](docs/f02-dashboard/01-features.md), [User stories](docs/f02-dashboard/02-user-stories.md), [Use cases](docs/f02-dashboard/03-use-cases.md), [Acceptance criteria](docs/f02-dashboard/04-acceptance-criteria.md), [INVEST backlog](docs/f02-dashboard/05-invest-requirements.md), [Information architecture](docs/f02-dashboard/information-architecture/README.md).
- F03 Ticket Management: [Index](docs/f03-ticket-management/README.md), [User stories](docs/f03-ticket-management/01-user-stories.md), [Use cases](docs/f03-ticket-management/02-use-cases.md), [Information architecture](docs/f03-ticket-management/information-architecture/README.md).
- F04 AI Ticket Processing: [Index](docs/f04-ai-ticket-processing/README.md), [Features](docs/f04-ai-ticket-processing/01-features.md), [Use cases](docs/f04-ai-ticket-processing/02-use-cases.md), [User stories (INVEST)](docs/f04-ai-ticket-processing/03-user-stories-invest.md), [Acceptance criteria](docs/f04-ai-ticket-processing/04-acceptance-criteria.md), [Business rules](docs/f04-ai-ticket-processing/05-business-rules.md), [Information architecture](docs/f04-ai-ticket-processing/information-architecture/README.md).
- F05 Workflow & Automation: [Index](docs/f05-workflow-automation/README.md), [Features](docs/f05-workflow-automation/01-features.md), [Use cases](docs/f05-workflow-automation/02-use-cases.md), [Acceptance criteria](docs/f05-workflow-automation/03-acceptance-criteria.md), [INVEST backlog](docs/f05-workflow-automation/04-invest-requirements.md), [Information architecture](docs/f05-workflow-automation/information-architecture/README.md).
- F06 Knowledge Base: [Index](docs/f06-knowledge-base/README.md), [Mind map](docs/f06-knowledge-base/01-mindmap.md), [User stories](docs/f06-knowledge-base/02-user-stories.md), [Use cases](docs/f06-knowledge-base/03-use-cases.md), [Information architecture](docs/f06-knowledge-base/04-information-architecture.md).
- F07 Reports & Analytics: [Index](docs/f07-reports-analytics/README.md), [Features](docs/f07-reports-analytics/01-features.md), [Use cases](docs/f07-reports-analytics/02-use-cases.md), [User stories (INVEST)](docs/f07-reports-analytics/03-user-stories-invest.md), [Acceptance criteria](docs/f07-reports-analytics/04-acceptance-criteria.md), [Business rules](docs/f07-reports-analytics/05-business-rules.md), [Information architecture](docs/f07-reports-analytics/information-architecture/README.md).
- F08 Activity Log: [Index](docs/f08-activity-log/README.md), [Features](docs/f08-activity-log/01-features.md), [Use cases](docs/f08-activity-log/02-use-cases.md), [User stories (INVEST)](docs/f08-activity-log/03-user-stories-invest.md), [Acceptance criteria](docs/f08-activity-log/04-acceptance-criteria.md), [Business rules](docs/f08-activity-log/05-business-rules.md), [Information architecture](docs/f08-activity-log/information-architecture/README.md).
- Requirements catalogs (F05 Workflow & Automation so far): [Business rules](docs/requirements/business-rules.md), [Non-functional requirements](docs/requirements/non-functional-requirements.md).
- Traceability (F05 Workflow & Automation so far): [Master traceability matrix](docs/traceability/master-traceability.md). The system-level stories and use cases trace to the branches through the "Branch stories" and "Branch use cases" columns above.

### Repository layout

No source code exists yet. The repository holds the documents.

```text
Fable/
├── docs/
│   ├── f02-dashboard/             # Dashboard: features, user stories, use cases, AC, INVEST, IA
│   ├── f03-ticket-management/     # Ticket management: user stories, use cases, IA
│   ├── f04-ai-ticket-processing/  # AI ticket processing: features, use cases, INVEST, AC, business rules, IA
│   ├── f05-workflow-automation/   # Workflow and automation: features, use cases, AC, INVEST, IA
│   ├── f06-knowledge-base/        # Knowledge base: mind map, user stories, use cases, IA, wireframes
│   ├── f07-reports-analytics/     # Reports and analytics: features, use cases, INVEST, AC, business rules, IA
│   ├── f08-activity-log/          # Activity log: features, use cases, INVEST, AC, business rules, IA
│   ├── requirements/              # Business rules and non-functional requirements (F05 so far)
│   ├── traceability/              # Traceability matrix (F05 so far)
│   ├── information-architecture.md  # System-wide information architecture
│   ├── mindmap-xmind.md           # Mind map outline for XMind
│   └── mindmap.png                # Mind map image
└── README.md
```

Every branch folder is named `fNN-<branch>` after its feature branch number (F01–F09, mapped to the mind map groups in [1.1](#11-product-mind-map)), its files are numbered `NN-<topic>.md` (F02, F04, F05, F07 and F08 keep their IA in an `information-architecture/` subfolder), and its `README.md` lists the files.

## Project Progress

- [x] Draw the product mind map
- [x] Write the system user stories (22) and use cases (22), with the use case diagrams
- [x] Design the system information architecture and screen hierarchy
- [x] F02 Dashboard: features, user stories, use cases, acceptance criteria, INVEST, IA (draft)
- [x] F03 Ticket Management: user stories, use cases, IA (TARGET V1)
- [x] F04 AI Ticket Processing: features, use cases, INVEST, acceptance criteria, business rules, IA (TARGET V1, draft for team review)
- [x] F05 Workflow & Automation: features, use cases, INVEST, acceptance criteria, IA (TARGET V1)
- [x] F06 Knowledge Base: mind map, user stories, use cases, IA (TARGET V1)
- [x] F07 Reports & Analytics: features, use cases, INVEST, acceptance criteria, business rules, IA (TARGET V1, draft for team review)
- [x] F08 Activity Log: features, use cases, INVEST, acceptance criteria, business rules, IA (TARGET V1, draft for team review)
- [ ] Align the branch docs with the four actors (remove Team Lead) and the system IA role matrix
- [ ] Specify F01 Authentication & Authorization and F09 User Management
- [ ] C4 System Context, Container, Component
- [ ] Database design (ERD)
- [ ] OpenAPI 3.0
