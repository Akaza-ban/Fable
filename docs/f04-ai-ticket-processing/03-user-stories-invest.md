# User Stories (INVEST) — F04 AI Ticket Processing

> Design status: TARGET V1
>
> Scope: US-AI-01 … US-AI-21

Each story is small enough to build and test on its own, or its review says why not. Authorization, ticket scope, credential masking, grounding and audit logging are **shared acceptance constraints**, not hidden features.

## INVEST checklist

| Principle | Application in this backlog |
|---|---|
| **Independent** | Each story produces a demonstrable result. A story that needs another to be meaningful says so in its review (`PARTIAL`). |
| **Negotiable** | Screens, model and prompt choices may change, while the outcome, permissions and AI guardrails stay fixed. |
| **Valuable** | Every story names the benefit for a role. System behaviour is written for the role that benefits from it. |
| **Estimable** | Inputs, outputs, roles, limits and failure cases (including AI failure) are explicit, and every story has a size. |
| **Small** | One capability per story. **S** = one screen element or rule; **M** = a flow with several rules; **L** = AI integration with validation and several checks. |
| **Testable** | Every story has Given / When / Then criteria in [04-acceptance-criteria.md](04-acceptance-criteria.md). AI quality is measured on named evaluation sets (ES-AI-01 … ES-AI-05). |

## Product backlog

MoSCoW: **Must** (required for V1), **Should** (planned, but the core works without it), **Could** (if time allows).

