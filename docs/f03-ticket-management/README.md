# F03 Ticket Management

> Design status: TARGET V1
>
> Implementation status: NOT IMPLEMENTED

The **Ticket Management** branch of the final mind map ([`docs/mindmap.png`](../mindmap.png)): raising tickets by email or free text, finding them, the summary a person reads when a ticket is handed to them, moving a ticket inside its division, adding missing information, resolving, and closing.

**F05 routes, F04 processes, F03 owns the ticket.** F03 receives the email or the portal request, stores the ticket, its conversation, attachments, links and Resolution, sends the emails, and gives every role the ticket pages. F05 classifies, fills the fields, sets the priority, holds the ticket and asks, and routes it to an AI agent or a technician; F04 resolves what reaches an AI agent.

## Scope — the seven leaves of the mind map

| # | Mind-map leaf | Use case | Stories |
|---|---|---|---|
| 1 | Raise ticket by email or free text, no category form | UC-TK-01 | US-TK-01 … 04 |
| 2 | View / search / filter tickets | UC-TK-02 | US-TK-05 … 09 |
| 3 | View ticket summary on handoff and approval | UC-TK-03 | US-TK-10 … 12 |
| 4 | Transfer / escalate within the same division (admin, technician) | UC-TK-04 | US-TK-13, 14 |
| 5 | Add missing information to on-hold tickets | UC-TK-05 | US-TK-15 … 18 |
| 6 | Update status and resolution | UC-TK-06 | US-TK-19 … 21 |
| 7 | Confirm / close / reopen / cancel ticket | UC-TK-07 | US-TK-22 … 24 |

Not in F03: classification, AI-filled fields, priority and SLA, duplicates, routing and assignment, re-routing (F05); AI resolution, API execution, Resolution drafts (F04); 1-click claim on the workbench (F02); ticket history and approval log (F08).

| File | Content |
|---|---|
| [01-user-stories.md](01-user-stories.md) | US-TK-01 … US-TK-24 with acceptance criteria, INVEST review, traceability, mapping from the previous version |
| [02-use-cases.md](02-use-cases.md) | Actors, lifecycle and transitions, UC-TK-01 … UC-TK-07, business rules BR-TK-01 … BR-TK-26, permissions, open questions |
| [information-architecture/](information-architecture/README.md) | Pages, dialogs, sections and emails (SCR-T, DLG-T, SEC-T, EML-T), IA decisions, routes and permissions, navigation, notifications, screen content and wireframes, user flows, UI states, data and API |

## Taken over and handed over

| Item | From / to | Note |
|---|---|---|
| Technician summary (UC-AI-04, US-AI-07, US-AI-08, SEC-AI-06) | **From F04** | Now UC-TK-03, US-TK-10 … 12, SEC-T-01 |
| Receiving the mailbox, threading replies, sending emails, storing the Resolution | F03 (confirmed by F04 D-AI-02) | UC-TK-01, UC-TK-05, UC-TK-06 |
| Resolution codes `ANSWERED_BY_KB`, `SOLVED_BY_KB`, `GUIDED_BY_AI` | Requested by F04 (Q-AI-18) | Added to UC-TK-06 |
| Similar tickets and KB while typing (old US-TK-04) | **To F05 / F04** | F05 UC-WF-10, F04 SEC-AI-01 |
| Assign / reassign, triage queue (old US-TK-10, DLG-T-01) | **To F05** | Automatic assignment UC-WF-05; no triage queue (F05 D-IA-07) |
| Claim (old US-TK-11) | **To F02** | Technician workbench US-DB-08 |
| Change category / priority (old US-TK-19, DLG-T-09) | **To F05** | `correctTicketField`, `changeTicketPriority` |

**Related:** [README](../../README.md) · [Previous: F02 Dashboard](../f02-dashboard/README.md) · [Next: F04 AI Ticket Processing](../f04-ai-ticket-processing/README.md)
