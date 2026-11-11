# UI/UX States and Accessibility

> Design status: V1 — draft, awaiting user review
>
> Implementation status: NOT STARTED

The interface copy in this document is written in English; the product UI language is decided at the UI/UX step. Copy quoted from an acceptance criterion is used word for word; copy not found in the requirements is marked *(proposed)* and may change during UI/UX.

## UI state contract

### General states

| State | Required behavior |
|---|---|
| Initial loading | Keep the page shell and sidebar; show a skeleton for the affected list or panel. |
| Empty | Say that nothing exists yet and offer the permitted next action (e.g. *"You have no tickets yet — Create ticket"* for the Requester). Distinguish "nothing yet" from "no results for these filters". |
| No results | Show the active filters and a **Clear filters** action. |
| Validation error | Keep the entered values, show the error next to the field, focus the first error. Examples: *"The target team has no active members"*, *"The threshold must be between 0 and 1"*, *"Please enter a reason"* *(proposed)*. |
| Server error | Show a short message and a **Retry** button; never show technical details. |
| Not signed in / session expired | Clear the session, go to SCR-01 with a message. |
| Forbidden | Show SCR-90 with a link to the home page. Another person's ticket also gives SCR-90, without hinting that it exists. |
| Saving | Disable the submit button to prevent double submission, show progress. |
| Success | Short confirmation, close the modal if any, refresh the affected list or panel. |
| Conflict (data changed by someone else) | Nothing is saved. Keep the entered data, show *"This item was just changed by someone else. Reload to see the change."* *(proposed)* and a **Reload** button. Applies to reassignment, retry, manual resolution, rule edits, SLA edits, AI scope edits. |
| Confirmation of an action with consequences | Name the object and the effect. **Disabling `AUTO_APPROVE`**: *"All new requests will go to technicians until it is re-enabled."* **Removing an entitlement from the catalog**: *"The AI will no longer grant this access; executions still pending will be blocked (BR-WF-12)."* **Archiving a rule**: *"New tickets will no longer match this rule."* **Retry**: *"Re-checks the policy and calls the API again; no duplicate grant."* *(all proposed)* |

### AI and execution states

| State | Display |
|---|---|
| AI analyzing | SCR-T-02: *"The AI is analyzing your request…"* with a running indicator; the page refreshes itself (DF-IA-07). Does not prevent the requester from leaving the page. |
| AI asks for clarification | AI message in the conversation panel, labelled **AI**; the answer box receives focus; shows *"Question 1 of 2"*. |
| AI unavailable | SCR-T-02: *"The AI system is temporarily unavailable. Your request has been handed to a technician."* *(proposed)*. No technical error shown. |
| AI approved | Progress bar step *Approved* with the label **Automatically approved by AI**; one-line summary reason. |
| Handed to a technician | The Requester sees a summary reason (DF-IA-08); the Technician sees the AI summary labelled **AI** and the detailed reason. |
| Executing | Progress bar step *Executing*; when waiting for a retry: *"The external system has not responded yet, retrying at 10:35"* *(proposed)*. |
| Execution failed | Requester: *"Something went wrong; a technician has been assigned."*; Technician: error code, failed step, attempt history. |

**AI-generated content** (summaries, questions, reasons) always carries a text label **AI**, never only an icon.

**Routing information** on SCR-T-02 and SCR-T-03:

| Chip | Meaning |
|---|---|
| *Filled by AI* | The field was written from the AI analysis; a corrected field shows *Corrected by <name>* instead. |
| `L1` / `L2` | Support level computed by F04 (Helpdesk vs application domain or business requirement). |
| *Email* / *Portal* | Source channel of the ticket. |
| *AI agent — Helpdesk* / *Technician — Hung* | Current owner of the ticket. |
| *P1 High · set by AI* / *P2 Medium · default* | Priority and how it was set. |
| *Duplicate of #T-1024* | The ticket was merged into the requester's open ticket; *Not the same problem* is shown for 7 days. |
| *Related (3)* | Tickets of other people about the same problem, handled by the same owner. |
| *Re-routed 1 / 2* | The ticket was routed again after a misroute or a failure; the routing history explains why. |
| *Main ticket · possible incident* | The oldest ticket of a burst of similar tickets; its related tickets are listed under it. |