| ID | User story | Value | Dependencies | MoSCoW | Size | AC |
|---|---|---|---|---|---|---|
| US-AI-01 | As a **Requester**, I want to describe my problem in free text and have the ticket classified automatically right after I submit it, so that I do not have to pick a category or fill in a category form, and it reaches the right flow straight away. | No manual triage, no rigid forms | F03 create ticket | Must | M | AC-UC-AI-01 |
| US-AI-02 | As an **Admin**, I want every AI analysis to store the request type, extracted fields with their source, missing fields, confidence and suggested route in a fixed, validated format, so that the workflow acts only on trustworthy data and I can audit every result. | Safe input for F05 | US-AI-01 | Must | L | AC-UC-AI-01 |
| US-AI-03 | As a **Requester**, I want the AI to ask me one specific question that names exactly what is missing, so that my request is understood correctly instead of being handled wrongly or left waiting. | Fewer wrong or stalled tickets | US-AI-02 | Must | M | AC-UC-AI-02 |
| US-AI-04 | As a **Requester**, I want to answer the AI in my own words inside the ticket and have it picked up automatically, so that I do not have to re-create the ticket or fill in a form. | Low effort for requesters | US-AI-03 | Must | M | AC-UC-AI-02 |
| US-AI-05 | As a **Requester**, I want to see up to 3 knowledge-base articles that match my problem while I write the ticket and after I submit it, and to mark one as the fix, so that I can solve common problems myself without waiting. | Self-service resolution | F06 published articles | Must | M | AC-UC-AI-03 |
| US-AI-06 | As a **Technician**, I want suggested solution steps where every step cites the KB article it comes from, so that I can check and reuse a known fix quickly instead of searching the KB. | Faster, verifiable fixes | F06 published articles | Must | L | AC-UC-AI-03 |
| US-AI-07 | As a **Technician**, I want a short summary at the top of each ticket routed to me (problem, impact, key details, what was tried, why it came to a human), so that I can start working without reading the whole conversation. | Less reading time | US-AI-02, F05 routing | Must | M | AC-UC-AI-04 |
| US-AI-08 | As a **Technician**, I want the summary to update when new messages arrive and to regenerate it on demand, so that it never describes an outdated situation. | Current information | US-AI-07 | Should | S | AC-UC-AI-04 |
| US-AI-09 | As a **Technician**, I want the AI to suggest a priority (P1–P4) based on impact and urgency with its reason, so that urgent tickets get a short SLA from the start. | Urgent work surfaces early | US-AI-01, F05 SLA | Should | M | AC-UC-AI-05 |
| US-AI-10 | As a **Technician**, I want to accept or change the suggested priority with a reason, so that the final priority is always a human-checked value when it matters. | Human control over SLA | US-AI-09 | Should | S | AC-UC-AI-05 |
| US-AI-11 | As a **Technician**, I want to see open duplicates and past similar tickets with how they were resolved, and link a duplicate in one click, so that I do not solve the same problem twice. | Reuse of earlier fixes | US-AI-01, F03 link tickets | Should | M | AC-UC-AI-06 |
| US-AI-12 | As a **Requester**, I want to be warned before I submit a ticket that looks the same as one of my open tickets, so that I follow the existing ticket instead of creating a repeat. | Fewer repeat tickets | F03 create form | Could | M | AC-UC-AI-06 |
| US-AI-13 | As a **Technician**, I want to correct the request type, category or an extracted field when the AI is wrong, so that the ticket data is right and the error is recorded for improving the AI. | Correct data, measurable errors | US-AI-02 | Should | S | AC-UC-AI-07 |
| US-AI-14 | As a **Technician**, I want to mark the summary and KB suggestions as helpful or not helpful, so that the team can see which AI outputs work and fix the ones that do not. | Measured AI quality (F07) | US-AI-06, US-AI-07 | Should | S | AC-UC-AI-07 |
| US-AI-15 | As a **Requester** who emails the support mailbox, I want the AI to read my email as I wrote it (without my signature or the quoted thread), know who I am and which company, department and site I belong to, find who the request is for, and fill in the ticket fields, so that I never fill in a form and the ticket still has complete data. | Email tickets fully understood | US-AI-01, F03 email intake, F09 directory | Must | L | AC-UC-AI-01 |
| US-AI-16 | As a **Technician**, I want the AI to draft the Resolution and the reply to the requester when I resolve a ticket, and to write the result reply when an automation succeeds, so that every ticket closes with a clear answer and a recorded Resolution without extra typing. | Every ticket has a Resolution | US-AI-07, F03 resolve | Must | M | AC-UC-AI-08 |
| US-AI-17 | As a **Requester** with a how-to question or a problem that has a known self-service fix, I want the domain agent to answer my ticket with cited steps from the KB and let me confirm or say it did not help, so that I get an answer in minutes and still reach a person if needed. | Tickets solved without a technician | US-AI-05, F05 `AUTO_RESOLVE_KB` and transitions (Q-AI-10) | Must | L | AC-UC-AI-03 |
| US-AI-18 | As a **Requester** with a business need, I want the AI to turn my request into a requirement brief that I can check and confirm, so that the dev team understands what to build without several rounds of emails. | Clear input for the dev team | US-AI-01, US-AI-03 | Should | M | AC-UC-AI-09 |
| US-AI-19 | As an **Admin**, I want corrected and confirmed tickets to be reused as examples when new tickets are classified, and to exclude a bad example, so that the AI improves from real, human-verified tickets without retraining a model. | AI improves over time | US-AI-13 | Should | M | AC-UC-AI-07 |
| US-AI-20 | As a **Requester**, I want my free-text ticket to go to the AI agent of its domain (Helpdesk, SAP, Fleet Management, …), which fills the domain form and resolves the ticket itself (allowed action, KB answer or one question), and as a **Technician** I want to receive a ticket only when the agent cannot resolve it, it is a system bug, or it is a software requirement, always with the reason, so that people work only on what needs people. | Most tickets resolved without a technician | US-AI-01, US-AI-15, US-AI-17, F05 whitelist | Must | L | AC-UC-AI-10 |
| US-AI-21 | As a **Technician** of a domain team, I want tickets that point to a defect in our system flagged as a suspected bug, with similar tickets from other users linked, so that we fix the system once instead of answering each user separately. | Bugs fixed at the source | US-AI-20, US-AI-11 | Should | M | AC-UC-AI-10 |

Grouped by use case:

