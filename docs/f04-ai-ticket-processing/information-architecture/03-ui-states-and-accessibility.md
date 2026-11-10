# UI States and Accessibility — F04 AI Ticket Processing

> Design status: TARGET V1
>
> Related: [IA overview](README.md) · [Screens and navigation](01-screens-and-navigation.md) · [User flows](02-user-flows.md)

## 1. Who did it: AI, workflow, or person (D-AI-IA-06)

Agent-first (D-AI-01) works only if everyone can tell who produced each thing on screen. Every AI-related element carries one of these markers, with an icon and text, never color alone.

| Marker | Meaning | Where |
|---|---|---|
| **AI** badge on a message | Written and sent by the AI Service (author `AI`) | SEC-AI-03, EML-AI-02 … 04 |
| **Filled by AI** | A field value from the analysis, not yet confirmed by a person; source quote on hover / focus | SEC-AI-07, SEC-AI-08 |
| **Corrected by {name}** | A person changed the AI value (UC-AI-07) | SEC-AI-07 |
| **Drafted by AI** | Text a person reviewed and sent in their own name | SEC-AI-13, EML-AI-06 |
| **Decided by workflow** | A decision of F05 (policy check, routing, escalation set by F05) | SEC-AI-05, SEC-AI-03 result messages |
| **Confirmed by requester** | The requester confirmed an answer or a brief | SEC-AI-03, SEC-AI-11 |

## 2. AI state contract

Every F04 section shows exactly one of these states. Sections load asynchronously; the rest of the ticket page never waits for them.

| State | When | What the user sees | Sections |
|---|---|---|---|
| `LOADING` | Analysis, summary or retrieval running | Skeleton and "AI is reading the ticket…"; no spinner longer than the time budget (10 s analysis, 3 s suggestions, 30 s summary) | All |
| `READY` | Result available | The content, with its marker | All |
| `EMPTY` | Nothing above the threshold | "No matching article in the knowledge base" / "No similar tickets" | SEC-AI-01, 04, 09, 10 |
| `UNAVAILABLE` | AI Service down or timed out | "Suggestions are temporarily unavailable" / "Summary unavailable"; the original description is shown; the ticket works normally | All |
| `INVALID_OUTPUT` | Analysis `INVALID` or `FAILED` | Technician / Admin: "The AI could not analyze this ticket; it was sent to a person" and the failed check (Admin only). Requester: nothing special | SEC-AI-05, 07, 15 |
| `UPDATED` | A newer version exists after a message | "Updated after a new message" with a link to earlier versions | SEC-AI-06, SEC-AI-07 |
| `LOW_CONFIDENCE` | A field or priority below the threshold | Value shown with "Low confidence — please check"; for priority, the ticket stays P3 (AC-AI-09.4) | SEC-AI-07, SEC-AI-08 |
| `HIDDEN` | The action is not allowed in this state | Not rendered (for example *This solved my problem* while `AUTOMATING`) | SEC-AI-03, SEC-AI-04 |

## 3. Escalation banner (SEC-AI-05)

Shown at the top of SCR-T-02 for technicians and Admins only (D-AI-IA-03).

| `escalationClass` | Icon + label | Content |
|---|---|---|
| `AGENT_CANNOT_RESOLVE` | person icon · "Needs a person" | Reason in plain words (for example "Hardware request", "Requester says the answer did not help", "Access not in Accounting's catalog"), who set it (domain agent or workflow) |
| `SYSTEM_BUG` | bug icon · "Suspected system bug" | The error, the affected system, the number of linked similar tickets (burst) |
| `SOFTWARE_REQUIREMENT` | document icon · "Software requirement" | Link to the brief and its state (confirmed / not confirmed) |

Every banner ends with **What the agent already did**: questions asked and answered, KB reply sent (with article), action proposed and its F05 result.

## 4. Status chips

| Chip | Source | Text |
|---|---|---|
| Category | `category` | "IT Helpdesk › Email", "SAP › FI" (domain and subcategory names, not codes) |
| Support level | `supportLevel` | "L1" / "L2" |
| Escalation | `escalationClass` | As in the banner label |
| Resolution | F03 resolution code | "Answered by AI (KB)" for `ANSWERED_BY_KB`; "Solved with a KB article" for `SOLVED_BY_KB` |

## 5. Email rules

- Plain, short emails in the language of the ticket (Vietnamese by default), readable on a phone.
- Every email shows the ticket number in the subject (`[#T-1024]`), so replies are threaded by F03.
- Every action link opens the ticket with the action pre-selected; nothing changes until the user clicks in the modal (D-AI-IA-02).
- An AI email says it was written by the AI ("This answer was prepared by the IT assistant") and how to reach a person (*Not solved* / reply "I need a person").

## 6. Responsive layout

| Width | SCR-T-02 layout |
|---|---|
| ≥ 1200 px | Timeline in the centre; side panel with SEC-AI-07 and SEC-AI-08; summary and banner above the timeline |
| 768–1199 px | Side panel below the summary; similar tickets and solution steps in tabs |
| < 768 px | One column; requester view only shows the conversation, KB suggestions and actions; technician sections collapse into accordions |

## 7. Accessibility (WCAG 2.1 AA)

- AI sections that update asynchronously are `aria-live="polite"` regions; the banner is announced once on load.
- Markers and chips use text and icons, not color alone; contrast ≥ 4.5:1.
- Source quotes and citations are reachable by keyboard (focus shows the same tooltip as hover); citation links have descriptive text ("KB-087, section 3: Reset VPN profile").
- Modals SCR-AI-02 … 05 trap focus, close with Esc, and return focus to the button that opened them.
- Vietnamese and English content carries the right `lang` attribute, so screen readers pronounce it correctly.
- Every action available by mouse (*Accept*, *Correct*, *Insert into reply*, *Link as duplicate*, votes) is available by keyboard.
