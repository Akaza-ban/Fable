# Master Traceability Matrix — Workflow & Automation (V1)

> Design status: TARGET V1
> Test status: no tests yet; all verification IDs are PLANNED

Screens (`SCR-*`), operationIds, tables and components are **proposed names**; they will be finalized with the OpenAPI contract, ERD and C4 model.

## Functional traceability

| Requirement | Use Case | Acceptance Criteria | Screen | Component (C3) | API operationId | Database | Business rule | Verification |
|---|---|---|---|---|---|---|---|---|
| US-WF-01 | UC-WF-01 | AC-UC-WF-01 | SCR-WF-01 | Workflow Rules | `listAssignmentRules`, `createAssignmentRule`, `updateAssignmentRule`, `archiveAssignmentRule` | `assignment_rules`, `audit_logs` | BR-WF-09 | `INT-WF-RULE-001` PLANNED |
| US-WF-02 | UC-WF-01 | AC-UC-WF-01 | SCR-WF-01 | Workflow Rules | `reorderAssignmentRules`, `dryRunAssignmentRules` | `assignment_rules` | BR-WF-09 | `UNIT-APP-WF-RULE-001` PLANNED |
| US-WF-03 | UC-WF-02 | AC-UC-WF-02 | SCR-WF-02 | SLA Engine | `listSlaPolicies`, `updateSlaPolicy`, `updateBusinessCalendar` | `sla_policies`, `business_calendars` | BR-WF-08 | `UNIT-APP-WF-SLA-001` PLANNED |
| US-WF-04 | UC-WF-02 | AC-UC-WF-02 | SCR-T-02 | SLA Engine | internal | `ticket_sla_timers`, `sla_events` | BR-WF-07 | `UNIT-APP-WF-SLA-002` PLANNED |
| US-WF-05 | UC-WF-03 | AC-UC-WF-03 | SCR-WF-03 | AI Gateway | `listAiActionSettings`, `updateAiActionSetting` | `ai_action_settings` | BR-WF-06 | `INT-WF-AI-001` PLANNED |
| US-WF-06 | UC-WF-03 | AC-UC-WF-03 | SCR-WF-03 | Automation Catalog | `listAutomationActions`, `createAutomationAction`, `updateAutomationAction` | `automation_actions`, `connectors` | BR-WF-01, BR-WF-15 | `INT-WF-WL-001`, `SEC-WF-SECRET-001` PLANNED |
| US-WF-07 | UC-WF-03 | AC-UC-WF-03 | SCR-WF-03 | Policy Engine | `listDepartmentEntitlements`, `updateDepartmentEntitlements` | `department_entitlements` | BR-WF-02 | `INT-WF-ENT-001` PLANNED |
| US-WF-08 | UC-WF-04 | AC-UC-WF-04 | SCR-T-01, SCR-T-02 | Ticket Router, AI Gateway | `createTicket` + internal | `tickets`, `ai_decisions` | BR-WF-03 | `INT-WF-ROUTE-001`, `PERF-WF-ROUTE-001` PLANNED |
| US-WF-09 | UC-WF-04 | AC-UC-WF-04 | SCR-T-02 | Ticket Router, AI Gateway | `answerClarification` | `ticket_messages`, `ai_decisions` | BR-WF-06 | `INT-WF-ROUTE-002` PLANNED |
| US-WF-10 | UC-WF-05 | AC-UC-WF-05 | Notification, SCR-T-02 | Assignment Engine | internal (ticket enters flow C) | `ticket_assignments`, `group_members`, `audit_logs` | BR-WF-03, BR-WF-09, BR-WF-10 | `UNIT-APP-WF-ASSIGN-001`, `PERF-WF-ASSIGN-001` PLANNED |
| US-WF-11 | UC-WF-05 | AC-UC-WF-05 | SCR-WF-10 | Assignment Engine | `listTriageQueue`, `reassignTicket` | `tickets`, `ticket_assignments`, `audit_logs` | BR-WF-09 | `INT-WF-RBAC-001` PLANNED |
| US-WF-12 | UC-WF-06 | AC-UC-WF-06 | Notification | SLA Engine, Notification | internal (scheduler) | `ticket_sla_timers`, `notifications` | — | `INT-WF-SLA-002` PLANNED |
| US-WF-13 | UC-WF-06 | AC-UC-WF-06 | Notification, SCR-T-02 | SLA Engine, Notification | internal (scheduler) | `tickets`, `sla_events`, `notifications` | — | `INT-WF-SLA-003` PLANNED |
| US-WF-14 | UC-WF-06 | AC-UC-WF-06 | Notification | SLA Engine, Notification | `reopenTicket` | `tickets`, `notifications` | BR-WF-11 | `INT-WF-SLA-004` PLANNED |
| US-WF-15 | UC-WF-07 | AC-UC-WF-07 | SCR-T-02 | Policy Engine, AI Gateway | internal (flow A) | `ai_decisions`, `department_entitlements` | BR-WF-02, BR-WF-16 | `UNIT-APP-WF-POLICY-001`, `SEC-WF-AIPOLICY-001` PLANNED |
| US-WF-16 | UC-WF-07 | AC-UC-WF-07 | SCR-WF-30, SCR-WF-31 | Policy Engine | `listAiDecisions`, `getAiDecision` | `ai_decisions` | BR-WF-17 | `INT-WF-DECISION-001` PLANNED |
| US-WF-17 | UC-WF-08 | AC-UC-WF-08 | SCR-T-02 (Automation tab) | Automation Executor, Connectors | internal (`AiDecisionApproved` event) | `workflow_runs`, `workflow_run_attempts`, `access_grants` | BR-WF-01, BR-WF-04, BR-WF-12, BR-WF-15 | `INT-WF-IDEM-001`, `UNIT-APP-WF-RETRY-001` PLANNED |
| US-WF-18 | UC-WF-08 | AC-UC-WF-08 | Notification | Automation Executor | internal (scheduler) | `access_grants`, `workflow_runs` | BR-WF-14 | `INT-WF-REVOKE-001` PLANNED |
| US-WF-19 | UC-WF-09 | AC-UC-WF-09 | SCR-T-02 | Automation Executor, Connectors | `confirmTicketResolution`, `reopenTicket` | `workflow_runs`, `tickets` | BR-WF-13 | `INT-WF-VERIFY-001` PLANNED |
| US-WF-20 | UC-WF-09 | AC-UC-WF-09 | SCR-T-02 | Automation Executor | `getWorkflowRun`, `retryWorkflowRun`, `resolveWorkflowRunManually` | `workflow_runs`, `workflow_run_attempts`, `audit_logs` | BR-WF-04, BR-WF-12, BR-WF-13 | `INT-WF-FAIL-001`, `INT-WF-RBAC-003` PLANNED |

