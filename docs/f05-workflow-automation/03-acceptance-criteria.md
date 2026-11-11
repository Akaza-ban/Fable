# Acceptance Criteria — Workflow & Automation (V1)

> Design status: TARGET V1
>
> Implementation status: NOT IMPLEMENTED — all criteria are acceptance targets

These criteria describe target product behavior. The UI may satisfy a criterion visually, but authorization and business rules **must** be enforced by the API.

## AC-UC-WF-01 — Configure workload scoring

*(US-WF-21)*

- Only Admin can change workload settings; other roles receive `403`.
- Priority points, complexity factors or maximum load ≤ 0 → `400`; every request type has exactly one complexity (Simple / Medium / Complex); a default support team with at least one active member is mandatory.
- The preview returns the AI category, the responsible team, each member's status (`ON` / `OFF`) and workload score, who is excluded and why, and the expected technician; the number of assignment and audit records is unchanged afterwards.
- New settings apply from the next assignment; tickets already assigned are not moved.
- Every change produces a `WORKLOAD_SETTINGS_*` audit record with before and after values.
- There is no screen or API to create assignment rules.

## AC-UC-WF-02 — Configure SLA

*(US-WF-03, US-WF-04)*

- Only Admin can update SLA policies and the business calendar.
- Resolution time shorter than response time, or warning threshold outside 1–99% → `400`.
- Tickets created after saving use the new version; open tickets keep their attached version unless their priority changes.
- Only three priority levels exist (P1 High, P2 Medium, P3 Low); a policy or ticket with any other priority → `400`.
- Deadlines skip non-working hours and holidays: a P2 ticket created at 16:30 on Friday (working hours 08:00–17:00) has its response deadline at 11:30 on Monday.
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
- Disabling `AUTO_FILL_FIELDS` → new tickets keep empty fields for a technician to fill; disabling `SET_PRIORITY` → new tickets get the default priority; disabling `AGENT_FIRST` for a domain → its tickets have no AI agent owner.
- The default priority must be one of P1–P3 → otherwise `400`.

## AC-UC-WF-04 — Route ticket

*(US-WF-08, US-WF-09, US-WF-22, US-WF-23, US-WF-24)*

- The routing decision uses the full AI analysis: request, requester (department, business unit, site), source `channel` (`EMAIL` / `PORTAL`) and `createdBy`; the source is shown on the ticket.
- A ticket from an unknown sender (`UNKNOWN_SENDER`) is never routed to flow A and goes to flow C.
- Every routed ticket shows its support level `L1` or `L2` as computed by F04; `HELPDESK` domains are `L1`, application domains and business requirements are `L2`.
- With `AUTO_FILL_FIELDS` on, request type, category, subcategory, support level, requested for, business unit, site and priority are filled and each is marked *"Filled by AI"*; a field below the confidence threshold stays empty and is asked for (flow B).
- With `SET_PRIORITY` on, the AI priority is applied when its confidence ≥ threshold, otherwise P2; the SLA of that priority is attached in the same step.
- A technician or the Admin can change an AI-filled field or the priority only with a reason (missing → `400`); the change is audited with before/after values and recalculates the SLA. A change of category or support level before work starts re-routes the ticket.
- When the domain has an AI agent and `AGENT_FIRST` is on, the ticket's owner is that agent in flows A and B; the owner becomes a technician when the ticket enters flow C.

- Every new ticket has exactly one current `ai_decisions` record with `outcome` ∈ {`AUTO_APPROVED`, `NEEDS_INFO`, `ROUTE_TO_HUMAN`}, a confidence and a reason.
- *"Please grant me read access to the shared Accounting folder"* from an Accounting employee, entitlement in the catalog, confidence ≥ threshold → flow A.
- *"My mouse is broken, I need a new one"* → flow C (needs a human); no `workflow_run` is created.
- Confidence < threshold or a required field missing → ticket `NEEDS_INFO`; the requester receives one specific question naming the missing information.
- After the requester answers, the ticket is re-routed; still not passing after 2 rounds → flow C with reason `MAX_CLARIFICATION_REACHED`.
- AI Service does not respond within 30 seconds → flow C with reason `AI_UNAVAILABLE`.
- Routing completes within ≤ 15 seconds (P95) after the ticket is created or the requester answers.

## AC-UC-WF-05 — Auto-assign technician

*(US-WF-10, US-WF-11)*

