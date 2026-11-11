# Features — F07 Reports & Analytics

> Design status: TARGET V1
>
> Related: [Use cases](02-use-cases.md) · [User stories](03-user-stories-invest.md) · [Acceptance criteria](04-acceptance-criteria.md)

## Context

About **8,000 tickets per day** arrive, most by email. F04 domain agents and F05 automation resolve what they can; technicians handle only what agents cannot resolve, system bugs and software requirements (F04 D-AI-01). Management needs to know, every week and with one definition per figure: how many tickets came in and where they went, how fast they were handled and whether the SLA held, how many the AI really closed without a person, whether the AI output is getting better, whether requesters are satisfied, and whether the teams are overloaded.

**V1 goal:** one Reports page with agreed metric definitions, limited to what each viewer may see, exportable without leaking sensitive data, and a weekly digest for management.

## Feature list

| ID | Feature | Main user | MoSCoW |
|---|---|---|---|
| F07.1 | **Ticket volume and status.** New, open (backlog), closed, cancelled, reopened, escalated; by type, `domain/subcategory`, group, priority P1–P4, channel (email / portal); daily / weekly / monthly trend | Admin, Technician (scope) | Must |
| F07.2 | **Handling time and SLA.** First response (human and AI separately), resolution time (mean, median, P90), SLA breach rate read from F05, breached and at-risk lists | Admin, Technician (scope) | Must |
| F07.3 | **AI self-resolution (agent first).** Share of tickets closed with no technician, split by path (automation, agent KB answer, requester self-help) and by domain agent; under-observation count; weekly line against the target (Q-AI-14) | Admin | Must |
| F07.4 | **Routing and escalation.** Route A / B / C share and route C reasons (F05); escalated tickets by `escalationClass` (`AGENT_CANNOT_RESOLVE`, `SYSTEM_BUG`, `SOFTWARE_REQUIREMENT`) | Admin | Must |
| F07.5 | **AI output quality.** Classification corrected by a person, helpful votes on summaries and KB suggestions, invalid / failed analyses; by model and prompt version | Admin | Should |
| F07.6 | **Requester satisfaction.** Average 1–5, distribution, response rate shown next to the average, low ratings list | Admin, Technician (scope) | Should |
| F07.7 | **Team workload.** Open tickets per technician and group, assigned vs resolved per period, oldest open ticket | Admin, Team Lead (own team) | Should |
| F07.8 | **Filter and export.** CSV / XLSX of the report on screen, allowlisted columns only, every export logged | Admin | Must |
| F07.9 | **Weekly digest and parameters.** Weekly email with the key figures; report parameters (observation window N, thresholds, limits, target) | Admin | Could (digest), Must (parameters) |

## Out of scope for F07

| Item | Where it belongs |
|---|---|
| Live home-page counters and widgets | F02 Dashboard (calls the same report API) |
| SLA targets, warning threshold, business calendar, breach detection | F05 UC-WF-02, UC-WF-06 |
| Producing AI decisions, analyses, corrections and votes | F04, F05 (F07 only reads them) |
| Browsing the audit log entry by entry | F08 Activity Log |
| Changing any ticket, SLA or AI setting from a report | Nowhere: reports are read-only (BR-BC-03) |
| Users, groups, the `is_lead` flag | F01 / F09 |
| PDF export | Not in V1 (Q-BC-11) |
| Forecasting and custom report builder | Future work |
