# Actors, Roles, Use Cases — Workflow & Automation

> Design status: TARGET V1
>
> Scope: F05 Workflow & Automation (UC-WF-01 … UC-WF-09)

## Actors / roles

| Role code | Display name | Workflow responsibilities |
|---|---|---|
| `ADMIN` | Administrator | Configures workload scoring, SLA, and the AI processing scope (AI actions, whitelist, entitlement catalog); reviews AI decisions |
| `TECHNICIAN` | Technician | Handles requests that need a human and automation failures; keeps their **availability status** up to date |
| `REQUESTER` | Requester | Submits requests, answers AI questions, confirms results |

| System actor | Audit code | Responsibilities |
|---|---|---|
| AI Service | `AI` | Classifies tickets with a confidence score, extracts information, asks for clarification when not confident, **approves basic in-scope requests**, summarizes tickets for technicians |

> The system has 3 user roles and **exactly 1 system actor: the AI Service**.
>
> Automated tasks (choosing the team and technician, SLA timers, API execution, verification) are **internal system behavior**, not actors; they are logged with source `AUTOMATION` (BR-WF-05). The external access-management system (AD, Google Workspace, mock API…) is an **integration detail** reached through connectors.
>
> **There is no Team Lead.** Nobody distributes work by hand: the system assigns tickets from each technician's availability and workload (see *Assignment by availability and workload*). The Admin is the only person who can reassign manually, as a last resort.

## Routing principles

About **8,000 tickets per day** are routed by the AI Service into three flows:

| Flow | Condition | Handling |
|---|---|---|
| **A — AI self-service** | Confidence ≥ threshold, request type is in the whitelist, **policy check passes** | AI approves → system calls the API → verification → requester confirms |
| **B — Clarification** | Confidence < threshold, or required information is missing | AI asks the requester → requester answers → re-route (max 2 rounds) |
| **C — Human handling** | Needs a human (hardware…), outside the whitelist, policy check fails, or still unclear after 2 rounds | Automatic technician assignment with an AI summary |

**AI approval has two steps** (BR-WF-02):

1. **AI understands the request:** classifies it and extracts the beneficiary, target system, entitlement / account type and duration.
2. **Deterministic policy check (no AI involved):** the action is in the whitelist, the entitlement is in the beneficiary's department catalog, the duration does not exceed the maximum, and the beneficiary is valid. Pass → approve; fail → flow C.

As a result, even if the AI misunderstands a request, the system cannot grant access outside the scope configured by the Admin.

```mermaid
flowchart TD
    t["New ticket (~8,000/day)"] --> ai["AI Service classifies<br/>+ extracts information"]
    ai --> conf{"Confidence ≥ threshold<br/>and information complete?"}
    conf -- "No" --> ask{"Clarification rounds<br/>< 2?"}
    ask -- "Yes" --> b["Flow B: AI asks the requester"]
    b --> ai
    ask -- "No" --> c
    conf -- "Yes" --> human{"Needs a human?<br/>(hardware, outside whitelist)"}
    human -- "Yes" --> c["Flow C: assign a technician"]
    human -- "No" --> policy{"Policy check<br/>(entitlements, duration, beneficiary)"}
    policy -- "Fail" --> c
    policy -- "Pass" --> a["Flow A: AI approves"]
    a --> exec["Execute via API"]
    exec --> verify{"Verification OK?"}
    verify -- "Yes" --> done["Requester confirms → closed"]
    verify -- "No" --> c

    classDef auto fill:#e8f1fc,color:#123,stroke:#3a7bd5
    classDef human fill:#fdecea,color:#5a1a14,stroke:#d9534f
    classDef askc fill:#fff6e0,color:#5a4300,stroke:#e0a800
    class a,exec,done auto
    class c human
    class b askc
```

## Assignment by availability and workload

A ticket in flow C is assigned by **deterministic rules**, not by the AI and not by a person. The AI only classifies the ticket; the rules below decide who gets it (BR-WF-09, BR-WF-10, BR-WF-18).

### Which team: AI category → responsible team

There are **no assignment rules to configure**. The AI Service gives every ticket a **category** (Hardware, Network, Software, Account & access…). Each technical team declares the categories it is **responsible for** when the team is set up in F09 User Management (e.g. Hardware Team → Hardware, Printer). The ticket goes to the team responsible for its category. If no team is responsible for the category, or the AI could not classify the ticket, it goes to the **default support team** (configured in UC-WF-01).

