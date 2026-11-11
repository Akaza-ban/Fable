# User Flows — F03 Ticket Management

> Design status: TARGET V1
>
> Related: [IA overview](README.md) · [Screens and navigation](01-screens-and-navigation.md) · [Screen content](02-screen-content.md) · [UI states](04-ui-states-and-accessibility.md)

Six flows follow a ticket from creation to a final state. Flows 1–2 are the main path; flows 3–6 are the branches people use most. Blue: an F03 page or dialog. Grey: done by another branch (F04 AI, F05 Workflow). Red: a final state. Statuses are the names in the [use cases](../02-use-cases.md#4-ticket-lifecycle-f03-view).

## Flow 1 — Create a ticket with evidence

*Lan (Sales) cannot open the shared Accounting folder and attaches a screenshot.* · US-TK-01, US-TK-02, US-TK-04 · UC-TK-01

```mermaid
flowchart LR
    a["SCR-T-03<br/>+ New ticket"] --> b["SCR-T-01<br/>title, description"]
    b --> c["Paste screenshot<br/>scan: Clean"]
    c --> d{"Similar own<br/>open ticket?"}
    d -- "Yes: Add my info" --> e["SCR-T-02 of the<br/>existing ticket"]
    d -- "No: Submit" --> f["SCR-T-02 new ticket<br/>NEW, 'assistant reviewing'"]
    f --> g["F05 routing<br/>A / B / C"]

    classDef f03 fill:#e8f1fc,color:#111,stroke:#1168bd
    classDef other fill:#f3f3f3,color:#333,stroke:#999,stroke-dasharray:4 3
    class a,b,c,e,f f03
    class g other
```

1. Lan clicks **+ New ticket** and fills title and description on **SCR-T-01**; category stays *I'm not sure*.
2. She pastes a screenshot; it uploads at once and shows *Scanning…* then *✔ Clean*. *Submit* stays disabled until the scan ends.
3. The suggestions panel shows no matching own ticket; she clicks *Submit* (`createTicket` with `Idempotency-Key`).
4. **SCR-T-02** opens with *"Submitted — the assistant is reviewing your request"*; within seconds F05 shows the routing outcome.
5. **Branches:** a similar own open ticket → *Add my info here* → no new ticket (UC-TK-01 A2); file blocked → removed with *"This file was blocked for security reasons"*; 21st ticket in an hour → `429` banner.

## Flow 2 — Technician handles, resolves; requester confirms

*Minh (Network L1) picks up Lan's VPN ticket.* · US-TK-06, US-TK-11, US-TK-14, US-TK-17, US-TK-18, US-TK-20 · UC-TK-03, UC-TK-05, UC-TK-06

```mermaid
flowchart LR
    a["SCR-T-04<br/>Team unassigned"] --> b["Claim<br/>ASSIGNED"]
    b --> c["SCR-T-02<br/>Start work<br/>IN_PROGRESS"]
    c --> d["Notes and<br/>public replies"]
    d --> e["DLG-T-05 Resolve<br/>RESOLVED"]
    e --> f["Requester: SCR-T-02<br/>'Is it working now?'"]
    f -- "Yes" --> g["DLG-T-08<br/>Confirm and rate"]
    g --> h(["CLOSED"])
    f -- "No reply 5 days" --> h
    f -- "No" --> flow5(["Flow 5"])

    classDef f03 fill:#e8f1fc,color:#111,stroke:#1168bd
    classDef final fill:#fdecea,color:#5a1a14,stroke:#d9534f
    class a,b,c,d,e,f,g f03
    class h,flow5 final
```

1. Minh opens **SCR-T-04** *My team — unassigned*, sorted by SLA due; he clicks *Claim* (a second claimer gets `409 ALREADY_ASSIGNED`).
2. On **SCR-T-02** he reads the AI summary, clicks *Start work* → `IN_PROGRESS`; Lan sees *In progress*.
3. He writes an internal note (🔒, never sent to Lan) and a public reply; Lan is notified by e-mail.
4. He clicks *Resolve* → **DLG-T-05**: code `FIXED`, note *"Updated your VPN client to 5.2"* → `RESOLVED`.
5. Lan gets *"Resolved — please confirm"*; on SCR-T-02 she clicks *Yes, it's solved* → **DLG-T-08**, rates 5 stars → `CLOSED` (`CONFIRMED`).
6. **Branch:** no answer → reminders day 1 and N−1, auto-close on day 5 (`AUTO_CONFIRMED`, BR-TK-12).

## Flow 3 — Technician asks the requester for information

*Minh needs Lan's VPN client version.* · US-TK-16 · UC-TK-04 A1–A3

```mermaid
flowchart LR
    a["SCR-T-02<br/>Ask requester"] --> b["DLG-T-04<br/>question"]
    b --> c["NEEDS_INFO<br/>waiting_on = TECHNICIAN<br/>SLA paused"]
    c --> d["Requester: banner<br/>'needs more information'"]
    d --> e["Public reply"]
    e --> f["IN_PROGRESS<br/>same assignee"]
    c -- "Found it elsewhere" --> g["Resume work<br/>+ reason"]
    g --> f
    c -- "No reply N days" --> h["F05 reminders,<br/>auto-close"]

    classDef f03 fill:#e8f1fc,color:#111,stroke:#1168bd
    classDef other fill:#f3f3f3,color:#333,stroke:#999,stroke-dasharray:4 3
    class a,b,c,d,e,f,g f03
    class h other
```

1. Minh clicks *Ask requester* → **DLG-T-04** with *"SLA will pause until the requester replies"*.
2. The ticket goes to `NEEDS_INFO`; the SLA badge shows *⏸ Paused*; the ticket appears in *Waiting for requester*.
3. Lan sees the banner on SCR-T-02 and in *Needs my action* on SCR-T-03; any public reply or attachment resumes the ticket to `IN_PROGRESS` with Minh as assignee.
4. **Branches:** Minh finds the answer himself → *Resume work* with a reason; no reply → F05 reminders and auto-close (UC-WF-06). An AI question (`waiting_on = AI`) looks the same to Lan, but her answer goes back to F05 routing.

## Flow 4 — Transfer a misrouted ticket, escalate a hard one

*A printer ticket reached Network L1; later a server fault needs L2.* · US-TK-10, US-TK-12, US-TK-13 · UC-TK-03

```mermaid
flowchart LR
    a["SCR-T-02<br/>wrong team"] --> b["DLG-T-02 Transfer<br/>WRONG_TEAM"]
    b --> c["ASSIGNED to target team<br/>no assignee"]
    c --> d["F05 picks a member<br/>or Admin assigns DLG-T-01"]
    b -. "routing correction" .-> ai["F04 learning"]
    e["SCR-T-02<br/>too hard for L1"] --> f["DLG-T-03 Escalate"]
    f -- "Higher tier" --> g["ASSIGNED to L2 team<br/>escalation_level + 1"]
    f -- "Admin" --> h["Same owner<br/>Admins notified"]

    classDef f03 fill:#e8f1fc,color:#111,stroke:#1168bd
    classDef other fill:#f3f3f3,color:#333,stroke:#999,stroke-dasharray:4 3
    class a,b,c,e,f,g,h f03
    class d,ai other
```

1. **Transfer:** Minh clicks *Transfer* → **DLG-T-02**: target team *Hardware*, reason `WRONG_TEAM`, note. The ticket is `ASSIGNED` to Hardware with no individual; F05 picks a member, otherwise it waits in the Hardware queue and Admins are notified. `WRONG_TEAM` is stored as a routing correction for F04. The SLA clock is not reset (BR-TK-06).
2. **Escalate to a higher tier:** **DLG-T-03** proposes only teams with a higher `support_tier` covering the category; *work done so far* (≥ 20 chars) becomes an internal note.
3. **Escalate to Admin:** the owner stays; Admins see the ticket in *Escalated to me* and may reassign (**DLG-T-01**) or raise the priority (**DLG-T-09**).
4. **Exceptions:** same team or no higher tier → `400`, UI suggests *Admin*; ticket `AUTOMATING` → actions hidden (BR-TK-08).

## Flow 5 — Not fixed, reopen, follow-up

*The VPN drops again two days after closure.* · US-TK-21 · UC-TK-06 A2–A5

```mermaid
flowchart LR
    a["RESOLVED:<br/>No, still a problem"] --> c["DLG-T-07<br/>reason + evidence"]
    b["CLOSED ≤ 7 days:<br/>Reopen"] --> c
    c --> d{"Assignee<br/>available?"}
    d -- "Yes" --> e["IN_PROGRESS<br/>same assignee"]
    d -- "No" --> f["ASSIGNED<br/>team queue"]
    c -- "3rd reopen" --> g["Escalated to Admin"]
    h["CLOSED > 7 days"] --> i["Create follow-up<br/>SCR-T-01 pre-filled"]

    classDef f03 fill:#e8f1fc,color:#111,stroke:#1168bd
    class a,b,c,e,f,g,h,i f03
```

1. From `RESOLVED`, Lan clicks *No, still a problem*; from `CLOSED` within 7 days, *Reopen*. Both open **DLG-T-07** (reason ≥ 10 chars, attachments, remaining window shown).
2. The ticket returns to Minh (`IN_PROGRESS`) or, if he is unavailable, to the team queue; `reopen_count` + 1.
3. The 3rd reopen escalates to Admin with *"Reopened 3 times"*.
4. After the window, *Reopen* is replaced by *Create follow-up ticket* → **SCR-T-01** pre-filled with a `FOLLOW_UP_OF` link.

## Flow 6 — Cancel a request

*Lan's problem disappears after a restart; a technician also finds a duplicate.* · US-TK-22 · UC-TK-06 A6–A7

```mermaid
flowchart LR
    a["Requester: NEW … ASSIGNED<br/>Cancel request"] --> b["DLG-T-06<br/>optional note"]
    b --> z(["CANCELLED"])
    c["Requester: IN_PROGRESS<br/>Ask to cancel"] --> d["Public message<br/>to technician"]
    d --> e["Staff: DLG-T-06<br/>reason code + note"]
    f["Staff: duplicate"] --> e
    e --> z

    classDef f03 fill:#e8f1fc,color:#111,stroke:#1168bd
    classDef final fill:#fdecea,color:#5a1a14,stroke:#d9534f
    class a,b,c,d,e,f f03
    class z final
```

1. Before work starts, Lan clicks *Cancel request* → **DLG-T-06** → `CANCELLED` (`REQUESTER_WITHDREW`); the assignee, if any, is notified.
2. Once `IN_PROGRESS`, the button becomes *Ask to cancel*, which posts a public message; the technician cancels with a reason.
3. Staff cancel duplicates and spam with a reason code; `DUPLICATE` needs the original ticket code. The requester is notified with the reason.
4. `AUTOMATING` tickets cannot be cancelled (*"This request is being executed automatically and cannot be cancelled now"*); `CANCELLED` is never reopenable.
