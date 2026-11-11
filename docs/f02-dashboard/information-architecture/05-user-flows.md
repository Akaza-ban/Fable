# Primary User Flows — F02 Dashboard

> Design status: V1 — draft, awaiting user review  
> Implementation status: NOT STARTED  

Five primary operational flows define user interactions across the Dashboard subsystem. Steps performed by human actors begin with 👤 (**AD** Admin, **TL** Team Lead, **TE** Technician); automated steps executed by the system or AI are highlighted in **purple**.

---

## Flow 1: Executive KPI Monitoring & Deep Metric Drill-Down

*Covers:* `US-DB-01`, `US-DB-04`  
*Actor:* Administrator (`AD`)  
*Scenario:* The Admin evaluates daily throughput, notices a spike in P1 tickets, and drills down to the underlying ticket records.

```mermaid
flowchart TD
    actAdmin["<span style='font-size:40px'>👤</span><br/><b>Admin</b>"]:::actor
    actAdmin --> open["👤 AD: Navigate to /admin/dashboard"]
    open --> load["SCR-DB-01 Executive Dashboard<br/>Render KPI cards, charts, and SLA radar"]
    load --> filter{"👤 AD: Adjust date range<br/>or select team?"}
    filter -->|"Yes"| applyFilter["Update query params<br/>Asynchronously reload cached metrics"]
    applyFilter --> load
    filter -->|"No"| inspect["👤 AD: Inspect priority breakdown chart<br/>Notices 14 P1 Critical Tickets"]
    inspect --> drill["👤 AD: Click on 'P1 Critical' chart slice"]
    drill --> nav["System navigates to SCR-T-03 Ticket List<br/>URL: /tickets?priority=P1&range=today"]
    nav --> inspectList["👤 AD: Inspect filtered ticket backlog"]

    classDef actor fill:none,stroke:none,color:#111
    classDef ai fill:#7b3fbf,color:#fff,stroke:#5a2d8c
```

---

## Flow 2: Proactive SLA Breach Radar & Rapid Escalation

*Covers:* `US-DB-03`  
*Actor:* Team Lead (`TL`), Administrator (`AD`)  
*Scenario:* A high-priority ticket approaches its SLA resolution deadline; the Team Lead intervenes to prevent breach.

```mermaid
flowchart TD
    actLead["<span style='font-size:40px'>👤</span><br/><b>Team Lead</b>"]:::actor
    start(["SLA Monitoring Widget"]) --> scan["System: Detect ticket #TK-985 remaining time ≤ 20%"]
    scan --> yellow["Render Yellow Warning Badge<br/>Countdown: 00:14:52"]
    actLead --> see["👤 TL: Notice blinking yellow warning on radar"]
    see --> click["👤 TL: Click on ticket #TK-985"]
    click --> modal["Open SCR-DB-11 SLA Escalation Modal<br/>Display current assignee & capacity matrix"]
    modal --> decide{"👤 TL: Action choice?"}
    decide -->|"Nudge Assignee"| nudge["Dispatch instant reminder notification (SCR-91)"]
    decide -->|"Reassign"| reassign["Select available engineer from dropdown<br/>Enter required reassignment reason"]
    reassign --> update["Update ticket assignee → Status: IN_PROGRESS<br/>Log audit record AUDIT_SLA_REASSIGN"]
    nudge --> close["Modal closes; SLA radar counter updates"]
    update --> close

    classDef actor fill:none,stroke:none,color:#111
```

---

## Flow 3: Team Workload Monitoring & AI-Assisted Rebalancing

*Covers:* `US-DB-05`, `US-DB-06`  
*Actor:* Team Lead (`TL`), AI Service  
*Scenario:* An engineer becomes overloaded due to incoming urgent incidents; AI computes a load-balancing proposal.