### Technician availability status

There are only **two statuses**:

| Status | Meaning | Receives new tickets? |
|---|---|---|
| `ON` | On shift and taking work | Yes |
| `OFF` | Not taking work: off shift, on leave, sick, or away (meeting, on site) | No; their not-started tickets are moved to colleagues, tickets in progress stay with them |

The status is set by the technician (top bar ON/OFF switch) and automatically by the system: `OFF` outside working hours of the business calendar and on registered leave, `ON` at the start of the shift. A technician who steps away for a while simply switches to `OFF`. The status itself is owned by F09 User Management; F05 reads it.

### Workload score

Counting tickets is not enough: three high-priority, complex tickets weigh more than five small ones. Each open ticket gets a **load score = priority points × complexity factor**; a technician's workload is the sum over their open tickets (`ASSIGNED`, `IN_PROGRESS`).

The system has **three priority levels** (there is no "critical" level):

| Priority | Points |
|---|---|
| P1 — High | 3 |
| P2 — Medium | 2 |
| P3 — Low | 1 |

| Complexity | Factor | Examples |
|---|---|---|
| Simple | ×1 | Replace a mouse, install a printer |
| Medium | ×1.5 | Reinstall software, network issue on one PC |
| Complex | ×2 | Server failure, incident affecting many users |

In V1 the complexity comes from the **request type**, configured by the Admin (e.g. "Server" = Complex). AI-estimated complexity is out of scope for V1. All numbers are defaults that the Admin can change (UC-WF-01).

*Example:* Mai holds 3 tickets P1 Complex → 3 × (3×2) = **18**; Hung holds 5 tickets P3 Simple → 5 × 1 = **5**. A new "broken mouse" ticket goes to **Hung**, although he holds more tickets.

### Selection rule

1. Keep members of the team responsible for the ticket's category (or the default support team).
2. Remove technicians who are `OFF`.
3. Remove technicians whose workload has reached the **maximum load** (default 20).
4. Pick the technician with the **lowest workload score**; on a tie, the one who received a ticket least recently.
5. Nobody left → the ticket waits in the team queue (`ASSIGNED` to the team, no individual); the first technician who becomes eligible receives it.

```mermaid
flowchart TD
    t["Ticket in flow C"] --> rule["AI category → responsible team<br/>(no team → default support team)"]
    rule --> f1["Keep members who are ON"]
    f1 --> f2["Drop members at maximum load"]
    f2 --> any{"Anyone left?"}
    any -- "Yes" --> pick["Lowest workload score<br/>→ ASSIGNED to that technician"]
    any -- "No" --> wait["Waits in the team queue<br/>first eligible technician gets it"]
```

## Role × feature matrix

| Feature | Admin | Technician | Requester |
|---|---|---|---|
| Configure SLA and business calendar | ✓ | | |
| Configure AI scope (AI actions, whitelist, entitlement catalog) | ✓ | | |
| View AI decisions and their grounds | ✓ | Assigned tickets | Own tickets (summary reason) |
| Configure workload scoring (priority points, complexity by request type, maximum load) | ✓ | | |
| Set own availability status | | ✓ | |
| Reassign tickets manually (last resort, reason required) | ✓ | | |
| Receive reminders / SLA warnings | | ✓ | ✓ (when the AI is waiting for them) |
| Receive SLA breach escalations | ✓ | | |
| Submit requests | ✓ | ✓ | ✓ |
| Answer AI questions | ✓ | ✓ | ✓ (own tickets) |
| Retry / resolve manually on failure | ✓ | Assigned tickets | |
| Confirm result / report not fixed | | | Own tickets |

Menus and buttons are hidden when the user lacks permission. Unauthorized URL access → `#/403`. **The backend enforces authorization**; hiding frontend controls serves UI/UX only.

## Use Case Diagram

