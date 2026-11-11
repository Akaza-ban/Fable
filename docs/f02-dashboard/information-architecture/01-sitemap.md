# Sitemap — F02 Dashboard

> Design status: V1 — draft, awaiting user review  
> Implementation status: NOT STARTED  

**Notation (D-IA-DB-01):**
- **Card:** Individual page, drawer, or widget container with ID and core functionality.
- **Area (box):** Logical functional grouping.
- **Solid arrow:** Direct drill-down or navigation flow.
- **Dashed arrow:** Role accessibility linkage.
- **Purple card / accent:** Elements powered by AI telemetry or automated prediction models.

---

## Visual Sitemap

```mermaid
flowchart TB
    admin["<span style='font-size:40px'>👤</span><br/><b>Admin</b>"]:::actor
    techLead["<span style='font-size:40px'>👤</span><br/><b>Technician Lead</b>"]:::actor
    tech["<span style='font-size:40px'>👤</span><br/><b>Technician</b>"]:::actor

    subgraph gateway["Routing Gateway"]
        direction TB
        gate["<b>/dashboard</b><br/>• JWT Role Verification<br/>• Automated Route Dispatcher"]
    end

    subgraph exec["Executive Dashboard (SCR-DB-01) — /admin/dashboard"]
        direction TB
        kpi["<b>KPI Summary Cards</b><br/>• Total, Open, In Progress, Closed<br/>• AI Automation Rate (%)"]:::ai
        charts["<b>Distribution Analytics</b><br/>• Priority breakdown (P1-P4)<br/>• Incident category bar chart<br/>• Hourly volume intake trend"]
        sla["<b>SLA & Bottleneck Watch</b><br/>• Top 10 critical tickets<br/>• Live countdown timers<br/>• Overdue breach badges"]
        workload["<b>Team Workload Matrix</b><br/>• Active load index per team<br/>• Overloaded engineer alerts"]
        alertStream["<b>Live Alert Feed</b><br/>• P1 critical incident banner<br/>• Spike detection alerts"]:::ai
    end

    subgraph techWork["Technician Workbench (SCR-DB-02) — /technician/dashboard"]
        direction TB
        myQueue["<b>My Active Queue</b><br/>• Assigned tickets due today<br/>• Ordered by SLA urgency"]
        unassignedPool["<b>Unassigned Team Pool</b><br/>• 1-click Claim Ticket button<br/>• Team ticket intake"]
        slaWatch["<b>Personal SLA Tracker</b><br/>• Live countdown per ticket"]
        csat["<b>Performance Metrics</b><br/>• Weekly output & CSAT rating"]
    end

    subgraph modals["Modals & Drawers"]
        direction TB
        draw10["<b>SCR-DB-10 Live Incident Drawer</b><br/>• Context telemetry, quick dispatch<br/>• Incident acknowledgement"]:::ai
        modal11["<b>SCR-DB-11 SLA Escalation Modal</b><br/>• Fast reassignment & reminder"]
        modal12["<b>SCR-DB-12 AI Rebalance Modal</b><br/>• Automated redistribution suggestion"]:::ai
    end

    subgraph external["Connected Subsystems"]
        direction TB
        tList["<b>SCR-T-03 Ticket List</b><br/>• Deep drill-down destination<br/>• Filter parameters retained"]
        tDetail["<b>SCR-T-02 Ticket Detail</b><br/>• Action execution target"]
        notif["<b>SCR-91 Notification Center</b>"]
    end

    %% Role connections
    admin -.-> gate
    techLead -.-> gate
    tech -.-> gate

    gate --> exec
    gate --> techWork

    %% Intra-dashboard flows
    alertStream --> draw10
    sla --> modal11
    workload --> modal12

    %% Deep drill-down linkages
    kpi --> tList
    charts --> tList
    sla --> tList
    myQueue --> tDetail
    unassignedPool --> tDetail
    draw10 --> tDetail
    modal11 --> tDetail

    classDef actor fill:none,stroke:none,color:#111
    classDef ai fill:#7b3fbf,color:#fff,stroke:#5a2d8c
```

---

## Screens by Role

| Role | Default Landing Route | Accessible Dashboard Views |
|---|---|---|
| **Admin** | `/admin/dashboard` (`SCR-DB-01`) | Full executive view, all engineering teams, company-wide KPI analytics, live incident broadcast, AI rebalancing proposals. |
| **Technician Lead** | `/technician/dashboard` (`SCR-DB-02`) | Frontline workbench plus team-scoped workload capacity matrix and SLA escalation controls. |
| **Technician** | `/technician/dashboard` (`SCR-DB-02`) | Personal daily task queue, unassigned group intake pool, personal SLA countdowns, individual throughput metrics. |
| **Requester** | `/my-tickets` (`SCR-T-03`) | **Access Denied (`403`)** for Dashboard routes. Redirected to personal ticket tracker. |

**Related:** [Decisions and Defaults](00-decisions.md) · [Routes and Permissions](02-routes-and-permissions.md) · [Screen Inventory](04-screen-inventory.md)

