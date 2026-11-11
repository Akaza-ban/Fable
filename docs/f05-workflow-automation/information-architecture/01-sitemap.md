# Sitemap by Role

> Design status: V1 — draft, awaiting user review
>
> Implementation status: NOT STARTED

**Notation** (D-IA-01)

- **Card:** one page, with its ID and main content.
- **Area (box):** a group of pages used by the same roles.
- **Solid arrow:** parent page → child page (where the child is opened from).
- **Dashed arrow:** the actor uses the pages in that area. The arrows match the [page-permission table](02-routes-and-permissions.md).
- **Purple card:** a page with AI features (AI-generated content, always labelled **AI**).
- Modals, the "Access denied" page and loading/error states are **not** cards; see the [screen inventory](04-screen-inventory.md) and the [UI states](06-ui-states-and-accessibility.md).
- Pages marked *(F03)* belong to the Ticket Management feature; only their workflow-related parts are described here.

```mermaid
flowchart TB
    req["<span style='font-size:40px'>👤</span><br/><b>Requester</b>"]
    tech["<span style='font-size:40px'>👤</span><br/><b>Technician</b><br/>(sets own availability)"]
    admin["<span style='font-size:40px'>👤</span><br/><b>Admin</b>"]

    subgraph access["Access"]
        direction TB
        scr01["<b>SCR-01 Login</b><br/>• Email, password<br/>• Goes to the role home page"]
    end

    subgraph tickets["Tickets (F03) — all roles, scope by role"]
        direction TB
        t03["<b>SCR-T-03 Ticket list</b><br/>• Requester: my tickets<br/>• Technician: assigned to me<br/>• Admin: all<br/>• Filter by status, flow A/B/C"]
        t01["<b>SCR-T-01 Create ticket</b><br/>• Free-text description<br/>• Quick templates: access request,<br/>new account, broken device"]
        t02["<b>SCR-T-02 Ticket detail</b><br/>• AI conversation (questions, answers)<br/>• Progress bar, SLA deadline<br/>• AI summary and routing reason (Technician)<br/>• Automation tab: API calls, errors<br/>• Confirm / Not fixed (Requester)"]
        t03 --> t01
        t03 --> t02
        t01 --> t02
    end

    subgraph notif["Notifications — all roles"]
        direction TB
        scr91["<b>SCR-91 Notifications</b><br/>• My events and reminders<br/>• Unread count in the top bar<br/>• Each item opens its object"]
    end

    subgraph config["Workflow configuration — Admin"]
        direction TB
        wf01["<b>SCR-WF-01 Assignment rules</b><br/>• Rules tab: conditions → team, order, Enable/Disable, Dry run<br/>• Workload tab: priority points, complexity by request type,<br/>maximum load, default support team"]
        wf02["<b>SCR-WF-02 SLA & business calendar</b><br/>• P1–P4 table: response, resolution, threshold<br/>• Working hours, holidays<br/>• Raise priority on breach"]
        wf03["<b>SCR-WF-03 AI scope</b><br/>• AI actions tab: on/off, threshold, clarification rounds<br/>• Automation actions tab: whitelist, connectors<br/>• Entitlement catalog tab by department"]
    end

    subgraph monitor["AI monitoring — Admin"]
        direction TB
        wf30["<b>SCR-WF-30 AI decisions</b><br/>• Self-resolution / clarification / human-routing rates<br/>• Decision list, filter by flow"]
        wf31["<b>SCR-WF-31 AI decision detail</b><br/>• Original ticket, AI-extracted information, confidence<br/>• Each policy condition ✓ / ✗<br/>• API execution history"]
        wf30 --> wf31
    end

    wf31 --> t02
    t02 --> wf31
    scr91 --> t02
    scr91 --> wf30

    req -.-> access
    req -.-> tickets
    req -.-> notif
    tech -.-> access
    tech -.-> tickets
    tech -.-> notif
    admin -.-> access
    admin -.-> tickets
    admin -.-> notif
    admin -.-> config
    admin -.-> monitor

    classDef ai fill:#7b3fbf,color:#fff,stroke:#5a2d8c
    class t02,wf30,wf31 ai
```

## Screens by role

| Role | Home page (DF-IA-02) | Pages available |
|---|---|---|
| **Requester** | SCR-T-03 *My tickets* | SCR-01, SCR-T-01, SCR-T-02, SCR-T-03, SCR-91 |
| **Technician** | SCR-T-03 *Assigned to me* | SCR-01, SCR-T-01, SCR-T-02, SCR-T-03, SCR-91 |
| **Admin** | SCR-WF-30 *AI decisions* (temporary, until the F02 Dashboard) | Every page in the sitemap |

**Related:** [Information architecture index](README.md) · [Routes and permissions](02-routes-and-permissions.md) · [Navigation](03-navigation.md)