```mermaid
flowchart LR
    subgraph left["Actors"]
        direction TB
        admin(["Admin"])
        tech(["Technician"])
    end

    subgraph system["Workflow & Automation — System Boundary"]
        direction TB
        subgraph config["Configuration"]
            direction LR
            uc01["UC-WF-01 Configure workload scoring"]
            uc02["UC-WF-02 Configure SLA"]
            uc03["UC-WF-03 Configure AI processing scope"]
        end
        subgraph routing["Routing & dispatch"]
            direction LR
            uc04["UC-WF-04 Route ticket"]
            uc05["UC-WF-05 Auto-assign technician"]
            uc06["UC-WF-06 SLA reminders & warnings"]
        end
        subgraph automation["Automated handling"]
            direction LR
            uc07["UC-WF-07 AI approves in-scope request"]
            uc08["UC-WF-08 Execute via API"]
            uc09["UC-WF-09 Verify / hand off on failure"]
        end
    end

    subgraph right["Actors"]
        direction TB
        requester(["Requester"])
        ai(["AI Service"])
    end

    admin --> uc01 & uc02 & uc03 & uc05
    tech --> uc05 & uc06 & uc09
    requester --> uc04 & uc06 & uc08 & uc09
    ai --> uc04 & uc05 & uc07

    classDef actor fill:#666,color:#fff,stroke:#333
    classDef sysactor fill:#8a5cf6,color:#fff,stroke:#5b34c9
    classDef usecase fill:#1168bd,color:#fff,stroke:#0b4f9e
    classDef core fill:#0b4f9e,color:#fff,stroke:#062f5e,stroke-width:3px
    class admin,tech,requester actor
    class ai sysactor
    class uc01,uc02,uc03,uc06,uc09 usecase
    class uc04,uc05,uc07,uc08 core
    style left fill:#fff,stroke:#bbb,stroke-dasharray:5 5
    style right fill:#fff,stroke:#bbb,stroke-dasharray:5 5
    style system fill:#f8fbff,stroke:#1168bd,stroke-dasharray:5 5
```

Thick border: core business use cases. Purple: system actor (AI Service). UC-WF-05, 06, 08 and 09 are triggered automatically by the system; the connected actors are those who benefit from the result or act on it (reassign, retry, confirm).

### Shared rules and constraints

```mermaid
flowchart LR
    auth["Authenticate session"]:::shared
    authorize["Check RBAC"]:::shared
    validate["Validate input"]:::shared
    audit["Write audit log (source USER / AI / AUTOMATION)"]:::shared
    whitelist["Whitelist check"]:::rule
    policy["Deterministic policy check"]:::rule
    idem["Idempotency key"]:::rule

    configUc["UC-WF-01..03 Configuration"] --> auth & authorize & validate & audit
    routeUc["UC-WF-04 Routing"] --> audit
    approveUc["UC-WF-07 AI approval"] --> whitelist & policy & audit
    execUc["UC-WF-08 API execution"] --> whitelist & policy & idem & audit
    retryUc["UC-WF-09 Manual retry"] --> auth & authorize & policy & idem & audit

    classDef shared fill:#1168bd,color:#fff,stroke:#0b4884
    classDef rule fill:#e8f1fc,color:#123,stroke:#3a7bd5
```

## Related states

### Ticket status (workflow-controlled part)

| Status | Meaning |
|---|---|
| `NEW` | Just created, waiting for AI classification |
| `NEEDS_INFO` | AI is asking the requester for clarification — **SLA paused** |
| `ASSIGNED` | Assigned to a technician, or waiting in the team queue when no technician is eligible |
| `IN_PROGRESS` | Technician is working on it |
| `AUTOMATING` | AI approved; the system is executing / verifying via API |
| `RESOLVED` | Handled, waiting for requester confirmation |
| `CLOSED` | Closed |
| `CANCELLED` | Cancelled by the requester |

### AI decision (`ai_decisions.outcome`)

| Value | Meaning | Flow |
|---|---|---|
| `AUTO_APPROVED` | Confidence and policy check passed | A |
| `NEEDS_INFO` | Not confident or information missing | B |
| `ROUTE_TO_HUMAN` | Needs a human / outside whitelist / policy check failed / clarification limit reached | C |

Each decision stores: category, confidence, extracted information, result of each policy condition, and reason.

### Workflow run status (`workflow_runs.status`)

`QUEUED` → `RUNNING` → (`RETRY_WAIT` → `RUNNING`)* → `VERIFYING` → `SUCCEEDED`
Failure branch: `FAILED` → `QUEUED` (manual retry) | `MANUALLY_RESOLVED`

