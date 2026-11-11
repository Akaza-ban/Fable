# F05 Workflow & Automation — Information Architecture

> Design status: V1 — draft, awaiting user review
>
> Implementation status: NOT STARTED
>
> Scope: **F05 Workflow & Automation**. Shared screens (login, tickets, notifications) are described only where they relate to the workflow; the rest will be specified with F01 and F03.

The information architecture organizes the screens of the Workflow & Automation feature: a sitemap by role, routes and page permissions, navigation, the screen inventory (**11 pages and 9 modals**; 5 pages are workflow-specific, 6 are shared or belong to F01/F03; SCR-WF-10 and SCR-WF-04 were removed with the Team Lead and the assignment rules), six primary user flows, and the UI states (including AI states).

Inputs: [Use cases](../02-use-cases.md), [INVEST backlog](../04-invest-requirements.md), [Business rules](../../requirements/business-rules.md). Output for: UI/UX design (wireframes) and API (screen-to-API mapping).

Three roles: **Requester** (`REQUESTER`), **Technician** (`TECHNICIAN`; there is no Team Lead — work is distributed by the system from availability and workload), **Admin** (`ADMIN`). The system actor **AI Service** has no screen; its output appears on people's screens and is always labelled **AI**.

| File | Content |
|---|---|
| [00-decisions.md](00-decisions.md) | Decisions D-IA-01..D-IA-10, defaults DF-IA-01..DF-IA-08, differences from the requirements documents |
| [01-sitemap.md](01-sitemap.md) | Sitemap by role (page cards in areas) and the home page of each role |
| [02-routes-and-permissions.md](02-routes-and-permissions.md) | Routes and the page-permission table |
| [03-navigation.md](03-navigation.md) | Sidebar and top bar by role, contextual navigation, notifications |
| [04-screen-inventory.md](04-screen-inventory.md) | Screen inventory SCR-xx, US-WF-01..20 → screen mapping, screen → API mapping |
| [05-user-flows.md](05-user-flows.md) | 8 flows: AI self-service, AI clarification, human handling, automation failure handling, Admin configures and reviews the AI, Admin configures workload scoring, duplicate and similar tickets, AI re-routing on failure or misroute |
| [06-ui-states-and-accessibility.md](06-ui-states-and-accessibility.md) | UI state contract, status chips, AI and execution states, responsive and accessibility rules |

**Related:** [README](../../../README.md) · [Use cases](../02-use-cases.md) · [Acceptance criteria](../03-acceptance-criteria.md) · [Traceability](../../traceability/master-traceability.md)
