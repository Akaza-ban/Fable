# F02 Dashboard — Information Architecture Index

> Design status: V1 — draft, awaiting user review  
> Implementation status: NOT STARTED  

This directory contains the modular Information Architecture (IA) specification suite for **F02 Dashboard** of the AI-Powered Automated Ticket Dispatching & Resolution System.

---

## Document Index

| Order | Document | Purpose & Core Contents |
|---|---|---|
| **00** | [00-decisions.md](00-decisions.md) | Architectural decisions (`D-IA-DB-01..10`), system defaults (`DF-IA-DB-01..08`), and differences from root requirements. |
| **01** | [01-sitemap.md](01-sitemap.md) | Visual sitemap diagram (Mermaid), page-level hierarchy, area groupings, and screens by role. |
| **02** | [02-routes-and-permissions.md](02-routes-and-permissions.md) | Application routes, query parameter protocols for drill-downs, and role-based page permission matrix. |
| **03** | [03-navigation.md](03-navigation.md) | Global navigation layout (Sidebar + Top bar), contextual breadcrumbs, and live incident alerting protocols. |
| **04** | [04-screen-inventory.md](04-screen-inventory.md) | Complete catalog of 6 UI surfaces (`SCR-DB-01..02`, `SCR-DB-10..13`), story traceability, and API mapping. |
| **05** | [05-user-flows.md](05-user-flows.md) | 5 end-to-end operational user flows with Mermaid sequence/flowcharts. |
| **06** | [06-ui-states-and-accessibility.md](06-ui-states-and-accessibility.md) | UI state contract (loading, empty, errors, live/polling), badge telemetry taxonomy, and WCAG 2.1 AA accessibility guidelines. |

---

## Related Feature Specifications
* [01-user-stories.md](../01-user-stories.md) — Agile Epics, user personas, and BDD acceptance criteria.
* [02-use-cases.md](../02-use-cases.md) — Use cases and system interaction flows.
* [03-information-architecture.md](../03-information-architecture.md) — Unified overview document.