```mermaid
stateDiagram-v2
    [*] --> NEW
    NEW --> NEEDS_INFO: AI not confident / information missing
    NEEDS_INFO --> NEW: requester answers → re-route
    NEW --> AUTOMATING: AI approves (flow A)
    NEW --> ASSIGNED: route to human (flow C)
    AUTOMATING --> RESOLVED: execution + verification OK
    AUTOMATING --> ASSIGNED: permanent error / retries exhausted → technician
    ASSIGNED --> IN_PROGRESS: technician starts
    IN_PROGRESS --> AUTOMATING: technician retries automation
    IN_PROGRESS --> RESOLVED: technician done
    RESOLVED --> CLOSED: requester confirms
    RESOLVED --> IN_PROGRESS: requester reports not fixed
    NEEDS_INFO --> CLOSED: no response after N days
    NEW --> CANCELLED: requester cancels
    CLOSED --> [*]
    CANCELLED --> [*]
```

## Use cases

### UC-WF-01 Configure workload scoring

> Assignment rules ("ticket type → team") were **removed**: the team comes from the AI category and the categories each team is responsible for (F09). This use case now only configures how the technician inside the team is chosen.

- **Actor:** Admin
- **Preconditions:** logged in as `ADMIN`; at least one technical team has active members and responsible categories (F09)
- **Main flow:** open SCR-WF-01 *Workload settings* → edit the priority points (P1–P3), the complexity of each request type (Simple / Medium / Complex) and its factor, the maximum load per technician, and the **default support team** for tickets whose category has no responsible team → `updateWorkloadSettings` → audit
- **Secondary flow — preview:** choose a sample ticket → `previewAssignment` → shows the AI category, the responsible team, each member's status (`ON` / `OFF`) and workload score, who is excluded and why, and the expected technician; no data is changed
- **Exceptions:** points, factors or maximum load ≤ 0 → `400`; a request type without a complexity → `400`; no default support team, or the default team has no active members → `400`
- **Permission:** `workflowConfigManage`
- **Result:** new settings apply from the next assignment; tickets already assigned are not moved

### UC-WF-02 Configure SLA policies

- **Actor:** Admin
- **Preconditions:** logged in as `ADMIN`
- **Main flow:** open SCR-WF-02 → choose a priority (P1–P3) → enter first-response time, resolution time, warning threshold (%) → choose a business calendar (working hours, holidays) → `updateSlaPolicy` → a new policy version is saved → audit
- **Suggested defaults** (three levels; there is no "critical" level):

  | Priority | First response | Resolution | Warning threshold |
  |---|---|---|---|
  | P1 — High | 1 hour | 8 business hours | 75% |
  | P2 — Medium | 4 business hours | 3 business days | 75% |
  | P3 — Low | 8 business hours | 5 business days | 75% |

- **Exceptions:** resolution time < response time, threshold outside 1–99% → `400`
- **Permission:** `workflowConfigManage`
- **Result:** new tickets, or tickets whose priority changes after saving, use the new version; open tickets keep their attached version (BR-WF-08)

### UC-WF-03 Configure the AI processing scope

- **Actor:** Admin
- **Preconditions:** logged in as `ADMIN`; connectors to the external access-management system are declared in deployment configuration (secrets are never entered through the UI)
- **Main flow — AI actions:** SCR-WF-03 tab *AI actions* → enable/disable each action (`AUTO_CLASSIFY`, `ASK_FOR_INFO`, `AUTO_APPROVE`, `SUGGEST_KB`, `SUGGEST_PRIORITY`) → set the confidence threshold (0.00–1.00) and the maximum clarification rounds (default 2) → `updateAiActionSetting` → audit
- **Main flow — whitelist:** tab *Automation actions* → *Add* → enter request type, connector, operation, required parameters, maximum access duration, fallback team on failure → `createAutomationAction` → audit. V1 operations:

  | Operation | Meaning | Changes data? |
  |---|---|---|
  | `CHECK_ACCOUNT` | Check account status (exists, locked, current access) and return it to the requester | No (read-only) |
  | `CREATE_ACCOUNT` | Create a new account from the department account template | Yes |
  | `GRANT_ACCESS` | Grant additional access on a system | Yes |
  | `REVOKE_ACCESS` | Revoke access on expiry (only invoked by the system) | Yes |

