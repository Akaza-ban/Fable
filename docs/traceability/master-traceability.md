# Master Traceability Matrix — Workflow & tự động hóa (V1)

> Trạng thái thiết kế: TARGET V1
> Trạng thái kiểm thử: chưa có test; mọi mã kiểm chứng là PLANNED

Màn hình (`SCR-*`), operationId, bảng dữ liệu và component là **tên dự kiến**; sẽ được chốt khi viết OpenAPI, ERD và C4.

## Truy vết chức năng

| Requirement | Use Case | Acceptance Criteria | Màn hình | Component (C3) | API operationId | Database | Business rule | Kiểm chứng |
|---|---|---|---|---|---|---|---|---|
| US-WF-01 | UC-WF-01 | AC-UC-WF-01 | SCR-WF-01 | Workflow Rules | `listAssignmentRules`, `createAssignmentRule`, `updateAssignmentRule`, `archiveAssignmentRule` | `assignment_rules`, `audit_logs` | BR-WF-09 | `INT-WF-RULE-001` PLANNED |
| US-WF-02 | UC-WF-01 | AC-UC-WF-01 | SCR-WF-01 | Workflow Rules | `reorderAssignmentRules`, `dryRunAssignmentRules` | `assignment_rules` | BR-WF-09 | `UNIT-APP-WF-RULE-001` PLANNED |
| US-WF-03 | UC-WF-02 | AC-UC-WF-02 | SCR-WF-02 | SLA Engine | `listSlaPolicies`, `updateSlaPolicy`, `updateBusinessCalendar` | `sla_policies`, `business_calendars` | BR-WF-08 | `UNIT-APP-WF-SLA-001` PLANNED |
| US-WF-04 | UC-WF-02 | AC-UC-WF-02 | SCR-T-02 | SLA Engine | nội bộ | `ticket_sla_timers`, `sla_events` | BR-WF-07 | `UNIT-APP-WF-SLA-002` PLANNED |
| US-WF-05 | UC-WF-03 | AC-UC-WF-03 | SCR-WF-03 | AI Gateway | `listAiActionSettings`, `updateAiActionSetting` | `ai_action_settings` | BR-WF-06 | `INT-WF-AI-001` PLANNED |
| US-WF-06 | UC-WF-03 | AC-UC-WF-03 | SCR-WF-03 | Automation Catalog | `listAutomationActions`, `createAutomationAction`, `updateAutomationAction` | `automation_actions`, `connectors` | BR-WF-01, BR-WF-15 | `INT-WF-WL-001`, `SEC-WF-SECRET-001` PLANNED |
| US-WF-07 | UC-WF-03 | AC-UC-WF-03 | SCR-WF-03 | Policy Engine | `listDepartmentEntitlements`, `updateDepartmentEntitlements` | `department_entitlements` | BR-WF-02 | `INT-WF-ENT-001` PLANNED |
| US-WF-08 | UC-WF-04 | AC-UC-WF-04 | SCR-T-01, SCR-T-02 | Ticket Router, AI Gateway | `createTicket` + nội bộ | `tickets`, `ai_decisions` | BR-WF-03 | `INT-WF-ROUTE-001`, `PERF-WF-ROUTE-001` PLANNED |
| US-WF-09 | UC-WF-04 | AC-UC-WF-04 | SCR-T-02 | Ticket Router, AI Gateway | `answerClarification` | `ticket_messages`, `ai_decisions` | BR-WF-06 | `INT-WF-ROUTE-002` PLANNED |
| US-WF-10 | UC-WF-05 | AC-UC-WF-05 | Thông báo, SCR-T-02 | Assignment Engine | nội bộ (ticket vào luồng C) | `ticket_assignments`, `group_members`, `audit_logs` | BR-WF-03, BR-WF-09, BR-WF-10 | `UNIT-APP-WF-ASSIGN-001`, `PERF-WF-ASSIGN-001` PLANNED |
| US-WF-11 | UC-WF-05 | AC-UC-WF-05 | SCR-WF-10 | Assignment Engine | `listTriageQueue`, `reassignTicket` | `tickets`, `ticket_assignments`, `audit_logs` | BR-WF-09 | `INT-WF-RBAC-001` PLANNED |
| US-WF-12 | UC-WF-06 | AC-UC-WF-06 | Thông báo | SLA Engine, Notification | nội bộ (scheduler) | `ticket_sla_timers`, `notifications` | — | `INT-WF-SLA-002` PLANNED |
| US-WF-13 | UC-WF-06 | AC-UC-WF-06 | Thông báo, SCR-T-02 | SLA Engine, Notification | nội bộ (scheduler) | `tickets`, `sla_events`, `notifications` | — | `INT-WF-SLA-003` PLANNED |
| US-WF-14 | UC-WF-06 | AC-UC-WF-06 | Thông báo | SLA Engine, Notification | `reopenTicket` | `tickets`, `notifications` | BR-WF-11 | `INT-WF-SLA-004` PLANNED |
| US-WF-15 | UC-WF-07 | AC-UC-WF-07 | SCR-T-02 | Policy Engine, AI Gateway | nội bộ (luồng A) | `ai_decisions`, `department_entitlements` | BR-WF-02, BR-WF-16 | `UNIT-APP-WF-POLICY-001`, `SEC-WF-AIPOLICY-001` PLANNED |
| US-WF-16 | UC-WF-07 | AC-UC-WF-07 | SCR-WF-30, SCR-WF-31 | Policy Engine | `listAiDecisions`, `getAiDecision` | `ai_decisions` | BR-WF-17 | `INT-WF-DECISION-001` PLANNED |
| US-WF-17 | UC-WF-08 | AC-UC-WF-08 | SCR-T-02 (tab Tự động hóa) | Automation Executor, Connectors | nội bộ (sự kiện `AiDecisionApproved`) | `workflow_runs`, `workflow_run_attempts`, `access_grants` | BR-WF-01, BR-WF-04, BR-WF-12, BR-WF-15 | `INT-WF-IDEM-001`, `UNIT-APP-WF-RETRY-001` PLANNED |
| US-WF-18 | UC-WF-08 | AC-UC-WF-08 | Thông báo | Automation Executor | nội bộ (scheduler) | `access_grants`, `workflow_runs` | BR-WF-14 | `INT-WF-REVOKE-001` PLANNED |
| US-WF-19 | UC-WF-09 | AC-UC-WF-09 | SCR-T-02 | Automation Executor, Connectors | `confirmTicketResolution`, `reopenTicket` | `workflow_runs`, `tickets` | BR-WF-13 | `INT-WF-VERIFY-001` PLANNED |
| US-WF-20 | UC-WF-09 | AC-UC-WF-09 | SCR-T-02 | Automation Executor | `getWorkflowRun`, `retryWorkflowRun`, `resolveWorkflowRunManually` | `workflow_runs`, `workflow_run_attempts`, `audit_logs` | BR-WF-04, BR-WF-12, BR-WF-13 | `INT-WF-FAIL-001`, `INT-WF-RBAC-003` PLANNED |

