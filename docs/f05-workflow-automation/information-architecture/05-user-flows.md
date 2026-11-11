# Primary User Flows

> Design status: V1 — draft, awaiting user review
>
> Implementation status: NOT STARTED

Six flows cover a ticket's path through the three processing flows (A — AI self-service, B — clarification, C — human handling), automation failure handling, and two Admin configuration flows. In every flow a **purple** step is performed by the AI, and every AI step has a failure path that lets a person continue. Each flow starts from its actors (person icon). A step performed by a person begins with 👤 and an abbreviation: **RQ** Requester, **TE** Technician, **AD** Admin. Steps without a prefix are done by the system or the AI. State names are the exact names in the [use cases](../02-use-cases.md#related-states).

## Flow 1: Basic request — AI self-service (flow A)

Example: Lan (Accounting) asks for read access to the shared Accounting folder. This entitlement is in the Accounting catalog.

```mermaid
flowchart TD
    actReq["<span style='font-size:40px'>👤</span><br/><b>Requester</b>"]:::actor
    actReq --> create["👤 RQ: SCR-T-01 Create ticket<br/>'Please give me read access to the shared Accounting folder'"]
    create --> detail["SCR-T-02 Ticket detail<br/>status: The AI is analyzing…"]
    detail --> classify["AI: classify + extract<br/>beneficiary, system, entitlement, duration"]:::ai
    classify --> conf{"Confidence ≥ threshold<br/>and information complete?"}
    conf -->|"No"| flowB(["→ Flow 2: clarification"])
    conf -->|"Yes"| human{"Needs a human?"}
    human -->|"Yes"| flowC(["→ Flow 3"])
    human -->|"No"| policy{"Deterministic policy check<br/>(no AI)"}
    policy -->|"Fail"| flowC
    policy -->|"Already has access"| already(["RESOLVED<br/>'You already have this access'"])
    policy -->|"Pass"| approved["AI approves — AUTO_APPROVED<br/>SCR-T-02: progress 'Approved'<br/>notification SCR-91"]:::ai
    approved --> kind{"Operation"}
    kind -->|"CHECK_ACCOUNT"| check["Call the check API<br/>result shown in SCR-T-02"]
    kind -->|"CREATE_ACCOUNT / GRANT_ACCESS"| exec["AUTOMATING: call the API<br/>SCR-T-02: 'Executing'"]
    exec -->|"Transient error"| retry["Automatic retry 1 / 5 / 15 min"]
    retry --> exec
    exec -->|"Permanent error / retries exhausted"| flowD(["→ Flow 4"])
    exec -->|"Success"| verify{"Verify on the<br/>external system"}
    verify -->|"Mismatch"| flowD
    verify -->|"Match"| resolved["RESOLVED — SCR-T-02 'Awaiting confirmation'<br/>notification SCR-91"]
    check --> resolved
    resolved --> confirm{"👤 RQ: result?"}
    confirm -->|"Confirm resolved"| closed(["CLOSED"])
    confirm -->|"SCR-WF-22 Not fixed<br/>(reason required)"| flowC

    classDef actor fill:none,stroke:none,color:#111
    classDef ai fill:#7b3fbf,color:#fff,stroke:#5a2d8c
```

## Flow 2: AI not confident — ask the requester (flow B)

Example: Minh writes *"Please give me access to the system"* without saying which system.

```mermaid
flowchart TD
    actReq["<span style='font-size:40px'>👤</span><br/><b>Requester</b>"]:::actor
    start(["Ticket NEW<br/>low confidence or missing field"]) --> ask["AI: send a specific question<br/>'Which system do you need access to?'"]:::ai
    ask --> wait["NEEDS_INFO — SLA paused<br/>'waiting for my answer' badge + notification SCR-91"]
    actReq --> open["👤 RQ: open SCR-T-02<br/>from the notification or My tickets"]
    wait --> open
    open --> answer["👤 RQ: answer in the conversation panel"]
    answer --> reroute["AI: re-analyze"]:::ai
    reroute --> ok{"Passing now?"}
    ok -->|"Yes"| flowA(["→ Flow 1 (policy check)"])
    ok -->|"No, round < 2"| ask
    ok -->|"No, 2 rounds used"| flowC(["→ Flow 3<br/>reason MAX_CLARIFICATION_REACHED"])
    wait -->|"No answer: remind on day 1, N−1"| remind["Reminder notification SCR-91"]
    remind -->|"Day N"| autoclose(["CLOSED — 'no response'<br/>reopenable within 7 days"])

    classDef actor fill:none,stroke:none,color:#111
    classDef ai fill:#7b3fbf,color:#fff,stroke:#5a2d8c
```

## Flow 3: Request that needs a human (flow C)

Example: Lan reports *"The left button of my mouse doesn't work"*. In the Hardware team, Mai holds 3 P1 Complex tickets (score 18) and Hung holds 5 P3 Simple tickets (score 5); Nam is *OFF* (in a meeting). The ticket goes to Hung.