```mermaid
flowchart TD
    actLead["<span style='font-size:40px'>👤</span><br/><b>Team Lead</b>"]:::actor
    metric["System calculates Workload Index:<br/>Active Tickets + (P1/P2 × 2)"] --> check{"Workload Index > 7<br/>or ≥ 2 P1/P2?"}
    check -->|"Yes"| overload["Render Red Badge: Overloaded (9/7)<br/>Engineer Alice"]
    overload --> ai["AI Service: Detect team capacity imbalance<br/>Find engineer Bob with Load: 2/7"]:::ai
    ai --> suggest["Generate Rebalancing Proposal:<br/>Transfer #TK-1082 from Alice to Bob"]:::ai
    actLead --> clickOverload["👤 TL: Click 'Overloaded' badge on Alice"]
    clickOverload --> openRebalance["Open SCR-DB-12 AI Rebalance Modal"]
    openRebalance --> review["👤 TL: Review proposal & confidence score"]
    review --> accept{"👤 TL: Decision?"}
    accept -->|"Apply"| apply["Execute transfer of #TK-1082<br/>Notify Bob & Alice via SCR-91"]
    accept -->|"Dismiss"| dismiss["Dismiss recommendation<br/>Log feedback for AI tuning"]
    apply --> done["Workload meters recalculate<br/>Alice: 7/7 (Busy) | Bob: 3/7 (Normal)"]
    dismiss --> done

    classDef actor fill:none,stroke:none,color:#111
    classDef ai fill:#7b3fbf,color:#fff,stroke:#5a2d8c
```

---

## Flow 4: Real-Time Critical P1 Incident Broadcast & Triage

*Covers:* `US-DB-07`  
*Actor:* On-Duty Engineer (`TE`), Administrator (`AD`)  
*Scenario:* A critical data center network switch fails, generating an instant P1 incident broadcast.

```mermaid
flowchart TD
    incident(["P1 Incident Ingested into Database"]) --> publish["Redis Pub/Sub publishes INCIDENT_P1_CREATED"]
    publish --> ws["WebSocket Gateway pushes event to /dashboard/live-feed"]
    ws --> client["Connected Dashboard Clients"]
    client --> audio["Play optional urgent audio chime"]
    client --> banner["Mount persistent flashing red banner at viewport top:<br/>'🚨 P1 Critical: Core Switch Failure in DC-01'"]
    
    actTech["<span style='font-size:40px'>👤</span><br/><b>On-Duty Engineer</b>"]:::actor
    actTech --> clickBanner["👤 TE: Click on P1 Alert Banner"]
    clickBanner --> drawer["Open SCR-DB-10 Live Incident Drawer<br/>Display AI root cause summary & grouped tickets"]:::ai
    drawer --> ack["👤 TE: Click 'Acknowledge Incident (ACK)'"]
    ack --> api["POST /api/v1/dashboard/alerts/:id/ack"]
    api --> quiet["Banner stops flashing<br/>Status transitions to ACKNOWLEDGED"]
    quiet --> dispatch["👤 TE: Click 'Open Incident War Room' / Dispatch"]

    classDef actor fill:none,stroke:none,color:#111
    classDef ai fill:#7b3fbf,color:#fff,stroke:#5a2d8c
```

---

## Flow 5: Technician Daily Workbench & 1-Click Ticket Claim

*Covers:* `US-DB-08`  
*Actor:* Support Technician (`TE`)  
*Scenario:* A technician logs in for their shift, reviews personal priorities, and claims an unassigned ticket from the group queue.

```mermaid
flowchart TD
    actTech["<span style='font-size:40px'>👤</span><br/><b>Technician</b>"]:::actor
    actTech --> login["👤 TE: Log in to application"]
    login --> redirect["Routing Gateway redirects to /technician/dashboard"]
    redirect --> bench["Render SCR-DB-02 Technician Workbench"]
    bench --> personal["Review 'My Active Queue'<br/>Items ordered by nearest SLA deadline"]
    bench --> pool["Review 'Unassigned Team Pool'<br/>Items waiting for pickup in team group"]
    actTech --> claim["👤 TE: Click 'Claim Ticket' on #TK-1011"]
    claim --> post["POST /api/v1/dashboard/tickets/TK-1011/claim"]
    post --> assign["Ticket assigned to TE → Status: IN_PROGRESS"]
    assign --> refresh["Ticket moves from Pool to 'My Active Queue'<br/>Personal capacity counter increments"]
    refresh --> start["👤 TE: Click 'Start Working' to open SCR-T-02"]

    classDef actor fill:none,stroke:none,color:#111
```

**Related:** [Screen Inventory](04-screen-inventory.md) · [UI States](06-ui-states-and-accessibility.md)

