# Decisions and Defaults

> Design status: V1 — draft, awaiting user review
>
> Implementation status: NOT STARTED

Information architecture IDs carry the `IA` prefix so they do not clash with other documents: decisions **D-IA-01..D-IA-10**, defaults **DF-IA-01..DF-IA-08**. All of them are **the author's proposals**, awaiting user confirmation; none changes the meaning of a user story or business rule.

## Decisions made in this document set

| ID | Decision | Why | Status |
|---|---|---|---|
| D-IA-01 | **The sitemap shows page cards with their main content**, grouped in areas. Actors connect with dashed arrows to the areas they use; solid arrows mean parent page → child page. **Purple cards** are pages with AI features. Modals, the 403 page and loading/error states are not drawn as cards. | Follows the notation of the reference repository; keeps the sitemap readable. | Proposed |
| D-IA-02 | **Every role uses the same layout: left sidebar + top bar**, including the Requester. The sidebar shows only permitted items. | The Requester also has several pages (list, create, detail, notifications); one layout lets everyone find their name and Log out in the same place. | Proposed |
| D-IA-03 | **Detail views are pages; short forms, confirmations and decisions are modals** opened on their parent page, with no route of their own. | Modals keep the context (e.g. the technician still sees the error log when clicking Retry). | Proposed |
| D-IA-04 | **One ticket detail page (SCR-T-02) serves all three roles**, with role-dependent panels: the Requester sees the AI conversation, progress bar and confirm buttons; the Technician sees the AI summary, the routing reason and the Automation tab; the Admin sees everything plus a link to the AI decision. | Avoids three near-identical pages; the server returns only what the role may see. | Proposed |
| D-IA-05 | **The AI scope is one page with 3 tabs** (AI actions, Automation actions, Entitlement catalog), not three pages. | All three answer one question: "what may the AI do on its own?". The Admin needs to see them side by side. | Proposed |
| D-IA-06 | **AI decisions have their own pages (SCR-WF-30, SCR-WF-31), Admin only.** Technicians see only a decision summary in SCR-T-02; Requesters see only a summary reason. | US-WF-16 belongs to the Admin; policy-condition details are administrative information (DF-IA-08). | Proposed |
| D-IA-07 | **Team Lead is not a separate role.** The "Triage queue" item and the "Reassign" button appear when the technician has the `is_lead` flag in at least one team. | Consistent with the use case documents (3 roles). | Proposed |
| D-IA-08 | **The Requester talks to the AI through a conversation panel in SCR-T-02**, not a separate chat page. AI questions and requester answers are ticket messages. | An AI question always belongs to a ticket; a separate chat page would separate the requester from the ticket status. | Proposed |
| D-IA-09 | **There is a shared notification centre (SCR-91)** with an unread count in the top bar, for all three roles. | The workflow produces many events that need to be announced (AI clarification, results, SLA warnings, automation failures). | Proposed |
| D-IA-10 | **There is no "approval" screen for humans.** The AI approves in the background; the requester only sees progress, and the Admin reviews afterwards through SCR-WF-30/31. | Follows the agreed flow: basic requests are approved by the AI, without Admin approval. | Proposed |

## Defaults pending confirmation

| ID | Default | Status |
|---|---|---|
| DF-IA-01 | **Routes are logical paths.** The router mode (history or hash) is chosen in the architecture documents. Authorization is always checked by the server on every request, never by the route. | Proposed |
| DF-IA-02 | **Home page by role:** Requester → SCR-T-03 *My tickets*; Technician → SCR-T-03 *Assigned to me*; Admin → SCR-WF-30 *AI decisions* (temporary, until the F02 Dashboard is specified). | Proposed |
| DF-IA-03 | **Display formats:** date and time `dd/MM/yyyy HH:mm` in Vietnam time; AI confidence as a percentage (0.87 → `87%`); remaining SLA time as `2 h 15 min`. | Proposed |
| DF-IA-04 | **Screen IDs are numbered by area:** `SCR-01`, `SCR-9x` shared; `SCR-T-xx` tickets (F03); `SCR-WF-0x` configuration; `SCR-WF-1x` dispatch; `SCR-WF-2x` automation actions on a ticket; `SCR-WF-3x` AI monitoring. A new screen takes the next free number in its area; existing numbers do not change. | Proposed |
| DF-IA-05 | **After reassigning (SCR-WF-11)**, the user returns to the page that opened the modal (SCR-WF-10 or SCR-T-02). | Proposed |
| DF-IA-06 | **The Admin is notified when the AI Service is unavailable** (all new tickets are being routed to humans, NFR-WF-REL-004). The requirements do not say who is notified. | Proposed |
| DF-IA-07 | **SCR-T-02 refreshes its status automatically** while the ticket is `NEW`, `NEEDS_INFO` or `AUTOMATING`, so the requester sees progress without reloading. The mechanism (polling, WebSocket…) is chosen in the architecture documents. | Proposed |
| DF-IA-08 | **The Requester only sees a summary reason** for an AI decision (e.g. *"Your request needs a technician because this access is not in your department's catalog"*), not the confidence or each policy condition. | Proposed |

## Differences from or additions to the requirements documents

| Topic | What the requirements say | What the information architecture does and why |
|---|---|---|
| Screen inventory | Traceability listed 8 proposed screens, no modals. | Adds 8 modals (SCR-WF-04..07, 11, 20..22), the ticket list page SCR-T-03, the notification page SCR-91, and the shared pages SCR-01 and SCR-90. Existing IDs are kept. |
| Notifications | The use cases mention "notifications" but no screen. | Adds SCR-91 and an unread count in the top bar (D-IA-09). |
| Who is told when the AI fails | NFR-WF-REL-004 only says tickets are routed to humans. | The Admin is notified (DF-IA-06). |
| Admin home page | No Dashboard yet (F02 not specified). | SCR-WF-30 is used temporarily (DF-IA-02). |
| "Not fixed" | UC-WF-09 has `reopenTicket` but does not say whether a reason is needed. | Modal SCR-WF-22 with a mandatory reason, so the technician knows what is wrong. |

**Related:** [Information architecture index](README.md) · [Sitemap](01-sitemap.md) · [Screen inventory](04-screen-inventory.md)