- **Main flow — entitlement catalog:** tab *Entitlement catalog* → choose a department → declare the entitlements / account templates the AI may grant to that department, with a maximum duration → `updateDepartmentEntitlements` → audit
- **Exceptions:** connector does not exist → `400`; entitlement references a system without a connector → `400`; threshold outside 0.00–1.00 → `400`
- **Permission:** `workflowConfigManage`
- **Result:** the AI only self-approves requests that are in an enabled whitelist entry **and** in the beneficiary department's entitlement catalog

### UC-WF-04 Route ticket

- **Actors:** Requester; AI Service
- **Trigger:** a new ticket is created, or the requester answers an AI question
- **Main flow:**
  1. The AI Service classifies the ticket, estimates confidence, and extracts information (beneficiary, system, entitlement / account type, duration)
  2. The system writes an `ai_decisions` record and selects the flow according to the *Routing principles* table
  3. Flow A → UC-WF-07; flow C → UC-WF-05
- **Alternative — clarification (flow B):** confidence < threshold or a required field is missing → the AI sends a specific question to the requester (e.g. *"Which system do you need access to?"*) → ticket `NEEDS_INFO` → requester answers → back to step 1. Still not confident after 2 rounds → flow C
- **Alternative — requester does not answer:** see UC-WF-06 (reminders and auto-close)
- **Exception:** AI Service unavailable / error → ticket goes to flow C with reason `AI_UNAVAILABLE`
- **Permission:** create ticket: `ticketCreate`; answer: ticket owner
- **Result:** each ticket has exactly one current routing decision, with its grounds, recorded in the audit log

### UC-WF-05 Auto-assign technician

- **Actors:** Technician (assignee); Admin (manual reassignment, last resort); AI Service (provides the summary)
- **Trigger:** a ticket enters flow C (needs a human, out of scope, policy check failed, clarification limit reached), a ticket is returned from UC-WF-09, or a technician becomes eligible again (status back to `ON`, load below the maximum)
- **Main flow:** the system takes the **AI category** of the ticket → finds the **team responsible** for that category (F09) → applies the *Selection rule* (members who are `ON`, below maximum load, lowest workload score) → creates an assignment record → ticket `ASSIGNED` → sends a notification including **the AI summary and the reason it was routed to a human** → audit with source `AUTOMATION`, the category, the team and the workload scores compared
- **Alternatives:**
  - No team is responsible for the category, or the AI could not classify the ticket → the **default support team** is used; same selection rule
  - Nobody in the team is eligible → the ticket waits in the team queue; the first technician who becomes eligible receives it automatically
  - A technician switches to `OFF` → their **not-started** tickets (`ASSIGNED`) are reassigned with the same selection rule; tickets `IN_PROGRESS` stay with them unless the SLA is breached (UC-WF-06)
  - Admin reassigns manually → `reassignTicket` with a mandatory reason → audit with before/after values
- **Permission:** automatic: internal system; manual reassignment: `ticketReassign` (Admin only)
- **Result:** every ticket that needs a human is assigned to the least-loaded technician who is `ON` in the right team, without anyone distributing work by hand

### UC-WF-06 Response reminders and SLA warnings

- **Actors:** Technician, Admin, Requester (notification recipients)
- **Trigger:** automatic — a system job runs every minute
- **Main flow — warning:** ticket reaches the warning threshold → reminder to the assignee (once per threshold) → `SLA_WARNING` event recorded
- **Main flow — breach:** deadline passed → flag `sla_breached` → if configured, raise priority by one level → **if the ticket is not started (`ASSIGNED`) or its assignee is now `OFF`, reassign it automatically** with the selection rule (UC-WF-05) → notify the Admin → `SLA_BREACHED` event recorded. A ticket `IN_PROGRESS` whose assignee is `ON` stays with them (the work already done is not lost).
- **Main flow — waiting for requester:** ticket in `NEEDS_INFO`, or `RESOLVED` awaiting confirmation → remind the requester on day 1 and day N−1 → on day N (default 5 business days) `NEEDS_INFO` becomes `CLOSED` with reason *no response*; the requester can reopen within 7 days
- **Exception:** `AUTOMATING` tickets are never auto-closed (BR-WF-11)
- **Permission:** internal system
- **Result:** no ticket becomes overdue without someone being notified

### UC-WF-07 AI approves in-scope request