- A flow-C ticket is assigned within ≤ 10 seconds (P95) to a technician of the target team who is `ON` and below the maximum load.
- Among eligible technicians, the one with the **lowest workload score** is chosen (score = Σ priority points × complexity factor of their `ASSIGNED` and `IN_PROGRESS` tickets). Example: Mai (3 × P1 Complex = 18) and Hung (5 × P3 Simple = 5) are both `ON` → a new ticket goes to Hung.
- A technician who is `OFF` never receives a new ticket, even with the lowest score.
- If nobody is `ON` or everyone is at maximum load, the ticket waits in the team queue and is assigned within ≤ 1 minute after a technician becomes eligible.
- The ticket is placed in the group of its domain and the sub-group of its subcategory (F09). Example: `HELPDESK/PRINTER` → IT Helpdesk group, Printer sub-group; the technician is chosen among the Printer sub-group members.
- No sub-group handles the subcategory → members of the group are used; no group owns the domain, or the AI could not classify the ticket → the default support team is used with the same selection rule; no ticket is left without a team.
- A technician switching to `OFF` → their `ASSIGNED` (not started) tickets are reassigned within ≤ 1 minute; their `IN_PROGRESS` tickets stay with them.
- The technician notification includes the AI summary **and the routing reason** (e.g. *"Hardware request"*, *"Entitlement X is not in the department catalog"*).
- Manual reassignment is Admin only (other roles → `403`) and requires a reason (missing → `400`).
- Automatic assignment audit records have source `AUTOMATION`, the AI category, the team and the workload scores compared.

## AC-UC-WF-06 — Reminders and SLA warnings

*(US-WF-12, US-WF-13, US-WF-14)*

- When a warning threshold is reached, the assignee receives exactly one notification per threshold within ≤ 2 minutes.
- On breach, the ticket gets `sla_breached = true`, the Admin is notified, and an `SLA_BREACHED` event is recorded.
- On breach, a ticket that is not started (`ASSIGNED`) or whose assignee is now `OFF` is reassigned automatically with the selection rule; a ticket `IN_PROGRESS` whose assignee is `ON` is not moved.
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

## AC-UC-WF-10 — Handle duplicate and similar tickets

*(US-WF-25, US-WF-26, US-WF-27)*

- With `HANDLE_DUPLICATES` on, a new ticket whose similarity with an **open ticket of the same requester** is ≥ the duplicate threshold (0.85) is linked as duplicate, closed with reason *"Duplicate of #T-…"*, and its text and attachments are added as a comment to the earlier ticket; it is never assigned and consumes no SLA.
- Example: Lan emails *"Outlook keeps asking for my password"* twice in 10 minutes → the second email is merged into the first ticket; Lan receives one notification with the **Not the same problem** action.
- **Not the same problem** within 7 days removes the link, reopens the ticket as `NEW` and routes it normally; after 7 days → `409`. The same pair is never merged again.
- A ticket is never merged into a ticket of another requester, nor into a ticket that is `RESOLVED`, `CLOSED` or `CANCELLED`.
- A ticket similar (≥ threshold) to an open ticket of **another** requester stays open, is linked as related and is assigned to the same owner, unless that owner is `OFF` or at maximum load (then the normal selection rule applies and the link stays).
- When at least 5 similar open tickets on the same system arrive within 60 minutes, the oldest becomes the main ticket, the others are linked to it and routed to its owner, the main ticket's priority rises by one level (never above P1), and the Admin receives exactly one *"Possible incident"* notification for the burst.
- **Unlink** by the assigned technician or the Admin requires a reason (missing → `400`), routes the ticket normally and prevents the pair from being linked again; other roles → `403`.
- A candidate below the threshold is never linked automatically.
- Every merge, link, undo and unlink is audited with source `AUTOMATION` or `USER`, the similarity and the ticket ids.
- With `HANDLE_DUPLICATES` off, no ticket is merged or linked automatically; F04 still shows candidates to technicians.

## AC-UC-WF-11 — AI re-routes on failure or misroute

*(US-WF-28, US-WF-29, US-WF-30)*

- With `AUTO_REROUTE` on, *Return for re-routing* by the assigned technician (reason required, missing → `400`) triggers a new AI analysis within ≤ 15 seconds (P95); the reason is given to the AI as context and the group just left is excluded.
- Example: Hai (Network team) returns *"VPN keeps disconnecting in SAP"* with *"SAP authorization error, not network"* → the ticket is re-analysed as `SAP/AUTHORIZATION` and assigned to the SAP group; Hai no longer sees it in his list.
- A domain agent reporting a wrong domain, or an API failure showing a wrong target, re-routes the ticket the same way with reason `AGENT_WRONG_DOMAIN` or `AUTOMATION_WRONG_TARGET`.
- An agent that cannot resolve the ticket (`AGENT_NOT_SOLVED`) does not change the group: the ticket goes to a technician of the same sub-group, chosen by the selection rule.
- A new analysis that points to the group just left sends the ticket to the default support team instead.
- After the maximum number of re-routes (default 2), the next failure sends the ticket to the default support team with reason `REROUTE_LIMIT_REACHED` and the Admin receives one notification.
- Each re-route stores a new `ai_decisions` version with its reason code; the routing history (groups, owners, reasons, times) is shown on the ticket.
- A re-route never resets the SLA, never lowers the priority, keeps the fields corrected by a person and keeps duplicate links.
- *Return for re-routing* by another role → `403`; on a `RESOLVED`, `CLOSED` or `CANCELLED` ticket → `409`.
- With `AUTO_REROUTE` off, misrouted and failed tickets go to a technician of the current group as before (UC-WF-09).

## Definition of Ready

A backend work item is ready only when it names its OpenAPI operationId, application service, repository dependencies, RBAC rule, validation/error outcomes and unit-test cases. The [traceability matrix](../traceability/master-traceability.md) provides this mapping.