### Status chips

A chip always carries its text. Colour supports the text and never replaces it.

**Ticket status**

| Chip | Code | Meaning shown to the user |
|---|---|---|
| New | `NEW` | Just created, the AI is analyzing. |
| More info needed | `NEEDS_INFO` | The AI is waiting for the requester's answer; SLA paused. |
| Assigned | `ASSIGNED` | Assigned to a technician (name shown). |
| Waiting for a technician | `ASSIGNED` (team only) | In the team queue: nobody is available right now; it will be assigned automatically as soon as someone is. |
| In progress | `IN_PROGRESS` | A technician is working on it. |
| Automating | `AUTOMATING` | Approved by the AI; the system is executing / verifying. |
| Awaiting confirmation | `RESOLVED` | Done, waiting for the requester's confirmation. |
| Closed | `CLOSED` | Finished; shows the reason if auto-closed. |
| Cancelled | `CANCELLED` | Cancelled by the requester. |

**Processing flow (AI decision)**

| Chip | Code | Meaning |
|---|---|---|
| AI self-service | `AUTO_APPROVED` | Confidence and policy check passed. |
| Clarification | `NEEDS_INFO` | Not confident or information missing. |
| Handed to technician | `ROUTE_TO_HUMAN` | Needs a human; the reason is shown next to the chip. |

**Technician availability** (top bar, Workload tab, Reassign modal)

| Chip | Code | Meaning |
|---|---|---|
| ON | `ON` | Taking work: receives new tickets. |
| OFF | `OFF` | Not taking work (off shift, leave, away): no new tickets; not-started tickets are moved to colleagues. |

Next to the status, the Workload tab and the Reassign modal show the **workload score** as a number with its maximum, e.g. *"12 / 20"*.

**Automation run** (Automation tab)

| Chip | Code |
|---|---|
| Queued | `QUEUED` |
| Running | `RUNNING` |
| Waiting to retry | `RETRY_WAIT` |
| Verifying | `VERIFYING` |
| Succeeded | `SUCCEEDED` |
| Failed | `FAILED` |
| Resolved manually | `MANUALLY_RESOLVED` |

**SLA:** chips *2 h 15 min left*, *Due soon* (warning threshold reached), *Overdue* (breached), *Paused* (in `NEEDS_INFO`).

**AI confidence** (Technician, Admin only): shown as a percentage with text relative to the threshold, e.g. *"87% — above the 80% threshold"*.

## Responsive and accessibility rules

- **Requesters can use their screens on a phone:** SCR-T-01, SCR-T-02, SCR-T-03 and SCR-91 must work on narrow screens; the AI conversation panel takes the full width; the *Confirm* / *Not fixed* buttons are always visible.
- **Configuration and monitoring screens** (SCR-WF-01..31) target desktops; on narrow screens tables scroll horizontally or switch to summary cards, never hiding data permanently.
- **Modals** fit within the viewport, keep their action buttons visible, trap focus, close with `Esc` when no submission is running, and return focus to the button that opened them.
- Every input has a visible label; icon-only buttons have accessible names; status is never conveyed by colour alone.
- Policy conditions on SCR-WF-31 use **✓ / ✗ with the words** "Pass" / "Fail", not only green/red.
- Automatic status updates on SCR-T-02 are announced to screen readers (polite `aria-live` region) without stealing focus.
- Keyboard users move through navigation, filters, table rows and modal actions in a predictable order.

**Related:** [Information architecture index](README.md) · [Screen inventory](04-screen-inventory.md) · [User flows](05-user-flows.md)