## Proposed screen inventory

| ID | Screen | Roles |
|---|---|---|
| SCR-T-01 | Create ticket (F03) | All roles |
| SCR-T-02 | Ticket detail — AI conversation, SLA tab, Automation tab (F03) | Per ticket scope |
| SCR-WF-01 | Assignment rules | Admin |
| SCR-WF-02 | SLA policies & business calendar | Admin |
| SCR-WF-03 | AI scope: AI actions, whitelist, department entitlement catalog | Admin |
| SCR-WF-10 | Manual triage queue | Admin, Team Lead |
| SCR-WF-30 | AI decision list (filter by flow, request type, date) | Admin |
| SCR-WF-31 | AI decision detail | Admin |

## Quality / NFR traceability

| Requirement | C4 Component | Proposed mechanism | Verification |
|---|---|---|---|
| NFR-WF-CAP-001, NFR-WF-PERF-001 | Ticket Router, AI Gateway | asynchronous processing queue, multiple workers calling the AI in parallel | `PERF-WF-LOAD-001`, `PERF-WF-ROUTE-001` PLANNED |
| NFR-WF-PERF-002, NFR-WF-SAFE-001 | Policy Engine | pure policy evaluation (no AI calls), cached entitlement catalog | `PERF-WF-APPROVE-001`, `SEC-WF-AIPOLICY-001` PLANNED |
| NFR-WF-PERF-003, NFR-WF-REL-001 | Automation Executor | DB-backed job queue (outbox), background workers | `PERF-WF-EXEC-001`, `INT-WF-DURABLE-001` PLANNED |
| NFR-WF-PERF-004 | Assignment Engine | in-memory rule cache, index by team | `PERF-WF-ASSIGN-001` PLANNED |
| NFR-WF-PERF-005 | SLA Engine | 1-minute job scanning `ticket_sla_timers` by `next_due_at` | `INT-WF-SLA-002` PLANNED |
| NFR-WF-REL-002/003 | Automation Executor, Connectors | unique idempotency key, 1/5/15 minute backoff | `INT-WF-IDEM-001`, `UNIT-APP-WF-RETRY-001` PLANNED |
| NFR-WF-REL-004 | Ticket Router | 30-second AI call timeout → flow C | `INT-WF-AIDOWN-001` PLANNED |
| NFR-WF-SEC-001/002 | Connectors | secrets from environment / secret store, least-privilege service accounts, payload masking | `SEC-WF-SECRET-001`, `SEC-WF-LEASTPRIV-001` PLANNED |
| NFR-WF-SEC-003 | HTTP API | deny-by-default policies | `INT-WF-RBAC-*` PLANNED |
| NFR-WF-AUD-001 | Audit Log | audit written in the same transaction as the state change | `INT-WF-AUDIT-001` PLANNED |
| NFR-WF-OBS-001 | Policy Engine, Reporting (F07) | aggregate queries on `ai_decisions` | `INT-WF-METRIC-001` PLANNED |
| NFR-WF-EXT-001 | Connectors | `IAutomationConnector` interface (`Execute`, `Verify`) | `ARCH-WF-001` PLANNED |
