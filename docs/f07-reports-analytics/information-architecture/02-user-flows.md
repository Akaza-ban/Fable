# User Flows — F07 Reports & Analytics

> Design status: TARGET V1
>
> Related: [IA overview](README.md) · [Screens and navigation](01-screens-and-navigation.md) · [UI states](03-ui-states-and-accessibility.md)

Four flows. Sample people are the same as in the [acceptance criteria](../04-acceptance-criteria.md). Purple: F07. Grey: other branches.

## Flow 1 — Weekly review of the AI

*Thu (Admin) checks on Monday whether the agents are on track for the 50% target.* · US-BC-11, US-BC-04, US-BC-08, US-BC-09

```mermaid
flowchart LR
    a["EML-BC-01<br/>weekly digest"] --> b["Sign in (F01)"]
    b --> c["SCR-BC-03 AI<br/>M-04 vs target"]
    c --> d["M-10 by class<br/>route C reasons"]
    d --> e["SCR-T-04<br/>filtered tickets"]
    e --> f["SCR-WF-03 AI scope (F05)<br/>or SCR-AI-01 (F04)"]

    classDef f07 fill:#f3ecfb,color:#111,stroke:#7b3fbf
    classDef other fill:#f3f3f3,color:#333,stroke:#999
    class a,c,d f07
    class b,e,f other
```

1. Thu receives EML-BC-01: "Closed without a technician: 41% (target 50%), 312 under observation".
2. She clicks the figure → signs in → SCR-BC-03 for last week.
3. The path split shows agent KB answers rising; M-10 shows `AGENT_CANNOT_RESOLVE` with reason "Access not in catalog" at the top.
4. She clicks the reason → SCR-T-04 lists those tickets → she opens a few in SCR-T-02.
5. She widens the Accounting entitlement catalog in F05 SCR-WF-03. Next week's M-04 shows the effect.
6. **Branch:** a new prompt version went live → she filters M-05 by prompt version; corrections went up → she reports it to the F04 owner.

## Flow 2 — Finding an SLA bottleneck

*Thu sees P2 breaches rising.* · US-BC-03, US-BC-01

1. SCR-BC-02 for 30 days → breach rate by priority: P2 resolution target 18%.
2. Breakdown by group → SAP has most breaches.
3. Tab **At risk** → 9 SAP tickets with `SLA_WARNING` → open in SCR-T-02 and escalate through F03 / F05 dialogs.
4. **Branch:** F05 SLA data unavailable → M-03 shows "Unavailable"; M-01 and M-02 are still usable.

## Flow 3 — Team Lead rebalances work

*Quang (lead of SAP) sees one technician overloaded.* · US-BC-10, US-BC-02

```mermaid
flowchart LR
    a["SCR-BC-06 Workload<br/>SAP team"] --> b["Technician row<br/>23 open, oldest 6 days"]
    b --> c["SCR-T-04<br/>filtered on technician"]
    c --> d["Reassign dialog<br/>(F05 SCR-WF-11 / F03 DLG-T-01)"]

    classDef f07 fill:#f3ecfb,color:#111,stroke:#7b3fbf
    classDef other fill:#f3f3f3,color:#333,stroke:#999
    class a,b f07
    class c,d other
```

1. Quang opens Reports → Workload; he sees only the SAP team.
2. One row: 23 open, oldest open 6 days. He opens the filtered ticket list and reassigns three tickets with a reason.
3. **Branch:** he changes the URL to IT Helpdesk → SCR-90.

## Flow 4 — Export for management

*Thu needs the monthly figures in a spreadsheet.* · US-BC-06

1. SCR-BC-01 with last month's filter → *Export* → SCR-BC-10 shows "4,812 rows, XLSX".
2. *Export* → the file downloads; the audit log has `REPORT_EXPORTED`.
3. **Branch:** 60,000 rows → `EXPORT_TOO_LARGE`, the modal asks to narrow the period.
4. **Branch:** audit log unavailable → `AUDIT_UNAVAILABLE`, no file, the modal says to try later.