```text
US-AI-01–02, 15   UC-AI-01  Read, classify and fill the ticket, suggest routing
US-AI-03–04       UC-AI-02  Ask back when context is missing
US-AI-05–06, 17   UC-AI-03  Suggest KB solutions with sources (incl. KB auto-reply)
US-AI-07–08       UC-AI-04  Summarize ticket for technician
US-AI-09–10       UC-AI-05  Suggest priority
US-AI-11–12       UC-AI-06  Suggest duplicate / similar tickets
US-AI-13–14, 19   UC-AI-07  Review, feedback and learning
US-AI-16          UC-AI-08  Draft the Resolution and the reply
US-AI-18          UC-AI-09  Structure a business requirement
US-AI-20–21       UC-AI-10  Domain agents resolve tickets (agent-first)
```

## MoSCoW summary

| Priority | Stories | Count |
|---|---|---|
| **Must**: the four core leaves of the mind map, email intake, the Resolution, and the agent-first handling the client decided (D-AI-01); F05 cannot route without US-AI-01/02 | US-AI-01 … US-AI-07, US-AI-15, US-AI-16, US-AI-17, US-AI-20 | 11 |
| **Should**: important, but tickets are still routed and solved without them | US-AI-08, US-AI-09, US-AI-10, US-AI-11, US-AI-13, US-AI-14, US-AI-18, US-AI-19, US-AI-21 | 9 |
| **Could** | US-AI-12 | 1 |
| **Won't (V1)** | Reading images and attachments (OCR); splitting a ticket with several requests; model fine-tuning; integration with ManageEngine (Q-AI-7) | — |

No Must story depends on a Should story. US-AI-09 is Should because F05 has a default priority (P3, Q-AI-1) when no suggestion exists. US-AI-17 is Must because agents resolving how-to tickets is the core of D-AI-01, but it needs F05 to add `AUTO_RESOLVE_KB` and two transitions (Q-AI-10), so plan it with the F05 owner. US-AI-20 is Must; V1 seeds only the `HELPDESK` and `SAP` agents (Q-AI-13). US-AI-20 flags a suspected bug from the error alone; US-AI-21 (Should) adds the "5 similar tickets in 60 minutes" rule, which needs US-AI-11.

## Per-story INVEST review

`PASS` means the design gives enough information to plan and accept the story. It does not mean that code or tests exist. `PARTIAL` means the story is weak on that criterion, and the note says why.

