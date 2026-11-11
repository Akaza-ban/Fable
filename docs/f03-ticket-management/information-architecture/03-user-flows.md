# User Flows — F03 Ticket Management

> Design status: TARGET V1
>
> Related: [IA overview](README.md) · [Screens and navigation](01-screens-and-navigation.md) · [Screen content](02-screen-content.md) · [UI states](04-ui-states-and-accessibility.md)

Seven flows, one per leaf of the **Ticket Management** branch of the mind map. Blue: an F03 page, dialog or email. Grey: done by another branch (F02, F04, F05). Red: a final state. Statuses are the names in the [use cases](../02-use-cases.md#4-ticket-lifecycle-f03-view).

| Flow | Mind-map leaf | Use case |
|---|---|---|
| 1 | Raise ticket by email or free text, no category form | UC-TK-01 |
| 2 | View / search / filter tickets | UC-TK-02 |
| 3 | View ticket summary on handoff and approval | UC-TK-03 |
| 4 | Transfer / escalate within the same division | UC-TK-04 |
| 5 | Add missing information to on-hold tickets | UC-TK-05 |
| 6 | Update status and resolution | UC-TK-06 |
| 7 | Confirm / close / reopen / cancel ticket | UC-TK-07 |

## Flow 1 — Raise a ticket by email or free text

*Lan (Sales) cannot open the shared Accounting folder. She emails IT; the next day she uses the portal for another request.* · US-TK-01 … 03 · UC-TK-01

```mermaid
flowchart LR
    a["Email to<br/>it-support@…"] --> b["Mailbox intake<br/>ticket NEW, source EMAIL"]
    c["SCR-T-01<br/>free text, no category"] --> d["createTicket<br/>NEW, source PORTAL"]
    b --> e["EML-T-01<br/>acknowledgement [TK-…]"]
    d --> e
    d --> f["SCR-T-02<br/>'assistant reviewing'"]
    b & d --> g["F05 analysis:<br/>fields Filled by AI,<br/>priority, route"]

    classDef f03 fill:#e8f1fc,color:#111,stroke:#1168bd
    classDef other fill:#f3f3f3,color:#333,stroke:#999,stroke-dasharray:4 3
    class b,c,d,e,f f03
    class a,g other
```

1. Lan writes *"I get Access denied on \\fs01\accounting since this morning"* with a screenshot to the support mailbox. Within 2 minutes the ticket exists (`EMAIL`) and she receives **EML-T-01** with its code.
2. F05 fills request type, category, support level and priority (*"Filled by AI"*) and routes it. Lan never picked a category.
3. Next day, on **SCR-T-01**, she types her request in the description box only, pastes a screenshot (*✔ Clean*), and clicks *Submit*; **SCR-T-02** shows *"Submitted — the assistant is reviewing your request"*, then the outcome.
4. **Branches:** she replies to EML-T-01 → the reply is added to the same ticket (no new ticket); she emails the same request twice → F05 merges it (*Not the same problem* within 7 days); an auto-reply → ignored; file blocked → *"This file was blocked for security reasons"*; 21st ticket in an hour → `429` (portal) or added to her latest open ticket (email).

## Flow 2 — Find a ticket

*Minh (IT Helpdesk › Network) starts his day; Lan checks her requests.* · US-TK-05 … 09 · UC-TK-02

```mermaid
flowchart LR
    a["F02 workbench"] --> b["SCR-T-04<br/>Assigned to me"]
    b --> c["Filters: source EMAIL,<br/>SLA at risk, P1–P2"]
    c --> d["Save view"]
    c --> e["SCR-T-02"]
    f["SCR-T-03<br/>Needs my action"] --> e

    classDef f03 fill:#e8f1fc,color:#111,stroke:#1168bd
    classDef other fill:#f3f3f3,color:#333,stroke:#999,stroke-dasharray:4 3
    class b,c,d,e,f f03
    class a other
```

1. Minh opens **SCR-T-04** *Assigned to me*, sorted by SLA due; he filters *Due soon* and *P1–P2* and saves the view.
2. He searches `TK-2026-004871` and gets that ticket only. A ticket outside his sub-groups → `/403`.
3. Lan opens **SCR-T-03** *Needs my action*: one ticket *Waiting for your reply*, one *Resolved — please confirm*.

## Flow 3 — Read the summary on handoff and on approval

*The Helpdesk AI agent could not fix Lan's VPN; F05 hands it to Minh. Later, an access request outside the catalog waits for the Admin's approval.* · US-TK-10 … 12 · UC-TK-03

```mermaid
flowchart LR
    a["Handoff: F05 assigns,<br/>re-route, agent escalation,<br/>transfer, reopen"] --> b["AI Service writes<br/>summary v1"]
    b --> c["Notification<br/>with summary"]
    c --> d["SCR-T-02<br/>SEC-T-01 on top"]
    d --> e["Click fact →<br/>source message"]
    f["New requester message"] --> g["Summary v2<br/>'Updated after a new message'"]
    g --> d
    h["Ticket waits<br/>for approval"] --> i["Summary + Requested action<br/>+ Why approval is needed"]
    i --> j["Approver decides<br/>(owner per Q-TK-10)"]

    classDef f03 fill:#e8f1fc,color:#111,stroke:#1168bd
    classDef other fill:#f3f3f3,color:#333,stroke:#999,stroke-dasharray:4 3
    class b,c,d,e,g,i f03
    class a,f,h,j other
```

1. F05 assigns the ticket to Minh. The notification already contains the summary: *Problem — VPN disconnects every ~60 min · Already tried — AI guided reinstall, not fixed · Why it came to you — the agent could not resolve*.
2. On **SCR-T-02**, **SEC-T-01** is on top; Minh clicks *"not fixed"* and the conversation scrolls to Lan's answer to the agent.
3. Lan adds a log file; within 30 s the summary becomes v2 *"Updated after a new message"*; v1 stays viewable.
4. **Approval:** a ticket waiting for a human approval shows the summary with *Requested action* (grant `FI-Admin` on SAP for 30 days to Lan) and *Why approval is needed* (*"FI-Admin is not in Sales' catalog"*); the approver decides with this context (decision owned outside F03, Q-TK-10).
5. **Branches:** summary not ready → *"Summary is being prepared"*; AI down → *"Summary unavailable"* + the original description; Lan never sees the summary.

## Flow 4 — Transfer or escalate within the division

*Minh's printer ticket needs the Printers sub-group; a hard network fault needs L2.* · US-TK-13, 14 · UC-TK-04

```mermaid
flowchart LR
    a["SCR-T-02<br/>IT Helpdesk › Network"] --> b["DLG-T-02 Transfer<br/>within IT Helpdesk"]
    b --> c["ASSIGNED to<br/>IT Helpdesk › Printers"]
    c --> d["F05 picks a member<br/>(ON, lowest load)"]
    d --> s["New owner gets<br/>the summary"]
    b -. "other group?" .-> r["F05 SCR-WF-24<br/>Return for re-routing"]
    e["SCR-T-02<br/>too hard for L1"] --> f["DLG-T-03 Escalate"]
    f -- "Higher tier in group" --> g["ASSIGNED to L2 sub-group<br/>escalation_level + 1"]
    f -- "Admin" --> h["Same owner<br/>Admins notified + summary"]

    classDef f03 fill:#e8f1fc,color:#111,stroke:#1168bd
    classDef other fill:#f3f3f3,color:#333,stroke:#999,stroke-dasharray:4 3
    class a,b,c,e,f,g,h,s f03
    class d,r other
```

1. **Transfer:** Minh clicks *Transfer* → **DLG-T-02** shows only sub-groups and technicians of *IT Helpdesk*; he picks *Printers*, reason `NEEDS_OTHER_SKILL`, note. F05 picks a member; the new owner is notified with the summary. The SLA is not reset.
2. **Wrong group:** the problem is an SAP authorization, not IT Helpdesk → the dialog's link opens F05 **SCR-WF-24** *Return for re-routing*; F05 routes it again.
3. **Escalate to a higher tier:** **DLG-T-03** lists only higher-tier sub-groups of IT Helpdesk; *work done so far* becomes an internal note.
4. **Escalate to Admin:** the owner stays; Admins get the ticket in *Escalated to me* with the summary.
5. **Exceptions:** target in another group → `400 OUTSIDE_DIVISION`; no higher tier → *"Escalate to Admin instead"*; ticket owned by the AI agent or `AUTOMATING` → actions hidden.

## Flow 5 — Add missing information to an on-hold ticket

*The AI needs to know which folder; later Minh needs Lan's VPN version.* · US-TK-15 … 18 · UC-TK-05

```mermaid
flowchart LR
    a["F05 asks<br/>(waiting_on = AI)"] --> c["NEEDS_INFO<br/>SLA paused"]
    b["DLG-T-04 Ask requester<br/>(waiting_on = TECHNICIAN)"] --> c
    c --> d["EML-T-02 +<br/>SEC-T-02 banner"]
    d --> e["Lan replies:<br/>portal or email"]
    e -- "AI" --> f["F05 re-routes<br/>NEW"]
    e -- "TECHNICIAN" --> g["IN_PROGRESS<br/>same assignee"]
    c -- "Found it elsewhere" --> h["Resume work<br/>+ reason"]
    h --> g
    c -- "No reply N days" --> i["F05 reminders,<br/>auto-close"]

    classDef f03 fill:#e8f1fc,color:#111,stroke:#1168bd
    classDef other fill:#f3f3f3,color:#333,stroke:#999,stroke-dasharray:4 3
    class b,c,d,e,g,h f03
    class a,f,i other
```

1. F05 is not confident about the folder and asks; the ticket is on hold. Lan gets **EML-T-02** and simply **replies to the email**: *"\\fs01\accounting\2026"*. The reply is threaded, passed to F05 and the ticket is routed again.
2. Later Minh clicks *Ask requester* → **DLG-T-04** (*"The ticket goes on hold and the SLA pauses"*). Lan sees **SEC-T-02** on SCR-T-02 and answers there with a screenshot; the ticket returns to `IN_PROGRESS` with Minh, the SLA resumes, the summary refreshes.
3. **Branches:** Minh finds the answer in AD → *Resume work* with a reason; no reply → F05 reminders day 1 and N−1, auto-close day N (Lan can reopen within 7 days); a reply from an unknown address → held for staff review, the ticket stays on hold.

## Flow 6 — Update status and resolution

*Minh fixes the VPN.* · US-TK-19 … 21 · UC-TK-06

```mermaid
flowchart LR
    a["SCR-T-02<br/>ASSIGNED"] --> b["Start work<br/>IN_PROGRESS"]
    b --> c["Replies, notes,<br/>on hold (Flow 5)"]
    c --> d["DLG-T-05 Resolve<br/>F04 drafts Resolution + reply"]
    d --> e["RESOLVED<br/>EML-T-03 sent"]
    f["F04 resolves<br/>(KB, guided, API)"] --> e
    d -. "related tickets" .-> g["Resolve related too"]

    classDef f03 fill:#e8f1fc,color:#111,stroke:#1168bd
    classDef other fill:#f3f3f3,color:#333,stroke:#999,stroke-dasharray:4 3
    class a,b,c,d,e,g f03
    class f other
```

1. Minh clicks *Start work* → `IN_PROGRESS`; Lan sees *In progress*.
2. He clicks *Resolve* → **DLG-T-05** opens with the Resolution (*"Root cause: client 4.8 timeout bug; updated to 5.2"*) and the reply drafted by F04; he edits and confirms with code `FIXED`.
3. The ticket is `RESOLVED`; Lan gets **EML-T-03** with *Confirm / Not fixed*.
4. **Branches:** duplicate → code `DUPLICATE` + original code; related tickets in SEC-T-03 → *Resolve the related tickets too*, each with its own reply; `AUTOMATING` → no *Resolve*, F04 verifies; when the AI agent resolves a ticket, F04 uses the same operation with author `AI` and `ANSWERED_BY_KB` / `SOLVED_BY_KB` / `GUIDED_BY_AI`.

## Flow 7 — Confirm, close, reopen, cancel

*Lan confirms; two days later the VPN drops again; a test ticket is cancelled.* · US-TK-22 … 24 · UC-TK-07

```mermaid
flowchart LR
    a["EML-T-03 or SCR-T-02<br/>'Is it working now?'"] -- "Yes" --> b["DLG-T-08<br/>confirm + rate"]
    b --> c(["CLOSED"])
    a -- "No reply 5 days" --> c
    a -- "No" --> d["DLG-T-07<br/>reason + evidence"]
    c -- "≤ 7 days: Reopen" --> d
    d --> e{"Assignee ON?"}
    e -- "Yes" --> f["IN_PROGRESS<br/>same assignee"]
    e -- "No" --> g["ASSIGNED<br/>F05 picks a member"]
    d -- "3rd reopen" --> h["Escalated to Admin"]
    c -- "> 7 days" --> i["Create follow-up<br/>SCR-T-01"]
    j["NEW … ASSIGNED:<br/>Cancel request"] --> k["DLG-T-06"]
    k --> z(["CANCELLED"])

    classDef f03 fill:#e8f1fc,color:#111,stroke:#1168bd
    classDef final fill:#fdecea,color:#5a1a14,stroke:#d9534f
    class a,b,d,f,g,h,i,j,k f03
    class c,z final
```

1. Lan clicks *Yes, it's solved* in **EML-T-03**; SCR-T-02 opens **DLG-T-08**; she rates 5 stars → `CLOSED` (`CONFIRMED`). No answer → reminders and auto-close on day 5 (`AUTO_CONFIRMED`).
2. Two days later she clicks *Reopen* → **DLG-T-07**; the ticket returns to Minh, who receives the summary again; `reopen_count` + 1. The 3rd reopen escalates to Admin. After 7 days → *Create follow-up ticket*.
3. Before work starts, Lan cancels a request she no longer needs → **DLG-T-06** → `CANCELLED`. Once `IN_PROGRESS`, she can only *Ask to cancel*; staff cancel test or spam tickets with a reason code. `AUTOMATING` tickets cannot be cancelled.
