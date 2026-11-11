# User Stories (INVEST) — F04 AI Ticket Processing

> Design status: TARGET V1
>
> Scope: US-AI-05, 06, 14, 16 … 28. US-AI-01 … 04, 07 … 13 and 15 are retired: see [Handover](README.md#handover-to-other-branches).

Each story is small enough to build and test on its own, or its review says why not. Authorization, ticket scope, credential masking, grounding and audit logging are **shared acceptance constraints**, not hidden features.

## INVEST checklist

| Principle | Application in this backlog |
|---|---|
| **Independent** | Each story produces a demonstrable result. A story that needs another to be meaningful says so in its review (`PARTIAL`). |
| **Negotiable** | Screens, model and prompt choices may change, while the outcome, permissions and AI guardrails stay fixed. |
| **Valuable** | Every story names the benefit for a role. System behaviour is written for the role that benefits from it. |
| **Estimable** | Inputs, outputs, roles, limits and failure cases (including AI and connector failure) are explicit, and every story has a size. |
| **Small** | One capability per story. **S** = one screen element or rule; **M** = a flow with several rules; **L** = AI or connector integration with validation and several checks. |
| **Testable** | Every story has Given / When / Then criteria in [04-acceptance-criteria.md](04-acceptance-criteria.md). AI quality is measured on named evaluation sets (ES-AI-02, ES-AI-04, ES-AI-07, ES-AI-08); execution is tested against a mock connector. |

## Product backlog

MoSCoW: **Must** (required for V1), **Should** (planned, but the core works without it), **Could** (if time allows).

| ID | User story | Value | Dependencies | MoSCoW | Size | AC |
|---|---|---|---|---|---|---|
| US-AI-05 | As a **Requester**, I want to see up to 3 knowledge-base articles that match my problem while I write the ticket and after I submit it, and to mark one as the fix, so that I can solve common problems myself without waiting. | Self-service resolution | F06 published articles | Must | M | AC-UC-AI-03 |
| US-AI-06 | As a **Technician**, I want suggested solution steps where every step cites the KB article it comes from, so that I can check and reuse a known fix quickly instead of searching the KB. | Faster, verifiable fixes | F06 published articles | Must | L | AC-UC-AI-03 |
| US-AI-17 | As a **Requester** with a how-to question or a problem that has a known self-service fix, I want the domain agent to answer my ticket with cited steps from the KB and let me confirm or say it did not help, so that I get an answer in minutes and still reach a person if needed. | Tickets solved without a technician | US-AI-05, US-AI-20, F05 `AUTO_RESOLVE_KB` and transitions (Q-AI-10) | Must | L | AC-UC-AI-03 |
| US-AI-20 | As a **Requester**, I want my ticket, once routed to the AI agent of its domain (Helpdesk, SAP, …), to be handled by that agent, which fills the domain form and resolves the ticket itself (allowed action, KB answer or guided troubleshooting), and as a **Technician** I want to receive a ticket only when the agent cannot resolve it, it is a system bug, or it is a software requirement, always with the reason, so that people work only on what needs people. | Most tickets resolved without a technician | F05 routing to the agent, F05 whitelist | Must | L | AC-UC-AI-10 |
| US-AI-21 | As a **Technician** of a domain team, I want tickets that point to a defect in our system flagged as a suspected bug, with similar tickets from other users linked, so that we fix the system once instead of answering each user separately. | Bugs fixed at the source | US-AI-20, F05 similar-ticket detection | Should | M | AC-UC-AI-10 |
| US-AI-26 | As a **Requester** whose business software is slow, hangs or shows an error, I want the agent to guide me through the checks and fixes one step at a time, with the source of each step, and to understand my answers in my own words, so that I can fix common problems myself within minutes. | L2 problems fixed without a technician | US-AI-20, F06 troubleshooting articles, F05 waiting state (Q-AI-15) | Should | L | AC-UC-AI-11 |
| US-AI-27 | As a **Technician**, I want a ticket where guidance did not fix the problem to reach me with every step done, the requester's answers and the findings, so that I do not repeat the same checks. | No repeated work | US-AI-26 | Should | S | AC-UC-AI-11 |
| US-AI-22 | As a **Requester**, I want my approved request to be executed through the API automatically, with no duplicate grants and automatic retries on temporary errors, so that I get my account or access within minutes without anyone doing it by hand. | Self-resolution without technicians | F05 approval (UC-WF-07) or F03 approval, F05 whitelist and connectors | Must | L | AC-UC-AI-12 |
| US-AI-23 | As an **Admin**, I want time-bound access to be revoked automatically on expiry, with the user told the day before, so that stale access does not accumulate. | Security | US-AI-22 | Could | S | AC-UC-AI-12 |
| US-AI-24 | As a **Requester**, I want the system to check the result on the target system before it tells me my request is done and asks me to confirm, so that I am sure it was really fulfilled. | Trustworthy result | US-AI-22 | Must | M | AC-UC-AI-13 |
| US-AI-25 | As a **Technician**, I want a ticket whose execution or verification failed to reach me with the error code, the failed step and the attempts, and to retry it or resolve it manually, so that no automation failure is forgotten. | Clear ownership of failures | US-AI-22, F05 re-routing | Must | M | AC-UC-AI-13 |
| US-AI-16 | As a **Requester**, I want a clear reply and a recorded Resolution when the AI resolves my ticket, and as a **Technician** I want the AI to draft both when I resolve a ticket, so that every ticket closes with a clear answer without extra typing. | Every ticket has a Resolution | US-AI-24, F03 Resolve dialog | Must | M | AC-UC-AI-08 |
| US-AI-18 | As a **Requester** with a business need, I want the AI to turn my request into a requirement brief that I can check and confirm, so that the dev team understands what to build without several rounds of emails. | Clear input for the dev team | US-AI-20, F05 hold and ask | Should | M | AC-UC-AI-09 |
| US-AI-14 | As a **Technician**, I want to mark cited steps, KB answers and guidance as helpful or not, correct the agent's domain form, and flag escalations the agent could have resolved, so that the team can see which AI outputs work and fix the ones that do not. | Measured AI quality (F07) | US-AI-06, US-AI-20 | Should | S | AC-UC-AI-07 |
| US-AI-28 | As a **Technician** of a domain team, I want to review the tickets that our domain's agent resolved without a technician and flag one as resolved incorrectly with a reason, so that wrong answers and wrong grants are caught, reported to the Admin, and never used for learning. | Mistakes of the agent are caught | US-AI-20, F01 / F03 review scope | Should | M | AC-UC-AI-07 |
| US-AI-19 | As an **Admin**, I want verified tickets and their resolutions to be reused as examples when the agents handle new tickets, and to exclude a bad example, so that the agents improve from real tickets without retraining a model. | AI improves over time | US-AI-14 | Should | M | AC-UC-AI-07 |

Grouped by use case, in the order of the mind-map leaves:

```text
US-AI-05–06, 17   UC-AI-03  Propose resolution from the KB with sources
US-AI-20–21       UC-AI-10  Domain agent handles the ticket (auto-resolve L1)
US-AI-26–27       UC-AI-11  Guide L2 troubleshooting
US-AI-22–23       UC-AI-12  Execute approved request via API
US-AI-24–25       UC-AI-13  Verify result, report failure
US-AI-16          UC-AI-08  Fill the Resolution and reply
US-AI-18          UC-AI-09  Structure a business requirement
US-AI-14, 19, 28  UC-AI-07  Learn from verified tickets and feedback
```

US-AI-22 … 25 replace F05's US-WF-17 … 20 (D-AI-02).

## MoSCoW summary

| Priority | Stories | Count |
|---|---|---|
| **Must**: agent-first handling (D-AI-01), the KB answer, executing and verifying approved requests with a failure path, and the Resolution | US-AI-05, US-AI-06, US-AI-16, US-AI-17, US-AI-20, US-AI-22, US-AI-24, US-AI-25 | 8 |
| **Should**: important, but tickets are still resolved without them | US-AI-14, US-AI-18, US-AI-19, US-AI-21, US-AI-26, US-AI-27, US-AI-28 | 7 |
| **Could** | US-AI-23 | 1 |
| **Won't (V1)** | Reading images and attachments (OCR); remote control of the requester's computer; model fine-tuning; integration with ManageEngine (Q-AI-7) | — |

No Must story depends on a Should story. US-AI-17 needs F05 to add `AUTO_RESOLVE_KB` and the transitions (Q-AI-10), so plan it with the F05 owner. US-AI-20 is Must; V1 seeds only the `HELPDESK` and `SAP` agents (Q-AI-13). US-AI-26 is Should because it needs troubleshooting articles from F06 and a waiting state from F05 (Q-AI-15); without it, L2 incidents escalate to a technician as before. US-AI-23 stays Could, as it was in F05.

## Per-story INVEST review

`PASS` means the design gives enough information to plan and accept the story. It does not mean that code or tests exist. `PARTIAL` means the story is weak on that criterion, and the note says why.

| Story | I | N | V | E | S | T | Notes |
|---|---|---|---|---|---|---|---|
| US-AI-05 | PARTIAL | PASS | PASS | PARTIAL | PASS | PARTIAL | Independent: needs published articles from F06 (a seeded set works for the demo). Estimable: relevance threshold calibrated on ES-AI-02. Testable on ES-AI-02. |
| US-AI-06 | PARTIAL | PASS | PASS | PARTIAL | PARTIAL | PASS | Same F06 dependency. Small: retrieval, cited generation and grounding check (L). |
| US-AI-17 | PARTIAL | PASS | PASS | PARTIAL | PARTIAL | PASS | Independent: needs F05 to add `AUTO_RESOLVE_KB` and the transitions (Q-AI-10). Estimable: the 0.85 threshold is calibrated on ES-AI-02; F06 must mark self-service articles. Small: grounded reply, state change and escalation path (L). |
| US-AI-20 | PARTIAL | PASS | PASS | PARTIAL | PARTIAL | PARTIAL | Needs F05 to route to the agent and publish its routing result. Estimable: each domain needs its form template and KB scope from the domain team. Small: agent queue, two agent profiles and the three escalation classes (L). Testable on ES-AI-07. |
| US-AI-21 | PARTIAL | PASS | PASS | PARTIAL | PASS | PASS | Needs F05's similar-ticket detection. "5 tickets in 60 minutes" is a default to calibrate on real volume per system. |
| US-AI-26 | PARTIAL | NEEDS CONFIRM | PASS | PARTIAL | PARTIAL | PARTIAL | Negotiable: waiting state, round limit and timeout are proposed (Q-AI-15). Estimable: needs at least 10 troubleshooting procedures from F06 for the demo domains. Small: session, branching and escalation (L). Testable on ES-AI-08. |
| US-AI-27 | PARTIAL | PASS | PASS | PASS | PASS | PASS | Only meaningful after US-AI-26; the transcript is the session data, so no extra AI call. |
| US-AI-22 | PARTIAL | PASS | PASS | PASS | PARTIAL | PASS | Needs F05 approval and the whitelist and connector configuration. Idempotency key, 3 retries at 1/5/15 minutes; demoable with a mock connector. Human approval from F03 depends on Q-AI-16. |
| US-AI-23 | PASS | PASS | PASS | PASS | PASS | PASS | Revocation reuses the `workflow_run` mechanism. |
| US-AI-24 | PARTIAL | PASS | PASS | PASS | PASS | PASS | Connectors must expose a check operation. |
| US-AI-25 | PARTIAL | PASS | PASS | PASS | PASS | PASS | Needs F05 to re-route the failed ticket. Retry always re-checks the current policy. |
| US-AI-16 | PARTIAL | PASS | PASS | PASS | PASS | PARTIAL | Needs F03's Resolve dialog. Testable: "no invented facts" by manual review of 50 drafts. |
| US-AI-18 | PARTIAL | PASS | PASS | PASS | PASS | PARTIAL | Needs F05 to ask the missing points. Testable: brief quality needs a manual review by the dev team. |
| US-AI-14 | PASS | PASS | PASS | PASS | PASS | PASS | A correction never starts an action or re-routes the ticket (UC-AI-07). Article quality ratings stay in F06. |
| US-AI-28 | PARTIAL | PASS | PASS | PASS | PASS | PASS | Needs F01 / F03 to let technicians read tickets in the review scope (BR-AI-16). The review never changes the ticket, so it is safe to build alone. |
| US-AI-19 | PARTIAL | PASS | PASS | PASS | PASS | PASS | Needs verified tickets and feedback from US-AI-14. Measured by running ES-AI-07 with and without examples. |

## Shared acceptance constraints

- Only the roles `ADMIN`, `TECHNICIAN`, `REQUESTER` exist; the only system actor is the **AI Service** (`AI_AGENT`).
- F05 decides the route and the policy check or an Administrator (in F03) decides what may run. F04 changes an external system **only** by executing an approved decision, with exactly its parameters (BR-AI-15).
- Every AI output is grounded: a form value, a solution or guidance step, a reply or a Resolution has a source in the ticket, the KB or the run (BR-AI-03, BR-AI-05).
- Ticket text is untrusted data; instructions inside it are never followed (BR-AI-04).
- Credentials and OTP codes are masked before any model call, in every stored output and in every stored connector payload (BR-AI-07, BR-WF-15).
- Every AI output, execution attempt and human correction is logged with source `AI`, `AUTOMATION` or `USER` (BR-WF-05).
- API errors use `ProblemDetails` (`400`, `403`, `404`, `409`, `422`). Times are stored in UTC.
- The design must hold under the reference load of **8,000 tickets per day**, about 35 per minute at peak (F05 NFR).

**Related:** [Use cases](02-use-cases.md) · [Acceptance criteria](04-acceptance-criteria.md) · [Business rules](05-business-rules.md)
