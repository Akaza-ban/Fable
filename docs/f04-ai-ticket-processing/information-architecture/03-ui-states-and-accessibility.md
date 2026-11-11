# UI States and Accessibility — F04 AI Ticket Processing

> Design status: TARGET V1
>
> Related: [IA overview](README.md) · [Screens and navigation](01-screens-and-navigation.md) · [User flows](02-user-flows.md)

## 1. Who did it: AI, workflow, or person (D-AI-IA-06)

Agent-first (D-AI-01) works only if everyone can tell who produced each thing on screen. Every AI-related element carries one of these markers, with an icon and text, never color alone.

| Marker | Meaning | Where |
|---|---|---|
| **AI** badge on a message | Written and sent by the AI Service (author `AI`) | SEC-AI-03, SEC-AI-17, EML-AI-03, 04, 07, 08 |
| **Filled by AI** | A domain-form value from the agent, not yet confirmed by a person; source quote on hover / focus | SEC-AI-07 |
| **Corrected by {name}** | A person changed the agent's value (UC-AI-07) | SEC-AI-07 |
| **Written by AI** | A Resolution the AI filled after resolving the ticket itself | Ticket Resolution field, SEC-AI-03 |
| **Drafted by AI** | Text a person reviewed and sent in their own name | SEC-AI-13, EML-AI-06 |
| **Executed by AI** · **Verified** | A change made in an external system by F04, and its successful check | SEC-AI-16 |
| **Approved by {policy / name}** | Who approved the executed action: the F05 policy check or an Administrator in F03 | SEC-AI-16 |
| **Decided by workflow** | A decision of F05 (policy check, routing, escalation set by F05) | SEC-AI-05 |
| **Confirmed by requester** | The requester confirmed an answer, a guided fix or a brief | SEC-AI-03, SEC-AI-11, SEC-AI-17 |
| **Reviewed by {name}** · **Flagged incorrect by {name}** | A technician or the Admin reviewed a ticket the agent resolved alone | SEC-AI-19, SEC-AI-14 |

## 2. AI state contract

Every F04 section shows exactly one of these states. Sections load asynchronously; the rest of the ticket page never waits for them.

| State | When | What the user sees | Sections |
|---|---|---|---|
| `LOADING` | Agent, retrieval or draft running | Skeleton and "The assistant is working on this ticket…"; no spinner longer than the time budget (10 s agent, 3 s suggestions, 10 s draft) | All |
| `READY` | Result available | The content, with its marker | All |
| `EMPTY` | Nothing above the threshold | "No matching article in the knowledge base" | SEC-AI-01, 04, 09 |
| `WAITING_FOR_REQUESTER` | A guidance step was sent and has no answer yet | Requester: the step card with its answers. Technician: "Waiting for the requester, step *n* of the procedure, sent {time}" | SEC-AI-17 |
| `RUNNING` | A run is `QUEUED`, `RUNNING`, `RETRY_WAIT` or `VERIFYING` | Technician: status, attempt count, next retry time. Requester: "Being processed" | SEC-AI-16 |
| `FAILED_RUN` | A run is `FAILED` | Technician: error code, failed step, *Retry*, *Resolved manually*. Requester: "A technician is taking over" | SEC-AI-16 |
| `UNAVAILABLE` | AI Service down or timed out | "Suggestions are temporarily unavailable" / "The assistant is unavailable"; the ticket works normally | All AI sections |
| `INVALID_OUTPUT` | Agent result `INVALID` or `FAILED` | Technician / Admin: "The assistant could not handle this ticket; it was sent to a person" and the failed check (Admin only). Requester: nothing special | SEC-AI-05, 07, 18 |
| `UPDATED` | A newer result version exists | "Updated after a correction" with a link to earlier versions | SEC-AI-07, SEC-AI-18 |
| `HIDDEN` | The action is not allowed in this state | Not rendered (for example *This solved my problem* while `AUTOMATING`, *Retry* for a non-assignee) | SEC-AI-03, 04, 16 |

## 3. Escalation banner (SEC-AI-05)

Shown at the top of SCR-T-02 for technicians and Admins only (D-AI-IA-03).

| `escalationClass` | Icon + label | Content |
|---|---|---|
| `AGENT_CANNOT_RESOLVE` | person icon · "Needs a person" | Reason in plain words (for example "Hardware request", "Guidance did not fix it", "Execution failed: folder not found", "Requester says the answer did not help", "Access not in Accounting's catalog"), who set it (domain agent or workflow) |
| `SYSTEM_BUG` | bug icon · "Suspected system bug" | The error, the affected system, the linked similar tickets (burst) |
| `SOFTWARE_REQUIREMENT` | document icon · "Software requirement" | Link to the brief and its state (confirmed / not confirmed) |

Every banner ends with **What the agent already did**: KB reply sent (with article), guidance steps and answers (link to SEC-AI-17), action proposed, who approved it and its run result (link to SEC-AI-16).

## 4. Status chips

| Chip | Source | Text |
|---|---|---|
| Escalation | `escalationClass` | As in the banner label |
| Agent outcome | `outcome` | "Answered from KB", "Guiding the requester", "Action proposed", "Escalated" |
| Automation | `workflow_runs.status` | "Processing", "Verifying", "Done", "Failed — with a technician" |
| Review | review of an AI-resolved ticket | "Not reviewed", "Looks right", "Resolved incorrectly" |
| Resolution | F03 resolution code | "Answered by AI (KB)" for `ANSWERED_BY_KB`; "Solved with a KB article" for `SOLVED_BY_KB`; "Fixed with AI guidance" for `GUIDED_BY_AI` |

## 5. Email rules

- Plain, short emails in the language of the ticket (Vietnamese by default), readable on a phone.
- Every email shows the ticket number in the subject (`[#T-1024]`), so replies are threaded by F03.
- Every action link opens the ticket with the action pre-selected; nothing changes until the user clicks in the modal or on the step card (D-AI-IA-02).
- An AI email says it was written by the AI ("This answer was prepared by the IT assistant") and how to reach a person (*Not solved*, *I need a person*, or reply "I need a person").
- A guidance email holds exactly one step, so the requester never has to scroll through a procedure on a phone.

## 6. Responsive layout

| Width | SCR-T-02 layout |
|---|---|
| ≥ 1200 px | Timeline in the centre; details panel with SEC-AI-07; banner above the timeline; tabs *Automation* (SEC-AI-16) and *AI* (SEC-AI-18) next to the conversation |
| 768–1199 px | Details panel below the banner; solution steps, automation and guidance transcript in tabs |
| < 768 px | One column; the requester view shows only the conversation, the current guidance step card, KB suggestions and actions; technician sections collapse into accordions |

## 7. Accessibility (WCAG 2.1 AA)

- AI sections that update asynchronously (new guidance step, run status) are `aria-live="polite"` regions; the banner is announced once on load.
- Markers and chips use text and icons, not color alone; contrast ≥ 4.5:1.
- Source quotes and citations are reachable by keyboard (focus shows the same tooltip as hover); citation links have descriptive text ("KB-087, section 3: Reset VPN profile").
- The guidance answers are real buttons in a labelled group ("Result of step 2"), and the free-text box has a visible label.
- Modals SCR-AI-04 … 07 trap focus, close with Esc, and return focus to the button that opened them.
- Vietnamese and English content carries the right `lang` attribute, so screen readers pronounce it correctly.
- Every action available by mouse (*Insert into reply*, *Correct*, *Retry*, *Resolved manually*, *Looks right*, *Resolved incorrectly*, guidance answers, votes) is available by keyboard.
