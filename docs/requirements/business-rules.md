# Business Rules — Workflow & Automation (V1)

> Design status: TARGET V1
> Implementation status: NOT IMPLEMENTED

Only rules confirmed by project documentation are catalogued here.

## BR-WF-01 — Only whitelisted actions are executed automatically

- **Source:** `03-use-cases.md` UC-WF-03, UC-WF-08
- **Related US:** US-WF-06, US-WF-17
- **Related API:** `createAutomationAction`, `updateAutomationAction`, internal execution
- **Rule:** The system only calls connectors automatically for enabled whitelist entries. V1 includes `CHECK_ACCOUNT`, `CREATE_ACCOUNT`, `GRANT_ACCESS`, `REVOKE_ACCESS`. Every other request is routed to a technician.

## BR-WF-02 — The AI approves only when the deterministic policy check passes

- **Source:** UC-WF-07, AC-UC-WF-07
- **Related US:** US-WF-07, US-WF-15
- **Related API:** internal (AI decision), `updateDepartmentEntitlements`
- **Rule:** The AI Service may self-approve basic requests, but only when **all** policy conditions (evaluated without AI) pass: operation in an enabled whitelist entry; entitlement / account template in the beneficiary department's catalog; duration ≤ maximum; valid beneficiary (BR-WF-16). The AI may never approve outside this scope, however high its confidence.

## BR-WF-03 — Requests that need a human always go to a technician

- **Source:** UC-WF-04, UC-WF-05
- **Related US:** US-WF-10
- **Related API:** internal (routing)
- **Rule:** Hardware requests (mouse, keyboard, PC repair…), requests outside the whitelist, requests failing the policy check, and requests still unclear after the maximum clarification rounds are always routed to a technician. The AI does not perform them; it only summarizes and states the routing reason.

## BR-WF-04 — Idempotent execution

- **Source:** AC-UC-WF-08, AC-UC-WF-09
- **Related US:** US-WF-17, US-WF-20
- **Related API:** `retryWorkflowRun`, connectors
- **Rule:** Idempotency key = `decisionId + action`. One approval decision leads to at most one change in the external access-management system, even when retried.

## BR-WF-05 — Action source in the audit log

- **Source:** `08-invest-requirements.md` cross-cutting constraints
- **Related US:** US-WF-03 … US-WF-24 (US-WF-01, 02 removed)
- **Related API:** all mutating operations
- **Rule:** Every audit record has `source` ∈ {`USER`, `AI`, `AUTOMATION`}. `USER` is tied to a user account; `AI` is the AI Service actor; `AUTOMATION` is an internal system task (rules, timers, API execution) — **not an actor**. `AI`/`AUTOMATION` records must include their grounds (`decisionId`, the category and team chosen, `slaPolicyVersion` or `runId`).

## BR-WF-06 — Confidence threshold and clarification

- **Source:** UC-WF-04, AC-UC-WF-04
- **Related US:** US-WF-05, US-WF-09
- **Related API:** `updateAiActionSetting`
- **Rule:** Confidence below the threshold or missing required information → the AI asks the requester. Maximum **2 rounds** (proposed value, configurable by Admin); still not passing → route to a technician.

## BR-WF-07 — SLA pause

- **Source:** AC-UC-WF-02
- **Related US:** US-WF-04
- **Related API:** internal (SLA engine)
- **Rule:** The SLA clock pauses while the ticket is in `NEEDS_INFO` and resumes when the requester answers. While a ticket waits in a team queue (`ASSIGNED` to a team, no technician yet) the clock keeps running.

## BR-WF-08 — SLA policy versioning

- **Source:** UC-WF-02
- **Related US:** US-WF-03
- **Related API:** `updateSlaPolicy`
- **Rule:** Every save creates a new version. A ticket is bound to the version in force when it was created, and only switches version when its priority changes.

## BR-WF-09 — Group, sub-group and owner from the AI category

- **Source:** UC-WF-04, UC-WF-05, AC-UC-WF-04, AC-UC-WF-05
- **Related US:** US-WF-10, US-WF-21, US-WF-24
- **Related API:** internal (assignment engine), `previewAssignment`
- **Rule:** There are no configurable assignment rules. The AI category is `domain/subcategory` (F04). The ticket goes to the **group** that owns its domain and, when one exists, the **sub-group** that handles its subcategory; groups and sub-groups declare their domains and subcategories in F09 User Management, and each domain or subcategory belongs to at most one group or sub-group. When the domain has an AI agent and `AGENT_FIRST` is on, that **AI agent** owns the ticket in flows A and B; in flow C the owner is a **technician** of the sub-group, or of the group when no sub-group handles the subcategory. If no group owns the domain, or the AI could not classify the ticket, the ticket goes to the configured **default support team**. There is no manual triage queue.

## BR-WF-19 — Routing uses the full AI analysis

- **Source:** UC-WF-04, AC-UC-WF-04
- **Related US:** US-WF-08, US-WF-22, US-WF-24
- **Related API:** internal (routing)
- **Rule:** The routing decision uses only the current F04 analysis of the whole ticket: request (type, category, support level, extracted and missing fields), requester (department, business unit, site, beneficiary) and source (`channel` `EMAIL` / `PORTAL`, `createdBy`). The source and the support level are stored on the ticket. A ticket whose sender is unknown is never routed to flow A.

