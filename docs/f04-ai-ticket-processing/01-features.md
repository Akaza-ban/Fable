# Features — F04 AI Ticket Processing

> Design status: TARGET V1
>
> Related: [Use cases](02-use-cases.md) · [User stories](03-user-stories-invest.md) · [Acceptance criteria](04-acceptance-criteria.md)

## Context

The group's IT division supports every company in the group: **Level 1** (computer setup, accounts and access, email, phone) and **Level 2** (software that is slow or hangs, how-to questions, business requirements for new software). About **8,000 tickets per day** arrive, most of them by **email** to the support mailbox: a subject, a sender, and a free-text body, often with signatures, disclaimers and quoted threads, mostly in Vietnamese. Today keywords in the subject route them step by step from a large group to a sub-group to a person. Tickets go to IT Helpdesk or to one of the application teams: AI Solutions, SAP, Fleet Management, EV Charging, Real Estate, Healthcare, Education.

**V1 goal:** the AI reads every ticket within seconds and returns a structured result that F05 can act on safely. That result covers who is asking and from where, what they need, which team it belongs to, what level and priority it is, and what is missing. Requesters get a fix from the KB, or one precise question, instead of waiting. Technicians get a ticket that is already summarized, prioritized and linked to similar cases, with the Resolution and the reply drafted for them. Verified tickets are reused, so the AI gets better over time.

## Feature list

| ID | Feature | Main user | MoSCoW |
|---|---|---|---|
| F04.1 | **Read and classify the ticket, fill its fields, suggest routing.** Clean email bodies; identify sender, creator and the person the request is for, with business unit, department and site; request type, `domain/subcategory`, support level, extracted fields, confidence, missing fields, suggested route and reason, in a fixed output format | AI Service → F05 | Must |
| F04.2 | **Ask back when context is missing.** One specific question naming the missing information, in the language of the ticket; the answer (portal or email reply) is read back into the fields | Requester | Must |
| F04.3 | **Suggest solutions from the KB, with sources.** Self-help articles in the portal and in the acknowledgement email; cited solution steps for the technician; the domain agent's auto-reply that resolves how-to tickets and problems with a self-service fix; "no matching article" instead of an invented answer | Requester, Technician | Must |
| F04.4 | **Summarize the ticket for technicians.** Problem, impact, extracted data, what was tried, why it came to a human; refreshed when the conversation changes | Technician | Must |
| F04.5 | **Suggest a priority.** P1–P4 with the reason (scope of impact, urgency, blocked work) | Technician | Should |
| F04.6 | **Suggest duplicate / similar tickets.** Open duplicates, past tickets with their resolution, warning to the requester before submitting a repeat | Technician, Requester | Should (requester warning: Could) |
| F04.7 | **Review, feedback and learning.** Correct category and fields, mark suggestions helpful or not; corrections and verified tickets become examples for later classification | Technician, Admin | Should |
| F04.8 | **Draft the Resolution and the reply.** When a technician resolves a ticket, or an automation succeeds, the AI drafts the Resolution field and the reply to the requester | Technician, Requester | Must |
| F04.9 | **Structure a business requirement.** Turn a free-text business request into a requirement brief for the dev team, confirmed by the requester | Requester, Technician | Should |
| F04.10 | **Domain AI agents, agent-first (D-AI-01).** The router sends each ticket to the AI agent of its domain (Helpdesk, SAP, …), which fills the domain form from free text and resolves the ticket itself: an allowed action through F05, a cited KB answer, or a question. A technician gets the ticket only when the agent cannot resolve it, when it is a system bug, or when it is a software requirement. Replaces "pick a category, fill a form" | Requester, Technician | Must |
| F04.11 | **Requester context.** Who sent it, who created it, who it is for, and their business unit, department and site, resolved from the directory | AI Service → F05 | Must (part of F04.1) |

## Out of scope for F04

| Item | Where it belongs |
|---|---|
| Receiving the mailbox, creating tickets from emails, threading replies, sending emails | F03 Ticket Management |
| Choosing the group and the person; the A / B / C flow; confidence threshold; number of ask-back rounds | F05 UC-WF-01, UC-WF-03 … UC-WF-05, BR-WF-06 (Q-AI-9) |
| Policy check, auto-approval, API execution, verification | F05 UC-WF-07 … UC-WF-09 |
| Turning AI actions on and off | F05 UC-WF-03 (`AUTO_CLASSIFY`, `ASK_FOR_INFO`, `SUGGEST_KB`, `SUGGEST_PRIORITY`; proposed `AUTO_RESOLVE_KB`, Q-AI-10) |
| SLA policies, pausing the SLA while a ticket is on hold | F05 UC-WF-02, BR-WF-07 |
| Status changes, attachments, linking and merging tickets, storing the Resolution | F03 Ticket Management |
| Writing, publishing, retiring and rating KB articles; drafting articles from resolved tickets | F06 Knowledge Base |
| Reports on AI accuracy, self-resolution rate | F07 Reports & Analytics (F04 records the data) |
| AI service account and API key | F01 Authentication & Authorization |
| Groups, their responsibility categories, members, skills; AI suggestions for group categories and skills | F09 / F01 (UC-08, UC-15, UC-16 on the `feature/author` branch) |
| Integration with ManageEngine | Not specified yet (Q-AI-7) |
| Reading images and attachments (OCR) | Future work (Q-AI-2) |
| Model fine-tuning and retraining | Future work; V1 improves through retrieved examples (F04.7) |
