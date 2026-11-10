# Non-Functional Requirements — Workflow & Automation (V1)

> Design status: PROPOSED V1 BASELINE
> Implementation status: TARGET; verification is PLANNED unless evidence says otherwise

These targets are a baseline for an internal enterprise system, not a commercial SLA. A requirement is only complete when its verification method passes under the reference workload.

## Reference workload

| Parameter | Value | Notes |
|---|---|---|
| New tickets | **8,000 / day** | From all departments |
| Average over business hours (8 h) | ~1,000 / hour ≈ 17 / minute | |
| Peak hour (assumed 2× average) | ~2,000 / hour ≈ 35 / minute | Used for load testing |
| AI clarification rounds | max 2 / ticket | Each round triggers a re-route |
| Accumulated data | ~2.9 million tickets / year | Plus `ai_decisions`, `workflow_runs`, audit logs |
| Number of technicians | not yet defined | See Open questions in README |

## NFR list

| ID | Statement | Rationale | Measurement | Verification method | Status |
|---|---|---|---|---|---|
| NFR-WF-CAP-001 | The system shall sustain a peak load of **35 tickets/minute** for 1 hour while meeting the PERF targets below. | Covers 8,000 tickets/day with headroom. | Load test at 2,000 tickets/hour. | `PERF-WF-LOAD-001` | PROPOSED / PLANNED |
| NFR-WF-PERF-001 | Routing (AI classification + flow decision) shall complete in ≤ 15 seconds (P95). | Requesters get an almost immediate response. | Ticket creation / answer time vs. `ai_decisions` record. | `PERF-WF-ROUTE-001` | PROPOSED / PLANNED |
| NFR-WF-PERF-002 | Policy check and approval decision shall take ≤ 5 seconds (P95) after routing. | Automatic approval must be instant. | Time difference. | `PERF-WF-APPROVE-001` | PROPOSED / PLANNED |
| NFR-WF-PERF-003 | The first connector call shall start ≤ 30 seconds (P95) after AI approval. | Requesters get access quickly. | Approval time vs. first attempt. | `PERF-WF-EXEC-001` | PROPOSED / PLANNED |
| NFR-WF-PERF-004 | Automatic technician assignment shall complete ≤ 10 seconds (P95) after a ticket enters flow C. | Tickets that need a human are picked up immediately. | Time difference. | `PERF-WF-ASSIGN-001` | PROPOSED / PLANNED |
| NFR-WF-PERF-005 | SLA warnings shall be sent ≤ 2 minutes after the threshold is reached. | Reminders are only useful when timely. | Theoretical threshold vs. send time. | `INT-WF-SLA-002` | PROPOSED / PLANNED |
| NFR-WF-REL-001 | Tickets awaiting routing, SLA timers and runs awaiting retry shall be persisted (DB-backed queue) and resume correctly after a service restart. | No lost tickets at 8,000/day. | Mid-flight restart test. | `INT-WF-DURABLE-001` | PROPOSED / PLANNED |
| NFR-WF-REL-002 | No duplicate change in the external access-management system on repeated calls, timeouts or restarts. | Guarantees BR-WF-04. | Number of changes on the mock connector under fault injection. | `INT-WF-IDEM-001` | PROPOSED / PLANNED |
| NFR-WF-REL-003 | Transient errors shall be retried at most 3 times with 1 / 5 / 15 minute backoff. | Tolerates short outages of external systems. | Attempt timestamps. | `UNIT-APP-WF-RETRY-001` | PROPOSED / PLANNED |
| NFR-WF-REL-004 | When the AI Service is unavailable, new tickets shall be routed to flow C (human) within ≤ 30 seconds; no ticket shall get stuck. | The system keeps operating when the AI fails. | AI Service shutdown test. | `INT-WF-AIDOWN-001` | PROPOSED / PLANNED |
| NFR-WF-SAFE-001 | The rate of AI-approved grants outside the catalog shall be **0** in an adversarial test suite (deliberately misleading tickets, requests for other departments, prompt injection). | AI misunderstanding must never lead to a wrongful grant. | Number of `GRANT_ACCESS`/`CREATE_ACCOUNT` outside the catalog. | `SEC-WF-AIPOLICY-001` | PROPOSED / PLANNED |
| NFR-WF-SEC-001 | Connector credentials shall only live in deployment secrets, never in the DB, logs or API responses. | Reduces key-leak risk. | Scan of sample logs/DB. | `SEC-WF-SECRET-001` | PROPOSED / PLANNED |
| NFR-WF-SEC-002 | Connector service accounts shall have only the minimum privileges required for whitelisted operations. | Limits damage if abused. | Service account permission review. | `SEC-WF-LEASTPRIV-001` | PROPOSED / PLANNED |
| NFR-WF-SEC-003 | 100% of workflow endpoints shall enforce authorization server-side and declare Bearer security. | The browser is not a security boundary. | Policy coverage and contract inspection. | `INT-WF-RBAC-*` | PROPOSED / PLANNED |
| NFR-WF-AUD-001 | 100% of actions with source `AI`/`AUTOMATION` shall have an audit record with grounds. | Traceability and accountability. | Event-to-audit reconciliation. | `INT-WF-AUDIT-001` | PROPOSED / PLANNED |
| NFR-WF-OBS-001 | The system shall measure the **AI self-resolution rate**, clarification rate and human-routing rate per day / request type. | Evaluates AI effectiveness (feeds F07). | Queries on `ai_decisions`. | `INT-WF-METRIC-001` | PROPOSED / PLANNED |
| NFR-WF-MAINT-001 | Unit test coverage of the policy check, assignment rules and SLA computation shall be ≥ 90%. | High-risk logic. | Coverage report by namespace. | CI coverage | PROPOSED / PLANNED |
| NFR-WF-EXT-001 | Adding a new connector shall only require implementing an `IAutomationConnector` interface (`Execute`, `Verify`), without changing workflow orchestration. | Easy extension to other access-management systems. | Architecture review. | `ARCH-WF-001` | PROPOSED / PLANNED |
