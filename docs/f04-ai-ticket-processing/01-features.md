# Features — F04 AI Ticket Processing

> Design status: TARGET V1
>
> Related: [Use cases](02-use-cases.md) · [User stories](03-user-stories-invest.md) · [Acceptance criteria](04-acceptance-criteria.md)

## Context

The group's IT division supports every company in the group: **Level 1** (computer setup, accounts and access, email, phone) and **Level 2** (software that is slow or hangs, how-to questions, business requirements for new software). About **8,000 tickets per day** arrive, most of them by **email** to the support mailbox. Tickets go to IT Helpdesk or to one of the application teams: AI Solutions, SAP, Fleet Management, EV Charging, Real Estate, Healthcare, Education.

F05 reads every ticket, classifies it and routes it. When F05 routes a ticket to the **AI agent** of its domain, F04 handles it.

**V1 goal:** the agent resolves the ticket without a technician wherever it can, and leaves a clear record when it cannot. Requesters get a cited answer, step-by-step guidance, or the access they asked for within minutes, followed by a clear reply. Technicians get only what the agents cannot resolve, system bugs and software requirements, with what the agent already tried. Every resolved ticket gets a Resolution, and verified tickets make the agents better over time.

## Feature list

The features follow the eight leaves of the **AI Ticket Processing** branch of the mind map.

| ID | Feature | Main user | MoSCoW |
|---|---|---|---|
| F04.1 | **Propose resolution from the KB, with sources.** Self-help articles on the create form, on the ticket and in the acknowledgement email; cited solution steps for the technician; the agent's KB auto-reply for how-to tickets and problems with a self-service fix; "no matching article" instead of an invented answer | Requester, Technician | Must |
| F04.2 | **Auto-resolve L1 requests (agent first, D-AI-01).** The domain agent receives the routed ticket, fills its domain form, and chooses how to resolve it: propose an allowed action, answer from the KB, or start guided troubleshooting. It escalates with exactly one class when it cannot: `AGENT_CANNOT_RESOLVE`, `SYSTEM_BUG`, `SOFTWARE_REQUIREMENT` | Requester, Technician | Must |
| F04.3 | **Guide the requester through L2 troubleshooting.** For slow, hanging or failing business software, the agent walks the requester through the checks and fixes of a self-service KB procedure, one step at a time, reads the answers, and stops to escalate with what was tried and what was found | Requester, Technician | Should |
| F04.4 | **Execute approved requests via the API.** Once a request is approved (by the F05 policy check or by an Administrator in F03), run the whitelisted operation through the connector exactly once, retry transient errors, and revoke time-bound access on expiry | Requester, Admin | Must |
| F04.5 | **Verify the resolution result.** Check the target system before the ticket is marked resolved; on failure, report the error, the failed step and the attempts so F05 can hand the ticket to a technician, who can retry or resolve manually | Requester, Technician | Must |
| F04.6 | **Fill the Resolution and reply to the requester.** After a verified execution, a KB answer or a guided fix, the AI writes the Resolution and sends its own reply; when a technician resolves a ticket, the AI drafts both for the technician to confirm | Technician, Requester | Must |
| F04.7 | **Structure a business requirement for the dev team.** Turn a free-text business request into a requirement brief, confirmed by the requester, sent with the ticket to the domain's dev team | Requester, Technician | Should |
| F04.8 | **Learn from verified tickets and technician feedback.** Votes on AI answers and steps, how much technicians change the drafts, and closed tickets that were not reopened become examples for the agents | Technician, Admin | Should |

## Out of scope for F04

| Item | Where it belongs |
|---|---|
| Receiving the mailbox, creating tickets, threading replies, sending emails | F03 Ticket Management |
| Reading and cleaning the ticket, requester context, classification, support level, filling the ticket fields | F05 Workflow & Automation (moved from F04, see [Handover](README.md#handover-to-other-branches)) |
| Priority, SLA, duplicate detection, holding the ticket and asking the requester | F05 |
| Routing to a group, a sub-group, an AI agent or a technician; re-routing after a failure; assignment | F05 |
| Policy check and auto-approval | F05 UC-WF-07 |
| Human approval of access-provisioning requests | F03 Ticket Management |
| Whitelist, entitlement catalog, connectors, AI action settings | F05 UC-WF-03 |
| Technician summary on handoff and approval | F03 Ticket Management |
| Status changes, attachments, linking and merging tickets, storing the Resolution | F03 |
| Writing, publishing, retiring and rating KB articles; drafting articles from resolved tickets | F06 Knowledge Base |
| Reports on AI accuracy and self-resolution rate | F07 Reports & Analytics (F04 records the data) |
| AI service account and API key | F01 Authentication & Authorization |
| Integration with ManageEngine | Not specified yet (Q-AI-7) |
| Reading images and attachments (OCR) | Future work (Q-AI-2) |
| Model fine-tuning and retraining | Future work; V1 improves through retrieved examples (F04.8) |
