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
- **Related US:** US-WF-01 … US-WF-20
- **Related API:** all mutating operations
- **Rule:** Every audit record has `source` ∈ {`USER`, `AI`, `AUTOMATION`}. `USER` is tied to a user account; `AI` is the AI Service actor; `AUTOMATION` is an internal system task (rules, timers, API execution) — **not an actor**. `AI`/`AUTOMATION` records must include their grounds (`decisionId`, `ruleId`, `slaPolicyVersion` or `runId`).

## BR-WF-06 — Confidence threshold and clarification

- **Source:** UC-WF-04, AC-UC-WF-04
- **Related US:** US-WF-05, US-WF-09
- **Related API:** `updateAiActionSetting`
- **Rule:** Confidence below the threshold or missing required information → the AI asks the requester. Maximum **2 rounds** (proposed value, configurable by Admin); still not passing → route to a technician.

## BR-WF-07 — SLA pause

- **Source:** AC-UC-WF-02
- **Related US:** US-WF-04
- **Related API:** internal (SLA engine)
- **Rule:** The SLA clock pauses while the ticket is in `NEEDS_INFO` and resumes when the requester answers. In `TRIAGE` the clock keeps running.

## BR-WF-08 — SLA policy versioning

- **Source:** UC-WF-02
- **Related US:** US-WF-03
- **Related API:** `updateSlaPolicy`
- **Rule:** Every save creates a new version. A ticket is bound to the version in force when it was created, and only switches version when its priority changes.

## BR-WF-09 — Assignment rule order

- **Source:** AC-UC-WF-01, AC-UC-WF-05
- **Related US:** US-WF-01, US-WF-02, US-WF-10
- **Related API:** `reorderAssignmentRules`, `dryRunAssignmentRules`
- **Rule:** Enabled rules are evaluated in ascending order; the first matching rule wins. No rule matches → `TRIAGE`.

## BR-WF-10 — Available technicians

- **Source:** AC-UC-WF-05
- **Related US:** US-WF-10
- **Related API:** internal (assignment engine)
- **Rule:** Tickets are only assigned to members of the target team whose account is active and who are not marked absent.

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