## Danh mục màn hình dự kiến

| Mã | Màn hình | Vai trò |
|---|---|---|
| SCR-T-01 | Tạo ticket (F03) | Mọi vai trò |
| SCR-T-02 | Chi tiết ticket — hội thoại với AI, tab SLA, tab Tự động hóa (F03) | Theo phạm vi ticket |
| SCR-WF-01 | Quy tắc phân công | Admin |
| SCR-WF-02 | Chính sách SLA & lịch làm việc | Admin |
| SCR-WF-03 | Phạm vi AI: hành động AI, whitelist, danh mục quyền theo phòng ban | Admin |
| SCR-WF-10 | Hàng chờ phân loại thủ công | Admin, Trưởng nhóm |
| SCR-WF-30 | Danh sách quyết định của AI (lọc theo luồng, loại yêu cầu, ngày) | Admin |
| SCR-WF-31 | Chi tiết quyết định của AI | Admin |

## Truy vết chất lượng / NFR

| Requirement | C4 Component | Cơ chế cài đặt dự kiến | Kiểm chứng |
|---|---|---|---|
| NFR-WF-CAP-001, NFR-WF-PERF-001 | Ticket Router, AI Gateway | hàng đợi xử lý bất đồng bộ, nhiều worker gọi AI song song | `PERF-WF-LOAD-001`, `PERF-WF-ROUTE-001` PLANNED |
| NFR-WF-PERF-002, NFR-WF-SAFE-001 | Policy Engine | kiểm tra chính sách thuần (không gọi AI), cache danh mục quyền | `PERF-WF-APPROVE-001`, `SEC-WF-AIPOLICY-001` PLANNED |
| NFR-WF-PERF-003, NFR-WF-REL-001 | Automation Executor | hàng đợi job lưu trong DB (outbox), worker nền | `PERF-WF-EXEC-001`, `INT-WF-DURABLE-001` PLANNED |
| NFR-WF-PERF-004 | Assignment Engine | quy tắc cache trong bộ nhớ, index theo nhóm | `PERF-WF-ASSIGN-001` PLANNED |
| NFR-WF-PERF-005 | SLA Engine | job định kỳ 1 phút quét `ticket_sla_timers` theo `next_due_at` | `INT-WF-SLA-002` PLANNED |
| NFR-WF-REL-002/003 | Automation Executor, Connectors | khóa idempotency duy nhất, backoff 1/5/15 phút | `INT-WF-IDEM-001`, `UNIT-APP-WF-RETRY-001` PLANNED |
| NFR-WF-REL-004 | Ticket Router | timeout gọi AI 30 giây → luồng C | `INT-WF-AIDOWN-001` PLANNED |
| NFR-WF-SEC-001/002 | Connectors | secret từ biến môi trường / secret store, tài khoản dịch vụ quyền tối thiểu, che payload | `SEC-WF-SECRET-001`, `SEC-WF-LEASTPRIV-001` PLANNED |
| NFR-WF-SEC-003 | HTTP API | policy deny-by-default | `INT-WF-RBAC-*` PLANNED |
| NFR-WF-AUD-001 | Audit Log | ghi nhật ký trong cùng transaction với thay đổi trạng thái | `INT-WF-AUDIT-001` PLANNED |
| NFR-WF-OBS-001 | Policy Engine, Reporting (F07) | truy vấn tổng hợp trên `ai_decisions` | `INT-WF-METRIC-001` PLANNED |
| NFR-WF-EXT-001 | Connectors | interface `IAutomationConnector` (`Execute`, `Verify`) | `ARCH-WF-001` PLANNED |