## BR-WF-20 — Priority is set automatically

- **Source:** UC-WF-04, AC-UC-WF-04
- **Related US:** US-WF-23
- **Related API:** `updateAiActionSetting`, `changeTicketPriority`
- **Rule:** With `SET_PRIORITY` on, the system applies the AI priority (based on impact and urgency only, BR-AI-09) when its confidence ≥ the threshold, otherwise the configured default priority (P2). The SLA of that priority is attached in the same step. A technician or the Admin may change the priority only with a reason; the SLA is then recalculated (BR-WF-08) and the change is audited.

## BR-WF-21 — Fields filled by AI are marked and correctable

- **Source:** UC-WF-04, AC-UC-WF-04
- **Related US:** US-WF-22
- **Related API:** internal (routing), `correctTicketField`
- **Rule:** With `AUTO_FILL_FIELDS` on, the system writes request type, category, subcategory, support level, requested for, business unit, site and priority from the AI analysis, each marked "Filled by AI". A field below the confidence threshold is left empty and asked for (flow B). A technician or the Admin may correct a field only with a reason; a change of category or support level before work starts re-routes the ticket.

## BR-WF-10 — Eligible technicians

- **Source:** AC-UC-WF-05
- **Related US:** US-WF-10, US-WF-11
- **Related API:** internal (assignment engine)
- **Rule:** A technician has exactly two statuses, `ON` (taking work) and `OFF` (not taking work). A new ticket is only assigned to a member of the target team whose account is active, whose status is `ON` and whose workload is below the maximum load. If nobody is eligible, the ticket waits in the team queue. When a technician switches to `OFF`, their not-started (`ASSIGNED`) tickets are reassigned; `IN_PROGRESS` tickets stay with them. There is no Team Lead; only the Admin may reassign manually, with a reason.

## BR-WF-11 — Automating tickets are never auto-closed

- **Source:** AC-UC-WF-06
- **Related US:** US-WF-14
- **Related API:** internal (scheduler)
- **Rule:** Auto-close only applies to overdue `NEEDS_INFO` tickets; `AUTOMATING` tickets are never closed automatically.

## BR-WF-12 — Policy is re-checked at execution time

- **Source:** UC-WF-08, UC-WF-09
- **Related US:** US-WF-17, US-WF-20
- **Related API:** `retryWorkflowRun`
- **Rule:** Every execution or retry re-checks the **current** whitelist and entitlement catalog. If the Admin narrowed the scope after the AI approved, the execution is blocked.

## BR-WF-13 — Verification before marking resolved

- **Source:** AC-UC-WF-09
- **Related US:** US-WF-19, US-WF-20
- **Related API:** `resolveWorkflowRunManually`, `confirmTicketResolution`
- **Rule:** An automated ticket only becomes `RESOLVED` when the connector's check operation succeeds or a technician records a manual resolution with a note.

## BR-WF-14 — Revocation of time-bound access

- **Source:** UC-WF-08
- **Related US:** US-WF-18
- **Related API:** internal (scheduler)
- **Rule:** Access granted with `expires_at` is revoked automatically on expiry through a `REVOKE_ACCESS` `workflow_run`; the revocation must be audited.

## BR-WF-15 — Protection of connector secrets

- **Source:** AC-UC-WF-03, AC-UC-WF-08
- **Related US:** US-WF-06, US-WF-17
- **Related API:** all whitelist and workflow-run operations
- **Rule:** Connector credentials only live in deployment secret configuration. Payloads stored in `workflow_run_attempts` must mask tokens, passwords and keys.

## BR-WF-16 — Valid beneficiary for AI self-approval

- **Source:** UC-WF-07, AC-UC-WF-07
- **Related US:** US-WF-15
- **Related API:** internal (AI decision)
- **Rule:** The AI only self-approves when the beneficiary of the access / account is the requester, or belongs to the same department as the requester. Requests for users in other departments are always routed to a technician.

## BR-WF-17 — AI decision grounds are stored

- **Source:** UC-WF-04, UC-WF-07
- **Related US:** US-WF-16
- **Related API:** `listAiDecisions`, `getAiDecision`
- **Rule:** Each AI decision stores confidence, extracted information, the result of each policy condition, the configuration version and the reason. Records are immutable once created.

## BR-WF-18 — Assignment by workload score

- **Source:** UC-WF-01, UC-WF-05, AC-UC-WF-05
- **Related US:** US-WF-10, US-WF-11, US-WF-21
- **Related API:** `updateWorkloadSettings`, internal (assignment engine)
- **Rule:** Among eligible technicians (BR-WF-10), the ticket goes to the one with the lowest workload score; ties go to the one who received a ticket least recently. Workload score = Σ (priority points × complexity factor) over the technician's `ASSIGNED` and `IN_PROGRESS` tickets. The system has three priority levels (no "critical" level). Defaults: P1 High = 3, P2 Medium = 2, P3 Low = 1; Simple ×1, Medium ×1.5, Complex ×2; maximum load 20. In V1 the complexity comes from the request type configured by the Admin, not from the AI. The number of tickets alone is never used to choose a technician.