| Story | I | N | V | E | S | T | Notes |
|---|---|---|---|---|---|---|---|
| US-AI-01 | PASS | PASS | PASS | PASS | PASS | PARTIAL | Testable: model output varies, so accuracy is measured on ES-AI-01 and logic is tested with a mocked model. |
| US-AI-02 | PASS | PASS | PASS | PASS | PARTIAL | PASS | Small: schema, validation and directory lookup in one story (L); each invalid case has its own criterion (AC-AI-02.2 … 02.5). |
| US-AI-03 | PARTIAL | PASS | PASS | PASS | PASS | PASS | Independent: needs `missingFields` from US-AI-02. The round limit is F05's (BR-WF-06), not part of this story. |
| US-AI-04 | PARTIAL | PASS | PASS | PASS | PASS | PASS | Independent: only meaningful after US-AI-03. |
| US-AI-05 | PARTIAL | PASS | PASS | PARTIAL | PASS | PARTIAL | Independent: needs published articles from F06 (a seeded set works for the demo). Estimable: relevance threshold calibrated on ES-AI-02. Testable on ES-AI-02. |
| US-AI-06 | PARTIAL | PASS | PASS | PARTIAL | PARTIAL | PASS | Same F06 dependency. Small: retrieval, cited generation and grounding check (L). |
| US-AI-07 | PARTIAL | PASS | PASS | PASS | PASS | PARTIAL | Independent: the route reason comes from F05. Testable: "no invented facts" is checked by the grounding check and a manual review of 50 summaries. |
| US-AI-08 | PASS | PASS | PASS | PASS | PASS | PASS | Re-uses the summary generator of US-AI-07. |
| US-AI-09 | PASS | NEEDS CONFIRM | PASS | PARTIAL | PASS | PARTIAL | Negotiable: depends on Q-AI-1 (who sets the initial priority). Estimable: the critical-system list must be provided. Testable on ES-AI-01 priority labels. |
| US-AI-10 | PARTIAL | PASS | PASS | PASS | PASS | PASS | Saving the priority is F03; re-binding the SLA is F05 (BR-WF-08). |
| US-AI-11 | PARTIAL | PASS | PASS | PARTIAL | PASS | PASS | Needs F03's link operation. Duplicate threshold calibrated on ES-AI-03. |
| US-AI-12 | PASS | PASS | PASS | PASS | PASS | PASS | Compares only with the requester's own open tickets, so no data leak. |
| US-AI-13 | PASS | PASS | PASS | PASS | PASS | PASS | A correction never re-routes by itself (UC-AI-07). |
| US-AI-14 | PASS | PASS | PASS | PASS | PASS | PASS | Votes on relevance for this ticket; article quality ratings stay in F06. |
| US-AI-15 | PARTIAL | PASS | PASS | PASS | PARTIAL | PASS | Independent: needs F03 to create tickets from the mailbox and F09's directory. Small: email cleaning, requester context and field filling in one story (L). Testable on ES-AI-06. |
| US-AI-16 | PARTIAL | PASS | PASS | PASS | PASS | PARTIAL | Independent: needs F03's Resolve dialog. Testable: "no invented facts" by manual review, as for the summary. |
| US-AI-17 | PARTIAL | PASS | PASS | PARTIAL | PARTIAL | PASS | Independent: needs F05 to add `AUTO_RESOLVE_KB` and the transitions `NEW` → `RESOLVED`, `RESOLVED` → `ASSIGNED` (Q-AI-10). Estimable: the 0.85 threshold is calibrated on ES-AI-02; F06 must mark self-service articles. Small: grounded reply, state change and escalation path (L). |
| US-AI-18 | PASS | PASS | PASS | PASS | PASS | PARTIAL | Testable: brief quality needs a manual review by the dev team. |
| US-AI-19 | PARTIAL | PASS | PASS | PASS | PASS | PASS | Independent: needs corrections from US-AI-13. Measured by running ES-AI-01 with and without examples. |
| US-AI-20 | PARTIAL | PASS | PASS | PARTIAL | PARTIAL | PARTIAL | Agent-first is decided (D-AI-01). Estimable: each domain needs its form template and KB scope from the domain team. Small: router hand-off, two agent profiles and the three escalation classes (L). Testable per domain on ES-AI-01; escalation class on ES-AI-07. |
| US-AI-21 | PARTIAL | PASS | PASS | PARTIAL | PASS | PASS | Independent: needs similar-ticket search (US-AI-11). Estimable: "5 tickets in 60 minutes" is a default to calibrate on real volume per system. |

## Shared acceptance constraints

- Only the roles `ADMIN`, `TECHNICIAN`, `REQUESTER` exist; the only system actor is the **AI Service** (`AI_AGENT`).
- The AI **suggests**; F05 decides the flow and the policy check decides what may run. Nothing in F04 changes an external system.
- Every AI output is grounded: an extracted value, a summary fact or a solution step has a source in the ticket or the KB (BR-AI-03, BR-AI-05).
- Ticket text is untrusted data; instructions inside it are never followed (BR-AI-04).
- Credentials and OTP codes are masked before any model call and in every stored output (BR-AI-07).
- Every AI output and every human correction is logged with source `AI` or `USER` and the `analysisId` (BR-WF-05).
- API errors use `ProblemDetails` (`400`, `403`, `404`, `409`, `422`). Times are stored in UTC.
- The design must hold under the reference load of **8,000 tickets per day**, about 35 per minute at peak (F05 NFR).

**Related:** [Use cases](02-use-cases.md) · [Acceptance criteria](04-acceptance-criteria.md) · [Business rules](05-business-rules.md)
