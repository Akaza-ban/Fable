# Feature Catalog

> Design status: TARGET V1
>
> Current documentation scope: only **F05 Workflow & Automation** is specified in detail; the remaining features will be added later.

System name: **AI-Powered IT Ticket Routing and Automated Resolution System**.

| ID | Name | V1 | Specification status |
|---|---|---|---|
| F01 | Authentication & Authorization | Yes | Not specified |
| F02 | Dashboard | Yes | Not specified |
| F03 | Ticket Management | Yes | Not specified |
| F04 | AI Ticket Processing | Yes | Not specified |
| **F05** | **Workflow & Automation** | Yes | **Specified (this document set)** |
| F06 | Knowledge Base — KB | Yes | Not specified |
| F07 | Reports & Analytics | Yes | Not specified |
| F08 | Activity Log | Yes | Not specified |
| F09 | User Management | Yes | Not specified |
| F10 | Personal Profile | Yes | Not specified |

## F05 Workflow & Automation

**Context:** the enterprise receives about **8,000 tickets per day** from its departments. Most of them are basic, repetitive requests (account check, account creation, additional access). Handling them manually would overload the technicians.

**V1 goal:** the AI Service approves, and the system automatically executes, basic requests **within the configured scope**; requests that need a human (hardware, out of scope) are routed to technicians; when the AI is not confident, it asks the requester for clarification.

| Request type | Examples | Handled by |
|---|---|---|
| Basic, within AI scope | Account check, account creation, additional access | **AI approves → system executes via API** |
| Needs a human | New mouse or keyboard, PC repair, complex incidents, access outside the catalog | **Technician** |
| AI not confident | Vague description, missing information | **AI asks the requester**, then re-routes |

- F05.1 Configure technician assignment rules (conditions, target team, distribution strategy, order, dry run)
- F05.2 Configure SLA policies by priority and business calendar
- F05.3 Configure the **AI processing scope**: AI actions and confidence thresholds, automation action catalog (whitelist), department entitlement catalog
- F05.4 **Ticket routing**: AI self-service / ask the requester / route to a human
- F05.5 **Automatic technician assignment by rules** + manual triage queue
- F05.6 Response reminders, SLA warnings and escalation on breach
- F05.7 **AI approval of in-scope requests** (AI understanding + deterministic policy check)
- F05.8 **API execution after approval** (idempotent, with retries, automatic revocation on expiry)
- F05.9 Result verification and hand-off to a technician on failure

**Outside F05 (owned by other features):**

- AI classification, extraction and summarization models → F04 (F05 only *configures* and *consumes* their output).
- Browsing workflow run history and errors → F08 Activity Log (F05 only *produces* the data).
- AI self-resolution rate and SLA breach statistics → F07 Reports & Analytics.
- Managing technical teams, departments and members → F09 User Management.

**Related:** [Use cases](03-use-cases.md) · [INVEST backlog](08-invest-requirements.md) · [Acceptance criteria](06-acceptance-criteria.md)