```mermaid
flowchart TD
    actTech["<span style='font-size:40px'>👤</span><br/><b>Technician</b>"]:::actor
    actAdmin["<span style='font-size:40px'>👤</span><br/><b>Admin</b>"]:::actor
    actReq["<span style='font-size:40px'>👤</span><br/><b>Requester</b>"]:::actor
    start(["ROUTE_TO_HUMAN<br/>hardware / out of scope / policy check failed<br/>/ clarification limit / AI failure"]) --> summary["AI: summarize the ticket + routing reason"]:::ai
    summary --> rules{"A group owns the domain?<br/>sub-group for the subcategory? (F09)"}
    rules -->|"Yes"| team["Sub-group, or group if none<br/>e.g. HELPDESK/PRINTER → Printer sub-group"]
    rules -->|"No group / not classified"| deflt["Default support team"]
    team --> select
    deflt --> select{"Eligible technician?<br/>ON · below max load"}
    select -->|"Yes"| assign["ASSIGNED to the lowest workload score<br/>notification SCR-91 with summary"]
    select -->|"No"| queue["Waits in the team queue"]
    queue -->|"A technician becomes eligible"| select
    queue -->|"Waiting > 30 min"| alert["Admin notified SCR-91"]
    actAdmin --> alert
    alert --> manual["👤 AD: SCR-WF-11 Reassign<br/>(last resort, reason required)"]
    manual --> assign
    actTech --> work
    assign --> work["👤 TE: SCR-T-02<br/>read the AI summary, do the work (e.g. replace the mouse)"]
    work -->|"SLA threshold reached"| warn["SLA warning SCR-91"]
    warn --> work
    work -->|"👤 TE switches OFF<br/>ticket not started"| select
    work --> done["👤 TE: record the result → RESOLVED"]
    actReq --> confirm
    done --> confirm{"👤 RQ: result?"}
    confirm -->|"Confirm resolved"| closed(["CLOSED"])
    confirm -->|"SCR-WF-22 Not fixed"| work

    classDef actor fill:none,stroke:none,color:#111
    classDef ai fill:#7b3fbf,color:#fff,stroke:#5a2d8c
```

## Flow 4: Automation failure — technician handles it

```mermaid
flowchart TD
    actTech["<span style='font-size:40px'>👤</span><br/><b>Technician</b>"]:::actor
    start(["workflow_run FAILED<br/>or verification mismatch"]) --> assign["Assigned to a technician of the owning team<br/>notification SCR-91 'Automation failed'"]
    actTech --> tab
    assign --> tab["👤 TE: SCR-T-02 Automation tab<br/>error code, failed step, attempt history"]
    tab --> choose{"👤 TE: choose"}
    choose -->|"SCR-WF-20 Retry"| recheck{"Current policy<br/>still allows it?"}
    recheck -->|"No"| conflict["409 — 'The policy has changed,<br/>cannot retry'"]
    conflict --> choose
    recheck -->|"Yes"| run["Call the API again, same idempotency key"]
    run -->|"Success + verification OK"| resolved(["RESOLVED — awaiting requester confirmation"])
    run -->|"Fails again"| tab
    choose -->|"SCR-WF-21 Resolved manually<br/>(note required)"| manual(["MANUALLY_RESOLVED<br/>ticket RESOLVED"])

    classDef actor fill:none,stroke:none,color:#111
```

## Flow 5: Admin configures the AI scope and reviews decisions

Example: Thao (Admin) adds the "View receivables report" entitlement for Accounting, after reviewing why a request was routed to a technician.

```mermaid
flowchart TD
    actAdmin["<span style='font-size:40px'>👤</span><br/><b>Admin</b>"]:::actor
    actAdmin --> home["👤 AD: SCR-WF-30 AI decisions<br/>self-resolution, clarification, human-routing rates"]
    home --> filter["👤 AD: filter flow 'Human'<br/>type 'Access request'"]
    filter --> detail["👤 AD: SCR-WF-31 Decision detail<br/>✗ 'Entitlement View receivables report<br/>is not in the Accounting catalog'"]
    detail --> decide{"👤 AD: should the AI<br/>grant this on its own?"}
    decide -->|"No"| keep(["Keep as is — still routed to technicians"])
    decide -->|"Yes"| scope["👤 AD: SCR-WF-03 Entitlement catalog tab<br/>choose Accounting"]
    scope --> form["👤 AD: SCR-WF-07 add the entitlement<br/>ERP system, maximum 90 days"]
    form --> saved(["Saved — applies immediately to<br/>new decisions (AC-UC-WF-03)"])
    actAdmin --> tune["👤 AD: SCR-WF-03 AI actions tab<br/>adjust the confidence threshold / clarification rounds"]
    tune --> saved2(["Saved, audited"])
    actAdmin -->|"Notification 'AI unavailable'"| home

    classDef actor fill:none,stroke:none,color:#111
```

## Flow 6: Admin configures workload scoring

There are no assignment rules: the **team** comes from the AI category and the categories each team is responsible for (set up in F09). The Admin only configures how the **person** inside the team is chosen, and can preview the result.

```mermaid
flowchart TD
    actAdmin["<span style='font-size:40px'>👤</span><br/><b>Admin</b>"]:::actor
    actAdmin --> page["👤 AD: SCR-WF-01 Workload settings"]
    page --> edit["👤 AD: edit e.g. request type 'Server' = Complex ×2<br/>maximum load 20, default support team"]
    edit --> valid{"Valid?"}
    valid -->|"Value ≤ 0 / no default team"| err["400 — entered data kept"]
    err --> edit
    valid -->|"Yes"| saved["Saved, audited — applies<br/>from the next assignment"]
    saved --> preview["👤 AD: SCR-WF-05 Assignment preview<br/>sample ticket 'Printer paper jam'<br/>→ AI category, team, members' status and workload, expected technician"]
    preview --> result{"Result as expected?"}
    result -->|"No"| edit
    result -->|"Yes"| done(["Done"])

    classDef actor fill:none,stroke:none,color:#111
```

**Related:** [Information architecture index](README.md) · [Navigation](03-navigation.md) · [UI states](06-ui-states-and-accessibility.md)