- **Actors:** AI Service; Requester (receives the result)
- **Trigger:** UC-WF-04 places the ticket in flow A
- **Preconditions:** `AUTO_APPROVE` is enabled; confidence ≥ threshold; required information is complete
- **Main flow:**
  1. The system runs the **deterministic policy check** on the information extracted by the AI:
     - the operation is in an enabled whitelist entry;
     - the entitlement / account template is in the beneficiary department's catalog;
     - duration ≤ maximum duration;
     - the beneficiary is the requester, or in the same department as the requester (BR-WF-16);
     - the beneficiary does not already have that access (avoid duplicate grants).
  2. All pass → `ai_decisions.outcome = AUTO_APPROVED` with the result of each condition → ticket `AUTOMATING` → triggers UC-WF-08
  3. Notify the requester: *"Your request has been approved automatically and is being processed"*
- **Alternatives:**
  - `CHECK_ACCOUNT` (read-only) → only checks the whitelist and that the requester may view that account, then proceeds to UC-WF-08
  - Any condition fails → `ROUTE_TO_HUMAN` with a specific reason (e.g. *"Entitlement X is not in the Accounting catalog"*) → UC-WF-05
  - Beneficiary already has the access → ticket `RESOLVED` with the message *"You already have this access"*
- **Exception:** the AI Service returns invalid output (missing fields, wrong format) → flow C
- **Permission:** internal system; the AI can never approve outside the whitelist and the entitlement catalog (BR-WF-02)
- **Result:** basic requests are approved within seconds without human involvement; every decision has grounds the Admin can review

### UC-WF-08 Execute request via API

- **Actor:** Requester (receives the result)
- **Trigger:** automatically right after AI approval (`AiDecisionApproved` event), or when a technician clicks *Retry* (UC-WF-09)
- **Preconditions:** decision `AUTO_APPROVED`; the whitelist and entitlement catalog **at execution time** still allow it (BR-WF-12)
- **Main flow:** the system creates a `workflow_run` with idempotency key = `decisionId + action` → calls the connector of the external access-management system → stores each call in `workflow_run_attempts` (request/response with sensitive data masked) → success → UC-WF-09 verification step → if the access is time-bound, schedule revocation
- **Alternatives:**
  - `CHECK_ACCOUNT` → the check result is sent to the requester, ticket `RESOLVED`
  - Transient error (timeout, `5xx`, `429`) → `RETRY_WAIT` → up to 3 retries, 1 / 5 / 15 minutes apart
  - Permanent error (business `4xx`, permission denied) or retries exhausted → `FAILED` → UC-WF-09 failure branch
  - Revocation due → create a `workflow_run` with `REVOKE_ACCESS`, notify the user 1 day in advance
- **Exception:** preconditions not met → no API call, `FAILED` with code `PRECONDITION_FAILED`
- **Permission:** internal system; connectors use least-privilege service accounts
- **Result:** one approval decision leads to **exactly one** change in the external access-management system

### UC-WF-09 Verify result, hand off on failure

- **Actors:** Requester (confirms the result), Technician (handles failures)
- **Trigger:** automatically after UC-WF-08 finishes
- **Main flow — verification:** the system calls the connector's check operation (e.g. the account exists, the user is in the access group) → match → `workflow_run` `SUCCEEDED` → ticket `RESOLVED` → notify the requester → requester clicks *Confirm* → `confirmTicketResolution` → `CLOSED`
- **Alternative — requester reports not fixed:** `reopenTicket` → assigned to a technician of the owning team (UC-WF-05)
- **Alternative — failure:** `workflow_run` `FAILED` or verification mismatch → assigned to a technician of the owning team with the error code, failed step and attempt history → the technician opens SCR-T-02 tab *Automation*:
  - *Retry* → `retryWorkflowRun` → back to UC-WF-08 (policy re-checked)
  - *Resolved manually* (note required) → `resolveWorkflowRunManually` → `MANUALLY_RESOLVED`, ticket `RESOLVED`
- **Exception:** retry when the current policy no longer allows it → `409`
- **Permission:** confirm: ticket owner; retry / manual resolution: `workflowRunRetry` + assignee or Admin
- **Result:** a ticket is never marked resolved without successful verification or a technician's confirmation (BR-WF-13)

**Related:** [Features](01-features.md) · [INVEST backlog](04-invest-requirements.md) · [Acceptance criteria](03-acceptance-criteria.md) · [Business rules](../requirements/business-rules.md)
