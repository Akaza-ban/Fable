# Information Architecture — F07 Reports & Analytics

> Design status: TARGET V1 (draft for team review)
>
> Implementation status: NOT STARTED

This folder places the 10 F07 stories on screens and emails. F07 owns **one page with tabs** (Reports), one settings page, one modal and one email. Its figures also appear on pages owned by other branches (F02 Dashboard, F05 SCR-WF-30), always through the same API. The inventory has four kinds of entries:

| Kind | ID | Meaning |
|---|---|---|
| F07 page or tab | `SCR-BC-xx` | A page or a tab of the Reports page |
| F07 modal | `SCR-BC-1x` | A modal opened on an F07 page, no route of its own |
| Host page section | `SEC-BC-xx` | F07 figures inside a page owned by F02 or F05. The host page keeps its own ID |
| Email | `EML-BC-xx` | An email F07 composes |

Inputs: [Use cases](../02-use-cases.md), [User stories](../03-user-stories-invest.md), [Acceptance criteria](../04-acceptance-criteria.md). Routes follow the system IA ([`/analytics/*`](../../information-architecture.md)); host IDs come from F03 (`SCR-T-02`, `SCR-T-04`) and F05 (`SCR-WF-02`, `SCR-WF-30`, `SCR-90`, `SCR-91`).

| File | Content |
|---|---|
| [01-screens-and-navigation.md](01-screens-and-navigation.md) | Screen, section and email inventory; routes and permissions; navigation per role; story-to-screen mapping; screen-to-API mapping |
| [02-user-flows.md](02-user-flows.md) | Three flows: weekly AI review, SLA bottleneck, export for management |
| [03-ui-states-and-accessibility.md](03-ui-states-and-accessibility.md) | Data state contract, chart rules, responsive layout, accessibility |

## IA decisions

| ID | Decision | Why |
|---|---|---|
| D-BC-IA-01 | **One Reports page with tabs** (Tickets, SLA, AI, Satisfaction) sharing one filter bar; filters live in the URL, so switching tabs keeps them and a link reopens the same view. | The reports answer one question from different angles; Admins compare them side by side. |
| D-BC-IA-02 | **F07 owns the metric, other pages host it.** F02 Dashboard widgets and F05 SCR-WF-30 cards call `/api/v1/reports/*` (SEC-BC-01, SEC-BC-02) and link to the full tab. | One number per metric everywhere (BR-BC-01). |
| D-BC-IA-03 | **Tabs are hidden, not disabled,** when the role may not open them (Technician: no AI tab). Opening the URL anyway gives SCR-90. | Same behaviour as F05 (D-IA-02). |
| D-BC-IA-04 | **Every figure drills down to the ticket list** (F03 SCR-T-04) with the same filters; nothing on a report page changes data. | Reports are read-only (BR-BC-03); the work happens in the ticket pages. |
| D-BC-IA-05 | **Report settings is a separate Admin page** under *Settings*, not a tab: it changes how figures are computed for everyone. | Different permission, audited (BR-BC-05). |
| D-BC-IA-06 | **The weekly digest is an email, not a page;** it links back to the tabs. | Management reads email; PDF is not in V1 (Q-BC-11). |

## Information architecture

Solid lines: a page and its tabs or modals. Dashed arrows: which areas each actor uses. Purple: F07-owned items. White: pages owned by other branches.

```mermaid
flowchart TB
    tech["<b>Technician</b>"]
    admin["<b>Admin</b>"]

    subgraph reports["Reports (F07) — /analytics"]
        direction TB
        r01["<b>SCR-BC-01 Tickets</b><br/>• Volume, backlog, trend<br/>• By type, domain, group, priority, channel"]
        r02["<b>SCR-BC-02 SLA</b><br/>• M-01, M-02, M-03<br/>• Breached / At risk lists"]
        r03["<b>SCR-BC-03 AI</b> (Admin)<br/>• M-04 by path and agent vs target<br/>• M-08, M-10, quality M-05/06/11"]
        r04["<b>SCR-BC-04 Satisfaction</b><br/>• Average + response rate<br/>• Ratings 1–2"]
        m10["<b>SCR-BC-10 Export</b> (modal, Admin)"]
    end

    subgraph settings["Settings — Admin"]
        r05["<b>SCR-BC-05 Report settings</b><br/>• N, thresholds, target, limits<br/>• Digest recipients"]
    end

    subgraph hosts["Hosted figures"]
        direction TB
        f02["<b>F02 Dashboard</b><br/>• SEC-BC-01 KPI widgets"]
        wf30["<b>SCR-WF-30 AI decisions</b> (F05)<br/>• SEC-BC-02 AI cards"]
    end

    subgraph tickets["Tickets (F03)"]
        t04["<b>SCR-T-04 Ticket list</b>"]
        t02["<b>SCR-T-02 Ticket detail</b>"]
    end

    eml["<b>EML-BC-01</b> Weekly digest"]

    r01 & r02 & r03 & r04 --- m10
    r01 & r02 & r03 & r04 --> t04
    t04 --> t02
    f02 --> r01
    wf30 --> r03
    eml --> r03
    r05 -. "SLA targets" .-> wf02["SCR-WF-02 SLA (F05)"]

    tech -.-> r01 & r02 & r04
    admin -.-> reports
    admin -.-> settings
    admin -.-> eml

    classDef actor fill:none,stroke:none,color:#111
    classDef host fill:#fff,color:#111,stroke:#1168bd,text-align:left
    classDef bc fill:#f3ecfb,color:#111,stroke:#7b3fbf,text-align:left
    class tech,admin actor
    class f02,wf30,t04,t02,wf02 host
    class r01,r02,r03,r04,r05,m10,eml bc
```

## Screen hierarchy per role

| Role | Home page | Where F07 shows up |
|---|---|---|
| Requester | My tickets (F03) | Nowhere |
| Technician | Work queues (F03) | Sidebar *Reports*: SCR-BC-01, 02, 04, scope only; SEC-BC-01 on the Dashboard |
| Admin | Dashboard (F02) | Every tab, SCR-BC-05, SCR-BC-10, EML-BC-01, SEC-BC-01, SEC-BC-02 |
| AI Service | — | No UI; its `AI_AGENT` account gets `403` on `/reports/*` |

**Related:** [F07 README](../README.md) · [Screens and navigation](01-screens-and-navigation.md) · [User flows](02-user-flows.md) · [UI states](03-ui-states-and-accessibility.md)
