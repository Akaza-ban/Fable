# Acceptance Criteria — Workflow & Automation (V1)

> Design status: TARGET V1
>
> Implementation status: NOT IMPLEMENTED — all criteria are acceptance targets

These criteria describe target product behavior. The UI may satisfy a criterion visually, but authorization and business rules **must** be enforced by the API.

## AC-UC-WF-01 — Configure assignment rules

*(US-WF-01, US-WF-02)*

- Only Admin can create, edit, enable/disable, reorder and archive rules; other roles receive `403`.
- A new rule is saved as Disabled and placed at the end of the ordered list.
- Target team without active members → `400`; empty name or no condition → `400`.
- When several rules match, the rule with the lowest order is applied.
- Dry run returns the matching rule, team and expected technician; the number of assignment and audit records is unchanged afterwards.
- Every change produces a `WORKFLOW_RULE_*` audit record with before and after values.

## AC-UC-WF-02 — Configure SLA

*(US-WF-03, US-WF-04)*

- Only Admin can update SLA policies and the business calendar.
- Resolution time shorter than response time, or warning threshold outside 1–99% → `400`.
- Tickets created after saving use the new version; open tickets keep their attached version unless their priority changes.
- Deadlines skip non-working hours and holidays: a P3 ticket created at 16:30 on Friday (working hours 08:00–17:00) has its response deadline at 11:30 on Monday.
- Time spent in `NEEDS_INFO` is not counted toward the SLA; each pause/resume is stored with its timestamp.

## AC-UC-WF-03 — Configure the AI processing scope

*(US-WF-05, US-WF-06, US-WF-07)*

- Only Admin can change the configuration; threshold outside 0.00–1.00 or clarification rounds outside 0–5 → `400`.
- Disabling `AUTO_APPROVE` → every new flow-A ticket goes to flow C (human) until it is re-enabled.
- A disabled AI action is not invoked for tickets created after it was disabled.
- Whitelist entry referencing a non-existent connector → `400`.
- Entitlement referencing a system without a connector → `400`.
- Entitlement catalog changes take effect immediately for decisions and executions after saving.
- API responses and the UI never contain connector secrets.

## AC-UC-WF-04 — Route ticket

*(US-WF-08, US-WF-09)*

- Every new ticket has exactly one current `ai_decisions` record with `outcome` ∈ {`AUTO_APPROVED`, `NEEDS_INFO`, `ROUTE_TO_HUMAN`}, a confidence and a reason.
- *"Please grant me read access to the shared Accounting folder"* from an Accounting employee, entitlement in the catalog, confidence ≥ threshold → flow A.
- *"My mouse is broken, I need a new one"* → flow C (needs a human); no `workflow_run` is created.
- Confidence < threshold or a required field missing → ticket `NEEDS_INFO`; the requester receives one specific question naming the missing information.
- After the requester answers, the ticket is re-routed; still not passing after 2 rounds → flow C with reason `MAX_CLARIFICATION_REACHED`.
- AI Service does not respond within 30 seconds → flow C with reason `AI_UNAVAILABLE`.
- Routing completes within ≤ 15 seconds (P95) after the ticket is created or the requester answers.

## AC-UC-WF-05 — Auto-assign technician

*(US-WF-10, US-WF-11)*

- A flow-C ticket matching a rule is assigned to an active, non-absent technician of the target team within ≤ 10 seconds (P95).
- With `ROUND_ROBIN`, N consecutive tickets are spread evenly across available members (difference ≤ 1); with `LEAST_LOADED`, the member with the fewest open tickets is chosen.
- The technician notification includes the AI summary **and the routing reason** (e.g. *"Hardware request"*, *"Entitlement X is not in the department catalog"*).
- No rule matches → ticket `TRIAGE` and the intake team lead is notified.
- Reassignment without a reason → `400`; a team lead assigning outside their own team → `403`.
- Automatic assignment audit records have source `AUTOMATION` and `ruleId`.

## AC-UC-WF-06 — Reminders and SLA warnings

*(US-WF-12, US-WF-13, US-WF-14)*

- When a warning threshold is reached, the assignee receives exactly one notification per threshold within ≤ 2 minutes.
- On breach, the ticket gets `sla_breached = true`, the team lead is notified, and an `SLA_BREACHED` event is recorded.
- If priority raise is enabled, priority increases by exactly one level and never above P1.
- `NEEDS_INFO` tickets are reminded on day 1 and day N−1, and automatically become `CLOSED` on day N with reason *no response*.
- The requester can reopen an auto-closed ticket within 7 days; afterwards → `409`.
- `AUTOMATING` tickets are never auto-closed.

## AC-UC-WF-07 — AI approves in-scope request

*(US-WF-15, US-WF-16)*

- Approval only when **all** policy conditions pass: operation in an enabled whitelist entry; entitlement / account template in the beneficiary department's catalog; duration ≤ maximum; beneficiary is the requester or in the same department; beneficiary does not already have that access.
- Any condition fails → `ROUTE_TO_HUMAN`, no data-changing connector call, and the reason names the failed condition.
- A request for a user in another department → flow C, even with confidence = 1.00.
- Beneficiary already has the access → ticket `RESOLVED` with the message *"You already have this access"*; `GRANT_ACCESS` is not called.
- AI output with missing fields or wrong format → flow C; approval never happens on invalid extracted data.
- Admin can view each decision with: confidence, extracted information, result of each condition, configuration version used; the requester only sees a summary reason.
- The approval decision completes within ≤ 5 seconds (P95) after routing.

## AC-UC-WF-08 — Execute via API

*(US-WF-17, US-WF-18)*

- The connector is only called when the decision is `AUTO_APPROVED` and the policy **at execution time** still allows it; otherwise → `FAILED` with code `PRECONDITION_FAILED` and no outbound call.
- `CHECK_ACCOUNT` does not change external data; the result (exists, locked, current access) is sent to the requester.
- The first execution starts within ≤ 30 seconds after AI approval (P95).
- Repeating a call with the same idempotency key creates no additional change in the external access-management system.
- Transient errors are retried up to 3 times (1 / 5 / 15 minutes); permanent errors are not retried automatically.
- Every call stores a `workflow_run_attempt` with result code, duration and a payload with sensitive data masked.
- Time-bound access is revoked within ≤ 5 minutes after expiry; the user is notified 1 day in advance.

## AC-UC-WF-09 — Verify and hand off on failure

*(US-WF-19, US-WF-20)*

- A ticket only becomes `RESOLVED` after successful verification or after a technician marks it resolved manually.
- Requester confirms → `CLOSED`; reports not fixed → assigned to a technician of the owning team.
- Run `FAILED` → ticket assigned to a technician with the error code, failed step and attempt history.
- Retry creates a new run with the **same** idempotency key and re-checks the policy; if the policy no longer allows it → `409`.
- Manual resolution without a note → `400`.
- Only the assigned technician or an Admin may retry / resolve manually; others → `403`.

## Definition of Ready

A backend work item is ready only when it names its OpenAPI operationId, application service, repository dependencies, RBAC rule, validation/error outcomes and unit-test cases. The [traceability matrix](../traceability/master-traceability.md) provides this mapping.
